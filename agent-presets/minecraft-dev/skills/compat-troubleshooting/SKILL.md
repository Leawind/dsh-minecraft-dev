---
name: compat-troubleshooting
description: Use when diagnosing a Minecraft mod compatibility problem — a crash, rendering glitch, or behavior conflict that appears only with other mods installed. Guides inspecting reference mod sources, isolating the conflict, and finding the root cause.
---

# 模组兼容性排查

模组兼容性问题通常只在多个模组共存时出现，单模组测试无法复现。排查的核心是**缩小范围、对照源码、定位冲突点**。排查手段以"冲突的产生机制"为纲（多个模组对同一目标的重叠修改），文中提到的开源模组（Lithium、Immersive Portals 等，见 `docs/References.md`）是印证实例，不是规则来源。

## 排查流程

### 1. 复现与隔离

- 先确认问题只在特定模组组合下出现：逐次移除模组，二分定位到"少了哪个模组问题就消失"。
- 记录完整崩溃日志（`crash-reports/` 和 `logs/latest.log`），从中找同时被多个模组修改的类或方法。
- 区分问题类型：崩溃 / 渲染异常 / 行为冲突，处理方式不同。

### 2. 查看相关模组的源码

**排查模组间兼容性时，对照相关模组的源码比猜更快**。源码的获取方式取决于用户环境与偏好，以用户为准：

- 用 `grep` 在双方源码中搜索同一个被修改的目标（崩溃日志里点名的类/方法）。
- 多版本模组通常按 MC 版本分 git 分支（Lithium 的 `1.21.4`、YACL 的 `lts` 等）——**必须切到与问题相同的版本分支再对照**，否则目标签名对不上。

### 3. 对照源码定位冲突

- **Mixins**：检查双方是否注入同一个目标方法。`@Redirect` 撞车是高频崩溃源；`@Inject` 一般共存但可能顺序敏感。查找日志中的 mixin apply 记录。
  - **注入顺序**由 `@Mixin(priority = N)` 决定——对方模组用高 priority（如 IP 的 `900`）压过你的注入时，先挂的注入可能被覆盖。
  - **注入点静默失效**：先查双方 mixin 配置的 `injectors.defaultRequire`（`1` 时不匹配会直接报错；`0` 时不匹配静默跳过，问题更隐蔽）。给 run 配置加 `-Dmixin.debug.export=true`（Elytra Trims 的做法）可导出反编译的目标方法，核对双方锚点。
  - **MixinExtras**（`@WrapOperation` / `@ModifyExpressionValue`）由 Fabric Loader 内置提供：双方都用 MixinExtras 包装同一调用点时通常能共存，比双 `@Redirect` 安全得多；但一方 MixinExtras、一方原生 `@Redirect` 时仍可能冲突。
  - **运行时禁用冲突 Mixin**：原理是"失败关闭式兼容"——与其在运行时与对方模组的行为冲突，不如在加载期检测到对方后跳过自己的冲突注入。做法：`IMixinConfigPlugin.shouldApplyMixin` 里检查 `isModLoaded("porting_lib")`，冲突时返回 `false`（IP 的实例）。
- **事件**：双方是否监听同一事件且都改动同一状态。
- **资源/模型/渲染**：双方是否覆盖同一资源路径或模型。
- **网络包**：双方是否注册了同 ID 的数据包。

### 4. 验证修复

- 修复后同时装两个模组实测，确认问题消失且无新问题。
- 多版本项目注意：冲突可能只出现在特定 MC 版本组合，按项目支持的版本分别验证。

## 高频冲突模式速查

| 现象                       | 常见根因                                                |
| -------------------------- | ------------------------------------------------------- |
| 启动崩溃，日志点名同一方法 | 双方 Mixin 注入同一点，尤其 `@Redirect`                 |
| 仅装有某模组时渲染异常     | 资源/模型覆盖冲突，或渲染管线 hook 冲突                 |
| 特定交互时崩溃             | 双方事件监听顺序/状态修改冲突                           |
| 注入"没生效"但无报错       | 某方 `defaultRequire` 为 0 静默跳过，或 priority 被覆盖 |
| 多版本下只有某版本崩溃     | 版本差异导致的目标签名不匹配（按该版本源码修正）        |

## 原则

- 除非确有必要，不要通过"给特定模组写专用兼容代码"来修问题——优先找通用的、对双方都安全的改法。
- 对方模组有高 priority 注入时，不要盲目调高自己的 priority 对抗；先确认是否真的需要抢同一注入点。
