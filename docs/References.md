# 参考模组源码仓库

通过 [Modrinth](https://modrinth.com) 筛选出的开源模组源码仓库清单。

## 使用说明

- 多版本模组通常按 Minecraft 版本建立分支，如 lithium 的 `dev/1.21.4`、YACL 的 `1.21`
- 优先参考：Elytra Trims（Stonecutter 作者亲作）、Lithium（Mixin 大规模实战）、Mod Menu（中小型清晰结构）。

## Mixin 开发参考

### Lithium

- Modrinth：<https://modrinth.com/mod/lithium>
- 源码：<https://github.com/caffeinemc/lithium-fabric>

大型优化模组，包含数千个 Mixin，是学习非侵入注入（`@Inject`、`@ModifyVariable` 等）、按版本分支写法、热路径优化的实战范例。

### Fabric API

- Modrinth：<https://modrinth.com/mod/fabric-api>
- 源码：<https://github.com/FabricMC/fabric>

Mixin 规范与事件系统实现的参考，多版本分支管理范例。

### Immersive Portals

- Modrinth：<https://modrinth.com/mod/immersiveportals>
- 源码：<https://github.com/qouteall/ImmersivePortalsMod>

高难度 Mixin 与渲染挂钩的参考，适合进阶学习。

### Mod Menu

- Modrinth：<https://modrinth.com/mod/modmenu>
- 源码：<https://github.com/TerraformersMC/ModMenu>

体量适中、结构清晰的入门级参考，适合学习小型模组的项目组织方式。

## Stonecutter 多版本参考

### Elytra Trims

- Modrinth：<https://modrinth.com/mod/elytra-trims>
- 源码：<https://codeberg.org/KikuGie/elytra-trims>

Stonecutter 作者 kikugie 自己的模组，同时使用 Stonecutter 条件编译（`stonecutter.gradle.kts`、`stonecutter.eval`）与 mod-publish-plugin 发布，是这两项技能最权威的实战范例。

### YACL（YetAnotherConfigLib）

- Modrinth：<https://modrinth.com/mod/yacl>
- 源码：<https://github.com/isXander/YetAnotherConfigLib>

配置库模组，用 Stonecutter + modstitch 管理 Fabric/NeoForge 多加载器多版本（根目录含 `stonecutter.gradle.kts`、`modstitch.ct`、`versions/`）。

### HCsCR

- Modrinth：<https://modrinth.com/mod/hcscr>
- 源码：<https://github.com/VidTu/HCsCR>

开源优化模组，`stonecutter.gradle.kts` 实战案例。

## 多加载器大型工程参考

### Botania

- Modrinth：<https://modrinth.com/mod/botania>
- 源码：<https://github.com/VazkiiMods/Botania>

老牌大型模组，Fabric/Forge/Quilt 多加载器，代码组织公认优秀。

### Applied Energistics 2

- Modrinth：<https://modrinth.com/mod/ae2>
- 源码：<https://github.com/AppliedEnergistics/Applied-Energistics-2>

用 `buildSrc` 拆分模块的多加载器大型工程，复杂事件/网络/渲染体系参考。

### Create

- Modrinth：<https://modrinth.com/mod/create>
- 源码：<https://github.com/Creators-of-Create/Create>

Fabric/Forge/NeoForge 多加载器大型模组。

### Sodium

- Modrinth：<https://modrinth.com/mod/sodium>
- 源码：<https://github.com/CaffeineMC/sodium>

渲染引擎级超大工程，渲染相关 Mixin 与性能优化参考。

### Continuity

- Modrinth：<https://modrinth.com/mod/continuity>
- 源码：<https://github.com/PepperCode1/Continuity>

资源包/模型加载相关，与兼容性排查（资源路径冲突）相关。
