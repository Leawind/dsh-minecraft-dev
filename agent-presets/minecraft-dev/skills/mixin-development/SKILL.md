---
name: mixin-development
description: Use when writing or modifying Mixins in a Minecraft mod — choosing an injection point, following compatibility-safe practices, or fixing Mixin-related crashes.
---

# Mixin 开发

Mixin 是 Minecraft 模组修改原版行为的主要手段。Mixin 用不好会与其他模组冲突甚至崩溃。本文的每条规则都从**冲突的本质**出发推导：多个模组对同一个目标类做修改时，谁改得越"整体"，越容易互相覆盖。文中提到的开源模组（Lithium、Fabric API、Immersive Portals 等，见 `docs/References.md`）只是印证这些原理的实例——做法是否合理，以原理为准，不是以"某个模组这么做"为准。

## 核心原理：注入方式的冲突面 = 它接管了多少语义

- `@Redirect` 接管的是"调用本身"：它把一次调用整体替换掉，两个模组都想接管同一个调用时就撞车。**冲突面最大。**
- `@Inject`、`@ModifyVariable`、`@ModifyArg`、`@ModifyExpressionValue`、`@WrapOperation` 只改动局部：注入代码与原调用共存，多个模组可以叠加。**冲突面小，可组合。**
- 结论：能局部改就不要整体接管；`@Inject` 能解决的不要用 `@WrapOperation`，`@WrapOperation` 能解决的不要用 `@Redirect`。

| 注入方式                                | 它接管了什么 / 为什么安全或危险                                                      |
| --------------------------------------- | ------------------------------------------------------------------------------------ |
| `@Inject`（HEAD / RETURN / 特定锚点）   | 只插入代码，不改返回值；与原逻辑共存                                                 |
| `@ModifyVariable`                       | 只改方法参数或局部变量的值                                                           |
| `@ModifyArg`                            | 只改某次调用的一个实参（如 Mod Menu 调整按钮高度）                                   |
| `@ModifyExpressionValue`（MixinExtras） | 只改表达式的结果值，不替换调用——多个模组可链式叠加，冲突面最小                       |
| `@WrapOperation`（MixinExtras）         | 保留原调用（`original.call(...)`），在其前后加逻辑；两个 `@WrapOperation` 可嵌套共存 |
| `@Local`（MixinExtras）                 | 只是取局部变量的语法糖，不改变注入语义                                               |
| `@Redirect`                             | **整体替换一次调用**；两个模组都 Redirect 同一点必冲突。确需"完全接管"时才用         |

`@WrapOperation`、`@ModifyExpressionValue`、`@Local` 来自 **MixinExtras**，不是原生 Mixin 注解。Fabric Loader 0.15.x+ 已内置传递提供（`mixinextras-fabric`，1.21.4 时代内置 0.4.1），无需显式声明；需要显式声明时坐标是 `io.github.llamalad7:mixinextras-fabric`。

## 必须遵守的兼容性细节（每条都有原理）

- **`remap = false`**：映射（refmap）只处理 Minecraft 类；`java.*`、`com.google.*`、`org.joml.*` 等非 MC 类不参与反混淆，若让 Mixin 按 MC 类改写 target 字符串会映射错乱、启动报 target not found。所以非 MC 调用点在 `@At` 的 `target` 里必须写 `remap = false`。
- **方法名的写法**：目标方法名随映射（yarn/mojmap）与版本变化。yarn 环境直接写 yarn 名（如 `"tick"`）；有重载歧义或 mojmap 环境写完整描述符（如 `"collide(Lnet/minecraft/world/phys/Vec3;)Lnet/minecraft/world/phys/Vec3;"`）消除歧义。
- **`require = N`**：注入点失配时，`defaultRequire = 0` 会静默跳过——注入"没生效"但没人知道。`require = N` 把"必须匹配 N 处"变成显式错误，防止反混淆或版本迁移后注入点静默失效。
- **`@Overwrite` 是冲突面最大的操作**：同一方法被两个模组 `@Overwrite` 时后应用者覆盖前者、静默生效。确需使用时，`@author` / `@reason` Javadoc 是给维护者和审核者的审计记录（这也是 Fabric 的审核要求）。
- **`@Unique` + `modid_` 前缀**：注入进目标类命名空间的所有标识符都是全局共享的，不加命名空间前缀的自创成员可能与另一模组的同名成员冲突。接口方法前缀（`fabric_`、`lithium$`、`IE*`）同理，是把命名空间显式划给本模组。
- **注入点选"被调用频率低、语义稳定"的地方**：每帧调用或热路径上的注入既拖性能，又放大与其他模组的交互概率。

## mixin 配置文件（`modid.mixins.json`）

字段的作用（以 Mod Menu、Fabric API、Lithium 的真实配置为验证）：

