---
name: mod-ecosystem-overview
description: Use when making ecosystem-level decisions in Minecraft mod development — choosing a mod loader, license, or source hosting; deciding where to publish; setting up CI auto-publishing; publishing mod documentation; or picking a multi-version development approach.
---

# Minecraft 模组开发生态速览

当前生态的方向性常识，用于技术选型与整体规划。本技能只给现状、方向与官方入口，不给操作细节；内容以官方来源为准。生态会变化，动手前用官方来源核对最新状态。

## 加载器现状与选择

- **NeoForge**：Forge 的社区延续（2023 年分叉），1.20.1+ 新模组的常见选择之一，活跃开发。官方：<https://neoforged.net>
- **Fabric**：轻量、更新最快（新版 MC 发布后数天内可用），性能优化类模组（Sodium、Lithium 等）的主场，生态庞大。官方：<https://fabricmc.net>
- **Forge**：老牌加载器，新开发已基本迁移到 NeoForge；1.20.1 及更早版本仍有大量存量模组。官方：<https://docs.minecraftforge.net>
- **Quilt**：Fabric 的分支，兼容多数 Fabric 模组，但自身生态小，新模组很少选择。官方：<https://quiltmc.org>
- 选择方向：新项目通常二选一（Fabric 或 NeoForge）；需要覆盖多个加载器时用多加载器方案（见下文"多版本开发方式"）。跨加载器兼容需要额外测试与适配。

## 发布平台

Modrinth 与 CurseForge 是使用最广的两个**第三方**发布平台。

- **Modrinth**：现代、API 优先、开源、社区友好，新项目常用；商业化运营，有广告。官方：<https://modrinth.com>
- **CurseForge**：老牌平台，用户基数大、历史模组多（Overwolf 运营）。官方：<https://www.curseforge.com/minecraft>
- 曝光与用户习惯因平台而异：CurseForge 用户基数大但政策与体验老旧，Modrinth 面向新生态；不少模组两个平台都发布。

## CI 自动发布

- 常见模式：GitHub Actions 在打 tag 或推 main 时触发构建，把各版本产物自动发布到 Modrinth / CurseForge。
- 两条主流工具链：
  - **mod-publish-plugin**（Gradle 插件，构建任务内发布）：<https://github.com/jpenilla/mod-publish-plugin>
  - **MC-Publish**（GitHub Action，独立于构建脚本）：<https://github.com/Kira-NT/mc-publish>
- 配套：changelog 从 git 提交 / tag 提炼；多加载器多版本用矩阵构建（每个加载器/版本一个构建任务，产物分别发布）。

## 文档发布

- **ModdedMC Wiki** 是模组文档的社区平台：文档以 Markdown 形式放在项目的 Git 仓库里，通过开发者门户发布到 wiki：<https://moddedmc.wiki>（作者指南：<https://docs.moddedmc.wiki>）。
- 前提：项目公开托管在 Modrinth 或 CurseForge 上，且仓库地址已填为项目的源码链接（用于所有权验证；登录使用 GitHub 账号）。
- 小项目也可用 GitHub 仓库 README / wiki 起步，或自建文档站（GitHub Pages + VitePress / Docusaurus / MkDocs、ReadTheDocs）。

## 源码管理方式

- 主流是公开托管在 **GitHub**；**Codeberg**（Gitea 系、注重自由软件与隐私）也有使用（如 Elytra Trims 托管在 Codeberg）。
- 公开源码利于社区信任与协作（issue、PR、贡献），许多知名模组开源；闭源则保护代码但失去社区参与。

## 许可证

- 模组需遵守 [Minecraft 最终用户许可协议](https://www.minecraft.net/en-us/eula)、[Minecraft 使用准则](https://www.minecraft.net/en-us/usage-guidelines)。
- 开源模组常用宽松许可证：**MIT**、**Apache-2.0**、**LGPL**、**MPL-2.0** 等；闭源或 **ARR**（All Rights Reserved）则保留全部权利。
- 基于开源模板/库开发时，注意其许可证的兼容与归属要求（保留版权声明、传染性等）。

## 多版本开发方式

- **Stonecutter**（一套源码、条件编译、多版本构建）：<https://stonecutter.kikugie.dev>
- **多加载器模板**：common 模块 + 各加载器薄壳，如 [MultiLoader Template](https://github.com/iamkaf/multiloader-template)、[MicrocontrollersDev 的多加载器多版本模板](https://codeberg.org/MicrocontrollersDev/multiloader-multiversion-template)。
- **按版本分支**：每个 MC 版本一个分支，迁移改动大、维护成本高，大型模组（如 Lithium）采用。
- 方案选型结合项目规模与维护能力。
