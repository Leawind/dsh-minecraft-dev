---
name: mod-ecosystem-overview
description: Use when making ecosystem-level decisions in Minecraft mod development — choosing a mod loader, deciding where to publish, setting up CI auto-publishing, publishing mod documentation, or picking a multi-version development approach.
---

# Minecraft 模组开发生态速览

当前生态的方向性常识，用于技术选型与整体规划。本技能只给现状、方向与官方入口，不给操作细节——具体做法见对应技能（`mod-publish-plugin`、`stonecutter-multiversion`、`compat-troubleshooting`、`reference-mod-architecture`）。生态会变化，动手前用官方来源核对最新状态。

## 加载器现状与选择

- **NeoForge**：Forge 的社区延续（2023 年分叉），1.20.1+ 新模组的常见选择之一，活跃开发。官方：<https://neoforged.net>
- **Fabric**：轻量、更新最快（新版 MC 发布后数天内可用），性能优化类模组（Sodium、Lithium 等）的主场，生态庞大。官方：<https://fabricmc.net>
- **Forge**：老牌加载器，新开发已基本迁移到 NeoForge；1.20.1 及更早版本仍有大量存量模组。官方：<https://docs.minecraftforge.net>
- **Quilt**：Fabric 的分支，兼容多数 Fabric 模组，但自身生态小，新模组很少选择。官方：<https://quiltmc.org>
- 选择方向：新项目通常二选一（Fabric 或 NeoForge）；需要覆盖多个加载器时用多加载器方案（见下文"多版本开发方式"）。跨加载器兼容问题见 `compat-troubleshooting` 技能。

## 发布平台

Minecraft 模组的正式发布平台是 **Modrinth** 与 **CurseForge**：

- **Modrinth**：现代、API 优先、开源、无广告，社区友好，新项目常用。官方：<https://modrinth.com>
- **CurseForge**：老牌平台，用户基数大、历史模组多（Overwolf 运营）。官方：<https://www.curseforge.com/minecraft>
- 发布操作（上传、依赖解析、自动发布）见 `mod-publish-plugin` 技能。

## CI 自动发布

- 常见模式：GitHub Actions 在打 tag 或推 main 时触发构建，把各版本产物自动发布到 Modrinth / CurseForge。
- 两条主流工具链：
  - **mod-publish-plugin**（Gradle 插件，构建任务内发布）：<https://github.com/jpenilla/mod-publish-plugin>
  - **MC-Publish**（GitHub Action，独立于构建脚本）：<https://github.com/Kira-NT/mc-publish>
- 配套：changelog 从 git 提交 / tag 提炼，多版本矩阵构建见 `stonecutter-multiversion` 技能的 CI 一节。

## 文档发布

- 小项目：GitHub 仓库 README + GitHub wiki 即可起步。
- 完整文档站：GitHub Pages 搭配静态站点生成器（VitePress / Docusaurus / MkDocs），或用 ReadTheDocs。
- API 文档：javadoc 发布到文档站或 javadoc 托管服务。
- 平台页（Modrinth / CurseForge）放简介与下载链接，详细文档放文档站并互相链接。

## 多版本开发方式

- **Stonecutter**（一套源码、条件编译、多版本构建）：见 `stonecutter-multiversion` 技能；官方：<https://stonecutter.kikugie.dev>
- **多加载器模板**：common 模块 + 各加载器薄壳，如 [MultiLoader Template](https://github.com/iamkaf/multiloader-template)、[MicrocontrollersDev 的多加载器多版本模板](https://codeberg.org/MicrocontrollersDev/multiloader-multiversion-template)。
- **按版本分支**：每个 MC 版本一个分支，迁移改动大、维护成本高，大型模组（如 Lithium）采用。
- 方案选型结合项目规模与维护能力；架构设计见 `reference-mod-architecture` 技能。
