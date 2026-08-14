---
name: reference-mod-architecture
description: Use when designing or restructuring a Minecraft mod's architecture — deciding module layout, multi-loader strategy, cross-platform abstraction, entry points, or build/publish structure — and when you want to ground the decision in how proven open-source mods are organized.
---

# 参考模组架构

设计模组架构时，先明确目标与约束（第一性原理），再对照优秀模组的实际结构——例子是证据，不是理由。本技能给出：架构决策的第一性原理、真实模式清单（每条含原理、具体例子、适用条件）、以及按场景选型的决策表。完整参考仓库清单见 `docs/References.md`。

## 第一性原理

架构决策的目标是让长期代价最小化：

- **平台差异最小化**：公共代码最大化、平台代码最小化；加载器（Fabric / Forge / NeoForge）差异的暴露面越小，测试与维护成本越低。
- **依赖方向单一**：公共层不依赖任何加载器；加载器只依赖公共层。
- **构建复杂度 vs 运行时复杂度的权衡**：多模块构建用"构建期复杂"换"运行时纯净"；单工程+分支用"构建简单"换"维护重复"。
- **单一事实来源**：版本、依赖、发布元数据只在一处声明，随版本变化的部分版本化。
- **可组合性优先**：扩展点/注入点选可组合的方案——修改面越窄，与其他模组重叠冲突的风险越低。

## 真实模式（原理 → 具体例子 → 何时用）

### 模式 1：公共代码并入各加载器 jar（Botania）

- **原理**：每个加载器产物都包含同一份公共代码，公共层零运行时依赖。
- **实现**：Xplat 模块仅 `compileOnly`，loader 模块用 `compileJava { source(project(":Xplat").sourceSets.main.allSource) }` + `processResources { from Xplat 资源 }` 把公共源码编进各自 jar。
- **何时用**：需要给多个加载器提供完全相同的公共代码，且不想引入运行时依赖。

### 模式 2：ServiceLoader 跨平台抽象（Botania）

- **原理**：平台差异点用 JDK 标准机制抽象成"接口 + 每加载器一个实现"，公共代码永远不 import 加载器 API。
- **实现**：Xplat 定义 `XplatAbstractions` 接口 + `ServiceUtil.findService(...)`；Fabric / Forge 模块在 `META-INF/services/` 注册 `FabricXplatImpl` / `ForgeXplatImpl`。
- **何时用**：平台行为差异点少而稳定（几个到十几个）。差异点很多时单接口会膨胀，按领域拆多个接口。

### 模式 3：入口薄壳（AE2 / Create）

- **原理**：初始化逻辑留在公共代码，加载器入口只回答"在哪个加载器上启动"——入口类几乎不包含业务。
- **实现**：AE2 的 `AppEngServerStartup implements DedicatedServerModInitializer`，`onInitializeServer()` 里只 `new AppEngServer()`；Create 的 `Create`（`@Mod`）+ `CreateClient`（`@Mod dist = CLIENT`）。
- **何时用**：任何多加载器项目都适用——入口应永远保持薄。

### 模式 4：单工程+分支 vs 多模块 vs Connector

三种真实形态：

- **多模块**（Botania）：Xplat / Fabric / Forge 同仓同分支。
- **单工程+分支/tag**（AE2）：同仓库内每条加载器一条发布线（`refs/tags/forge/v15.4.10` 与 `fabric/v11.7.6` 并存），源码基本不共享。
- **Fabric 源码 + Sinytra Connector 双元数据**（Continuity）：NeoForge 版直接跑同一套 Fabric 代码，靠 connector 适配。

**决策（按第一性原理）**：

- 维护者少、想共享最大代码 → 多模块（代价：构建系统复杂）。
- 各加载器维护节奏独立、代码差异大 → 分支（代价：公共修复要重复合）。
- Fabric 生态优先、NeoForge 只是附带 → Connector（代价：依赖 connector 的适配质量）。

