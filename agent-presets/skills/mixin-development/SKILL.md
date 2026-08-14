---
name: mixin-development
description: Use when writing or modifying Mixins in a Minecraft mod — choosing an injection point, following compatibility-safe practices, or fixing Mixin-related crashes.
---

# Mixin 开发

Mixin 是 Minecraft 模组修改原版行为的主要手段。Mixin 用不好会与其他模组冲突甚至崩溃，因此选择注入点和注入方式时要保守。

## 核心规则

### 禁止 `@Redirect`

**同一个调用点被两个模组 Redirect 是常见崩溃源。** 优先使用侵入性更低的注入方式：

| 注入方式                            | 适用场景                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| `@Inject`（HEAD / RETURN / 特定行） | 在方法前、后或某处插入代码，不改返回值                       |
| `@ModifyVariable`                   | 修改方法参数或局部变量                                       |
| `@WrapOperation`                    | 包装某次调用，可以替换调用或修改结果（比 Redirect 兼容性好） |
| `@Redirect`                         | **避免使用**，尤其是公共 API 的调用点                        |

### 其他兼容性要点

- 目标方法签名随 Minecraft 版本变化，尽量把目标字符串抽成常量并用条件编译区分版本（见 `stonecutter-multiversion` 技能）。
- 只在必要时使用 `@Overwrite` / `@Unique`，并清楚标注。
- 注入点选在"被调用频率低、语义稳定"的地方；不要在热路径或每帧调用处做重活。

## 编写步骤

1. 确认目标类在目标 Minecraft 版本中的真实类名、方法名、签名（参考该版本的反编译源码或 mappings）。
2. 选择侵入性最低的注入方式：`@Inject` 能解决的不要用 `@WrapOperation`，`@WrapOperation` 能解决的不要用 `@Redirect`。
3. 使用 `@At` 指定精确位置；`RETURN` 注入时注意 `CallbackInfo` 的使用（`cancellable` 只在需要取消时开启）。
4. 在 mixins 配置（如 `modid.mixins.json`）中注册 mixin 类，注意包名和 `client` / `common` 的归属。

## 验证

- 运行游戏到注入点实际触发的位置（不只是启动），确认注入逻辑生效且无报错。
- 如果可能，与另一个也修改同类的模组一起测试，验证无冲突。
- 崩溃日志中出现 `Mixinsub> target ... method not found` 时，通常是版本差异导致的目标签名不匹配，按对应版本的源码修正。
