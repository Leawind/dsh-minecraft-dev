---
name: mixin-development
description: Use when writing or modifying Mixins in a Minecraft mod — choosing an injection point, following compatibility-safe practices, or fixing Mixin-related crashes.
---

# Mixin 开发

Mixin 是 Minecraft 模组修改原版行为的主要手段。Mixin 用不好会与其他模组冲突甚至崩溃，因此选择注入点和注入方式时要保守。以下内容来自 Lithium、Fabric API、Mod Menu、Immersive Portals 等开源模组的实际源码，可对照参考（见 `docs/References.md`）。

## 核心规则

### 优先非侵入注入，避免 `@Redirect`

**同一个调用点被两个模组 Redirect 是常见崩溃源。** 优先使用侵入性更低的注入方式：

| 注入方式                                | 适用场景                                                         |
| --------------------------------------- | ---------------------------------------------------------------- |
| `@Inject`（HEAD / RETURN / 特定锚点）   | 在方法前、后或某处插入代码，不改返回值                           |
| `@ModifyVariable`                       | 修改方法参数或局部变量                                           |
| `@ModifyArg`                            | 只改某次调用的一个实参（如 Mod Menu 调整按钮高度）               |
| `@ModifyExpressionValue`（MixinExtras） | 改一个布尔/值表达式的结果，不改控制流（Lithium 大量使用）        |
| `@WrapOperation`（MixinExtras）         | 包住一次调用，前后加逻辑或改结果；比 Redirect 兼容性好           |
| `@Local`（MixinExtras）                 | 在 `@Inject` 回调里直接取目标方法的局部变量                      |
| `@Redirect`                             | **避免使用**；确需"完全接管一次调用"时才用（如 IP 接管碰撞计算） |

`@WrapOperation`、`@ModifyExpressionValue`、`@Local` 来自 **MixinExtras**，不是原生 Mixin 注解。Fabric Loader 0.15.x+ 已内置传递提供（`mixinextras-fabric`，1.21.4 时代内置 0.4.1），无需显式声明；需要显式声明时坐标是 `io.github.llamalad7:mixinextras-fabric`。

### 必须遵守的兼容性细节

- 非 Minecraft 类的调用点（`java.*`、`com.google.*`、`org.joml.*` 等）在 `@At` 的 `target` 里**必须写 `remap = false`**，否则启动时报 target not found（Lithium、IP 都是这么写的）。
- 目标方法名：yarn 环境直接写 yarn 名（如 `"tick"`）；有重载歧义或 mojmap 环境写完整描述符（如 `"collide(Lnet/minecraft/world/phys/Vec3;)Lnet/minecraft/world/phys/Vec3;"`）。
- `method` 可以给字符串数组一次注入多个方法；**`require = N` 强制要求匹配 N 处**，防止反混淆后注入点静默失效（Lithium 的 `require = 4` 范例）。
- `@Overwrite` 换掉整个方法实现时必须写 `@author` / `@reason` Javadoc（Fabric 审核要求，Lithium 的 MthMixin 是范例）；能不用 `@Overwrite` 就不用。
- 自创成员一律 `@Unique`；给原版类"加能力"时让 Mixin `implements` 自定义接口，接口方法带 `modid_` 前缀（Fabric 官方 `fabric_`、Lithium `lithium$`、IP 的 `IE*` 命名），防止与其他模组冲突。
- 注入点选在"被调用频率低、语义稳定"的地方；不要在热路径或每帧调用处做重活。

## mixin 配置文件（`modid.mixins.json`）

参考 Mod Menu、Fabric API、Lithium 的实际配置：

- `package`：Mixin 类所在包。
- `required: true` + `compatibilityLevel`（`JAVA_17` / `JAVA_21`）：基础要求。
- `minVersion`：可选（Mod Menu 用 `"0.8"`；较新的 Lithium 已去掉它）。
- `injectors.defaultRequire: 1`：注入点必须匹配，不匹配启动即报错——**排查"注入没生效"先看这个**。
- `injectors.maxShiftBy`：允许 `At.Shift` 偏移的指令数（Fabric API 用 `3`）。
- `mixins` / `client` / `server` 三分组：按运行端分发，客户端专用 Mixin 放 `client`，服务端放 `server`。
- `plugin`：`IMixinConfigPlugin` 实现类，可在运行时按条件启用/禁用单个 Mixin（见下）。
- `overwrites.conformVisibility`：让 `@Overwrite` 与目标方法可见性一致（Lithium fabric 配置里有）。

## 实用模式（来自实际源码）

- **接口 Mixin**：目标是接口时，Mixin 也写成 `interface`，`@Shadow` 方法写成抽象方法、注入逻辑写成 `default` 方法（Lithium 的 BlockGetterMixin），避免强转。
- **换静态数据结构**：`@Shadow @Final @Mutable` 字段 + 在 `<clinit>` 的 `@At("RETURN")` 注入后改写（Lithium 替换 `Mth.SIN`）。
- **打匿名内部类**：外层 Mixin 类里嵌套 `static class` + `@Mixin(targets = "全限定名$1")`（Lithium 打 `CompoundTag$1`）。
- **锚定 lambda 生成的方法**：`@Shadow(aliases = "lambda$...")`，因为 lambda 编译名不稳定（Lithium）。
- **精确锚点**："在某字段写入之后" = `@At(value = "FIELD", target = "Lnet/...;字段名:Z", opcode = Opcodes.PUTFIELD, ordinal = 0, shift = At.Shift.AFTER)`（Fabric API 的 ServerWorldMixin）。
- **压过其他模组的注入顺序**：`@Mixin(value = X.class, priority = 900)`（IP 对 ServerGamePacketListenerImpl 的用法）。
- **按其他模组是否加载禁用冲突 Mixin**：`IMixinConfigPlugin.shouldApplyMixin` 里检查 `FabricLoader.getInstance().isModLoaded(...)`，冲突时返回 `false`（IP 对 porting_lib 的处理）。

## 版本差异处理

目标方法签名随 Minecraft 版本变化。两种主流做法：

- **按 MC 版本分 git 分支**（Lithium：`1.20.1`、`1.21.4`、`26.1.x` 等各一条分支），跨版本少量差异用平台 API 判断 + 独立 `*Mixin` 变体。
- **条件编译**（Stonecutter），同一份源码里用条件块区分版本——见 `stonecutter-multiversion` 技能。

## 编写步骤

1. 确认目标类在目标 Minecraft 版本中的真实类名、方法名、签名（参考该版本的反编译源码或 mappings；多版本项目记得切到对应分支）。
2. 选择侵入性最低的注入方式：`@Inject` 能解决的不要用 `@WrapOperation`，`@WrapOperation` 能解决的不要用 `@Redirect`。
3. 使用 `@At` 指定精确位置；`RETURN` 注入时注意 `CallbackInfo` 的使用（`cancellable` 只在需要取消时开启）。
4. 在 mixin 配置文件（如 `modid.mixins.json`）中注册 mixin 类，注意包名和 `client` / `server` 的归属。

## 验证

- 运行游戏到注入点实际触发的位置（不只是启动），确认注入逻辑生效且无报错。
- 给 run 配置加 `-Dmixin.debug.export=true`（Elytra Trims 就是这么配的）可导出反编译的目标方法，核对注入点。
- 注入点"没生效"时先查 `injectors.defaultRequire` 是否开启、`require` 是否满足、`remap` 是否写对。
- 崩溃日志出现 `Mixinsub> target ... method not found` 时，通常是版本差异导致的目标签名不匹配，按对应版本的源码修正。