### 模式 5：包结构分层（Create）

- **原理**：按依赖方向分层——基础设施不依赖游戏内容，游戏内容依赖基础设施。
- **实现**：`content`（游戏内容）/ `foundation`（加载器无关基础设施：mixin、model、networking、data、events）/ `infrastructure`（命令、配置、datagen、worldgen）。
- **何时用**：中型以上模组；小模组按需分层即可，不必照搬。

### 模式 6：元数据模板生成（AE2 / Create）

- **原理**：加载器元数据（`neoforge.mods.toml` 等）含版本等变量，手写多份易失同步；模板 + 构建期展开保证单一来源。
- **实现**：`src/main/templates/META-INF/neoforge.mods.toml` 占位符模板 + `generateModMetadata`（ProcessResources expand）+ `sourceSets.main.resources.srcDir(generateModMetadata)` + `neoForge.ideSyncTask(...)`。
- **何时用**：多版本/多加载器项目；单版本单加载器手写即可。

### 模式 7：mixin 配置生成（Lithium）

- **原理**：mixin 数量上百后，手写配置文件与代码、文档三处失同步；生成让"配置 = 代码上的注解"成为单一来源。
- **实现**：`@MixinConfigOption`（写在 mixin 包的 `package-info.java` 上）+ 构建期扫描生成 mixins.json。
- **何时用**：仅当 mixin 数量达到数百级；小项目手写配置文件更简单。

### 模式 8：按版本分支 vs 条件编译（Lithium vs Elytra Trims）

- **原理**：差异的"结构"决定工具——结构性大差异（大版本重构）用分支，点状差异（签名、方法名微调）用条件编译。
- **具体做法**：条件编译用 Stonecutter——版本差异写在 `//? if <版本> { ... }` 条件块里，构建时按激活版本取舍；结构性大差异仍走分支。

### 模式 9：发布管线形态（Elytra Trims vs HCsCR）

- **原理**：发布自动化程度应与平台覆盖成正比——平台越多，越值得自动化；只出一个 Release 就不需要插件。
- **实现**：Elytra Trims = CI 矩阵 + mod-publish-plugin 遍历发布（Modrinth / CurseForge）；HCsCR = GitHub Releases + `updater_*.properties`（给 ModMenu 做版本检查）。
- **何时用**：需要多平台发布 → mod-publish-plugin；只想出 GitHub Release → 简单脚本即可。

### 模式 10：内置资源包（Continuity）

- **原理**：可选的资源（默认纹理替换等）作为"内置包"而非强制资源，用户可在资源包界面开关。
- **实现**：`ResourceLoader.registerBuiltinPack(...)`，包体在 `src/main/resources/resourcepacks/`。
- **何时用**：模组自带可选资源、且希望用户可关闭时。

## 从参考仓库学习架构：读什么

按关心的问题选仓库，按此顺序读一个项目的结构：

1. `settings.gradle(.kts)` —— 模块划分 / 复合构建 / 版本矩阵。
2. 构建脚本 —— 依赖、插件、发布、版本号拼接。
3. 入口类 —— 初始化如何进入公共代码（模式 3 的体现）。
4. 平台实现目录 —— 差异点在哪里、如何抽象（模式 2 的体现）。
5. mixin 配置 —— mixin 组织与运行端分组。

## 选型决策

| 情况                       | 推荐                                                                                       |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| 单加载器、中小型           | 单工程，入口薄壳，包结构按依赖方向分层                                                     |
| 多加载器、共享代码多               | 多模块（Xplat + ServiceLoader）或 Stonecutter + modstitch                                      |
| 多加载器、各加载器独立演进 | 单工程 + 分支                                                                              |
| Fabric 优先、NeoForge 附带 | Fabric 源码 + Connector 双元数据                                                           |
| mixin 上百个               | 配置生成（`@MixinConfigOption`）                                                           |
| 多平台发布                    | mod-publish-plugin                                                                       |