- `package`：Mixin 类所在包。
- `required: true` + `compatibilityLevel`（`JAVA_17` / `JAVA_21`）：基础要求。
- `minVersion`：可选（Mod Menu 用 `"0.8"`；较新的 Lithium 已去掉它）。
- `injectors.defaultRequire: 1`：注入点必须匹配，不匹配启动即报错——**排查"注入没生效"先看这个**。
- `injectors.maxShiftBy`：允许 `At.Shift` 偏移的指令数（Fabric API 用 `3`）。
- `mixins` / `client` / `server` 三分组：按运行端分发，客户端专用 Mixin 放 `client`，服务端放 `server`。
- `plugin`：`IMixinConfigPlugin` 实现类，可在运行时按条件启用/禁用单个 Mixin（见下）。
- `overwrites.conformVisibility`：让 `@Overwrite` 与目标方法可见性一致（Lithium fabric 配置里有）。

## 实用模式（原理 + 实例）

- **接口 Mixin**：目标是接口时，Mixin 也写成 `interface`，`@Shadow` 方法写成抽象方法、注入逻辑写成 `default` 方法。原理：`default` 方法天然可组合——多个 Mixin 可以各自提供不同的 `default` 实现而互不覆盖；同时避免 `(Target)(Object)this` 强转。（实例：Lithium 的 BlockGetterMixin）
- **换静态数据结构**：`@Shadow @Final @Mutable` 字段 + 在 `<clinit>` 的 `@At("RETURN")` 注入后改写。原理：静态数据只有一份，改写必须在类初始化完成后；`<clinit>` 恰好只执行一次、语义稳定。（实例：Lithium 替换 `Mth.SIN`）
- **打匿名内部类**：外层 Mixin 类里嵌套 `static class` + `@Mixin(targets = "全限定名$1")`。原理：匿名类没有源码名，只能用编译产物名定位——这类注入天生脆弱，只在确实需要时才用。（实例：Lithium 打 `CompoundTag$1`）
- **锚定 lambda 生成的方法**：`@Shadow(aliases = "lambda$...")`。原理：lambda 生成的方法名由编译器决定、版本间可能变化，`aliases` 提供多个候选名提高命中率。
- **精确锚点**："在某字段写入之后" = `@At(value = "FIELD", target = "Lnet/...;字段名:Z", opcode = Opcodes.PUTFIELD, ordinal = 0, shift = At.Shift.AFTER)`。原理：注入点越精确，注入时目标状态越确定，语义越可靠。（实例：Fabric API 的 ServerWorldMixin）
- **`priority` 是"抢注"，慎用**：同一点多个注入按 priority 排序应用，调高 priority 会压过其他模组，但也增加了与它们的耦合。（实例：IP 用 `900` 接管传送包处理——它的场景是"必须独占"，不是常规选择）
- **加载期规避冲突**：与其在运行时与对方模组的行为冲突，不如在加载期检测到对方后跳过自己的冲突注入——失败关闭式兼容。（实例：IP 的 `IMixinConfigPlugin.shouldApplyMixin` 检查 `isModLoaded("porting_lib")`）

## 版本差异处理

目标方法签名随 Minecraft 版本变化。**工具选择取决于差异的"结构"**：

- **结构性大差异（大版本间的重构、类体系变化）→ 按 MC 版本分 git 分支**，各分支独立演进；跨版本少量差异用平台 API 判断 + 独立 `*Mixin` 变体。（实例：Lithium 的 `1.20.1`、`1.21.4`、`26.1.x` 各一条分支）
- **点状差异（签名、方法名、参数微调）→ 条件编译**，同一份源码里用条件块区分版本——见 `stonecutter-multiversion` 技能。

## 编写步骤

1. 确认目标类在目标 Minecraft 版本中的真实类名、方法名、签名（参考该版本的反编译源码或 mappings；多版本项目记得切到对应分支）。
2. 按"冲突面最小"原则选注入方式：`@Inject` 能解决的不要用 `@WrapOperation`，`@WrapOperation` 能解决的不要用 `@Redirect`。
3. 使用 `@At` 指定精确位置；`RETURN` 注入时注意 `CallbackInfo` 的使用（`cancellable` 只在需要取消时开启）。
4. 在 mixin 配置文件（如 `modid.mixins.json`）中注册 mixin 类，注意包名和 `client` / `server` 的归属。

## 验证

- 运行游戏到注入点实际触发的位置（不只是启动），确认注入逻辑生效且无报错。
- 给 run 配置加 `-Dmixin.debug.export=true` 可导出反编译的目标方法，核对注入点是否落在预期位置（这是调试手段，不是惯例要求）。
- 注入点"没生效"时按顺序查：`injectors.defaultRequire` 是否开启 → `require` 是否满足 → `remap` 是否写对 → 目标方法名是否随版本变化。
- 崩溃日志出现 `Mixinsub> target ... method not found` 时，通常是版本差异导致的目标签名不匹配，按对应版本的源码修正。
