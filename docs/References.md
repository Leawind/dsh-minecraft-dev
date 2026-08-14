# 参考模组源码仓库

通过 [Modrinth](https://modrinth.com) 筛选出的开源模组源码仓库清单。

## 使用说明

- 多版本模组通常按 Minecraft 版本建立分支，如 lithium 的 `1.21.4`、YACL 的 `lts`、Elytra Trims 的 `4.x-26.3`（无 main 分支）。
- 优先参考：Elytra Trims（Stonecutter 作者亲作）、Lithium（Mixin 大规模实战）、Mod Menu（中小型清晰结构）。

## Mixin 开发参考

### Lithium

- Modrinth：<https://modrinth.com/mod/lithium>
- 源码：<https://github.com/CaffeineMC/lithium>（原 caffeinemc/lithium-fabric 已迁移）

大型优化模组，包含数百个 Mixin，是学习非侵入注入（`@Inject`、`@ModifyVariable`、MixinExtras 的 `@WrapOperation` / `@ModifyExpressionValue` 等）、接口扩展、按版本分支写法的实战范例。mixin 配置由 `@MixinConfigOption` 注解 + 构建期插件自动生成，也值得参考。

### Fabric API

- Modrinth：<https://modrinth.com/mod/fabric-api>
- 源码：<https://github.com/FabricMC/fabric>

Mixin 规范与事件系统实现的参考，多版本分支管理范例。

### Immersive Portals

- Modrinth：<https://modrinth.com/mod/immersiveportals>
- 源码：<https://github.com/qouteall/ImmersivePortalsMod>

高难度 Mixin 与渲染挂钩的参考（`priority` 压过其他模组、`IMixinConfigPlugin` 按 mod 加载禁用冲突 Mixin），适合进阶学习。

### Mod Menu

- Modrinth：<https://modrinth.com/mod/modmenu>
- 源码：<https://github.com/TerraformersMC/ModMenu>

体量适中、结构清晰的入门级参考，全项目仅 3 个 client Mixin，适合学习小型模组的项目组织方式。

## Stonecutter 多版本参考

### Elytra Trims

- Modrinth：<https://modrinth.com/mod/elytra-trims>
- 源码：<https://codeberg.org/KikuGie/elytra-trims>

Stonecutter 作者 kikugie 自己的模组，Stonecutter 0.9.7 + `versions.json` + `stonecutter.properties.toml` + 按加载器拆 `build.fabric.gradle.kts` / `build.neoforge.gradle.kts`，同时使用 mod-publish-plugin 2.x 发布，是这两项技能最权威的实战范例。

### YACL（YetAnotherConfigLib）

- Modrinth：<https://modrinth.com/mod/yacl>
- 源码：<https://github.com/isXander/YetAnotherConfigLib>

配置库模组，用 Stonecutter + modstitch 管理 Fabric/NeoForge 多加载器多版本：`versions/<版本>-<加载器>/gradle.properties` 每版本一份属性，`modstitch.ct` 是加载器无关的 class tweaker（AW v1 格式）。

### HCsCR

- Modrinth：<https://modrinth.com/mod/hcscr>
- 源码：<https://github.com/VidTu/HCsCR>

开源优化模组，Stonecutter 处理 `java` + `json`（mixin 配置也能条件编译），发布走 GitHub Releases 而非 mod-publish-plugin（可作反例参考）。

## 多加载器大型工程参考

### Botania

- Modrinth：<https://modrinth.com/mod/botania>
- 源码：<https://github.com/VazkiiMods/Botania>

Xplat + Fabric + Forge 三模块结构：Xplat 存公共代码并直接编入各加载器 jar，跨平台抽象用 Java ServiceLoader，代码组织公认优秀。

### Applied Energistics 2

- Modrinth：<https://modrinth.com/mod/ae2>
- 源码：<https://github.com/AppliedEnergistics/Applied-Energistics-2>

单 Gradle 工程，多加载器靠同仓库独立分支/发布线（fabric / forge tag 并存），main 分支的 buildSrc 放构建插件，复杂事件/网络/渲染体系参考。

### Create

- Modrinth：<https://modrinth.com/mod/create>
- 源码：<https://github.com/Creators-of-Create/Create>

单工程维护，入口统一为 `Create` / `CreateClient`，公共/平台边界靠包结构（content/foundation/infrastructure），多加载器支持靠独立分支或仓库。

### Sodium

- Modrinth：<https://modrinth.com/mod/sodium>
- 源码：<https://github.com/CaffeineMC/sodium>

渲染引擎级超大工程，渲染相关 Mixin 与性能优化参考。

### Continuity

- Modrinth：<https://modrinth.com/mod/continuity>
- 源码：<https://github.com/PepperCode1/Continuity>

资源包/模型加载参考（Fabric API ResourceLoader + 模型修饰器）；NeoForge 版靠同一套 Fabric 源码 + Sinytra Connector 双元数据运行，与兼容性排查相关。
