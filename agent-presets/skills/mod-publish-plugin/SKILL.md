---
name: mod-publish-plugin
description: Use when publishing a Minecraft mod to Modrinth or CurseForge via the mod-publish-plugin Gradle plugin — configuring it, setting up release metadata, or troubleshooting a failed publication.
---

# mod-publish-plugin 模组发布

本文介绍使用 `me.modmuss50.mod-publish-plugin` 这个 Gradle 插件发布 Minecraft 模组的方法。它有自动解析依赖、选择平台、上传产物等能力。

## 版本要求

`me.modmuss50.mod-publish-plugin` **必须使用 2.1.0 及以上版本**。版本过低可能导致缺少新平台的 API 支持。

在 `build.gradle.kts` 中：

```kotlin
plugins {
    id("me.modmuss50.mod-publish-plugin") version "2.2.0"
}
```

## 配置要点

- `modrinth` / `curseforge` 块需要各自的 API Token，通常通过环境变量提供（如 `MODRINTH_TOKEN`、`CURSEFORGE_TOKEN`），不要把 Token 写进仓库。
- `modrinth` 块应配置 `environment`，声明模组运行在客户端还是服务端（如客户端专用模组用 `CLIENT_ONLY`），见下方可选值。
- `versionType`：alpha / beta / release 三选一，与项目的版本阶段对应。
- `depends` / `conflicts`：声明模组依赖与冲突，发布后平台会自动展示。

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
