---
name: mod-publish-plugin
description: Use when publishing a Minecraft mod to Modrinth or CurseForge via the mod-publish-plugin Gradle plugin — configuring it, setting up release metadata, or troubleshooting a failed publication.
---

# mod-publish-plugin 模组发布

本文介绍使用 `me.modmuss50.mod-publish-plugin` 这个 Gradle 插件发布 Minecraft 模组的方法。它有自动解析依赖、选择平台、上传产物等能力。配置范例参考 Elytra Trims、YACL 的实际项目（见 `docs/References.md`）。

## 版本要求

`me.modmuss50.mod-publish-plugin` **必须使用 2.1.0 及以上版本**（当前最新 2.2.0，见 Gradle Plugin Portal）。版本过低可能导致缺少新平台的 API 支持。

```kotlin
plugins {
    id("me.modmuss50.mod-publish-plugin") version "2.2.0"
}
```

**注意 DSL 随大版本变化**：2.x 用 `publishMods { }` 顶层块（内嵌 `modrinth { }` / `curseforge { }`）；0.x / 1.x 用顶层 `modrinth { }` / `curseforge { }` 块（实测 Elytra Trims 3.x 用 `1.0.+`、YACL lts 用 `0.8.4`，都是旧写法）。先确认项目用的插件版本再决定写法。

## 配置要点（2.x 写法）

Elytra Trims 的真实配置（节选，Stonecutter 多版本项目）：

```kotlin
publishMods {
    val mrToken = providers.environmentVariable("MODRINTH_TOKEN")
    val cfToken = providers.environmentVariable("CURSEFORGE_TOKEN")

    // 没有配置 token 时自动进入 dry-run，不实际上传
    dryRun = (!mrToken.isPresent && publishMr) || (!cfToken.isPresent && publishCf)

    type = ReleaseType.of(sc.properties["publish.status"])   // alpha / beta / release
    file = loomx.modJar.map { it.archiveFile.get() }
    version = sc.properties.get<String>("mod.version")
    changelog = providers.fileContents { file("../../CHANGELOG.md") }.asText
    modLoaders.add("fabric")

    // game versions 直接来自 Stonecutter 版本属性，随激活版本变化
    val versions = sc.properties.raw("mod:releases").to<List<String>>()

    modrinth {
        projectId = property("publish.modrinth_id") as String
        accessToken = mrToken
        minecraftVersions.addAll(versions)
        requires("fabric-api", "fabric-language-kotlin")
    }
}
```

要点：

- `modrinth` / `curseforge` 块的 API Token 通过环境变量提供（`MODRINTH_TOKEN`、`CURSEFORGE_TOKEN`），**不要把 Token 写进仓库**；`dryRun` 在 token 缺失时自动开启可避免误上传。
- `type`：alpha / beta / release 三选一，与项目的版本阶段对应。
- `depends` / `requires` / `conflicts`：声明模组依赖与冲突，发布后平台会自动展示。
- `minecraftVersions`：单版本项目写死即可；Stonecutter 项目从版本化属性动态取。实测三种来源：版本化 gradle.properties 的 `mod.mc_targets`（空格分隔，Elytra Trims 3.x）、toml 的 `mod.releases`（`sc.properties.raw("mod:releases").to<List<String>>()`，Elytra Trims 4.x）、gradle.properties 的 `pub.stableMC` / `pub.modrinthMC`（逗号分隔，YACL）。并可在 `stonecutter.gradle.kts` 里给发布 task 排序：`stonecutter tasks { order("publishModrinth", ordering) }`（Elytra Trims 按版本号 + 加载器排序）。
- `modrinth` 块应配置 `environment`，声明模组运行在客户端还是服务端（如客户端专用模组用 `CLIENT_ONLY`），见下方可选值。
- 发布用的产物文件用 loom 的 `remapJar` 或 `modJar`（已重映射的 jar），不要传原始构建产物。

### Modrinth `environment` 可选值

| 值                              | 含义                                           |
| ------------------------------- | ---------------------------------------------- |
| `CLIENT_ONLY`                   | 全部功能在客户端侧                             |
| `SERVER_ONLY`                   | 全部功能在服务端侧，单人游戏中也生效           |
| `DEDICATED_SERVER_ONLY`         | 仅在专用服务器上运行，单人游戏中无功能         |
| `CLIENT_AND_SERVER`             | 客户端和服务端都必须安装该模组                 |
| `SERVER_ONLY_CLIENT_OPTIONAL`   | 服务端必须安装，客户端可选安装以获得额外功能   |
| `CLIENT_ONLY_SERVER_OPTIONAL`   | 客户端必须安装，服务端可选安装以获得额外功能   |
| `CLIENT_OR_SERVER_PREFERS_BOTH` | 客户端或服务端均可运行，两端都安装时有额外功能 |
| `CLIENT_OR_SERVER`              | 客户端或服务端均可运行，两端都安装时无额外功能 |
| `SINGLEPLAYER_ONLY`             | 仅在单人游戏中运行，多人游戏中无功能           |

## 故障排查

- 上传失败先确认 token 环境变量是否已设置（`dryRun` 模式下会"成功"但不实际上传，留意输出）。
- `gameVersions` 里写了平台不认的版本字符串（如 snapshot 名称）会导致校验失败，对照项目支持的版本列表核对。
- 多版本项目里某个版本发布失败，先确认该版本的版本属性（`mod.releases`）是否配置了正确的 MC 版本列表。
