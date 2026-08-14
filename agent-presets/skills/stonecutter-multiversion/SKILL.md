---
name: stonecutter-multiversion
description: Use when working on a Minecraft mod that supports multiple Minecraft versions via Stonecutter — switching the active version, writing conditional-compilation blocks, running version-specific builds, or understanding how the version matrix is configured.
---

# Stonecutter 多版本开发

Stonecutter 是一个 Gradle 插件，用一套源码为多个 Minecraft 版本生成构建产物。同一份代码中，版本差异通过条件编译块表达，构建时按目标版本切换。

## 先弄清楚项目结构

1. 读 `stonecutter.gradle.kts`，确认当前激活的 Minecraft 版本和已配置的版本集合。
2. 读 `gradle.properties` 和 `settings.gradle.kts`，了解项目是否还叠加了其他多版本工具（如 modstitch）。
3. 确认加载器（Fabric / Forge / NeoForge）——有的项目用 Stonecutter 同时管理多个加载器。

## 条件编译块的写法

### 语法

版本条件用注释块包裹，**当前版本对应的代码保持裸写**，其他版本的代码包在 `/*  */` 里：

```java
/*? if >=1.21.11 {*/
return currentVersion().dataVersion().version();
/*? } else {*/
/*return currentVersion().getDataVersion().getVersion();
 *//*? }*/
```

### 规则

- 嵌套时，符合当前条件的代码仍用 `/* */`，被注释的其他版本代码用 `/^ ^/` 表示内层条件（参考项目自身的既有写法，保持一致）。
- 需要 else / else-if 时，尽量用 `>=` 条件，不要用 `<` 或 `<=`。
- 方法体中出现条件编译且需要注释说明原因时，把原因写在该方法的 Javadoc 中，不要写在方法体里。
- 合并相邻条件编译块时，`*/` 后紧跟 `/*?`，中间不要留空白。
- 多版本差异应尽量下沉封装（例如到一个 bridge/适配层），避免业务代码到处散落版本分支——具体分层看项目架构约定。

## 切换版本与构建

- 切换激活版本前，先读项目的 AGENTS.md 或文档确认命令（常见为 `./gradlew chiseledBuild`、`stonecutter` 相关 task，或直接改 `stonecutter.gradle.kts`）。
- 多版本全量构建通常很慢：用后台任务运行，例如 `./gradlew buildAndCollect`（项目定义了该 task 时），不要阻塞对话。
- 只构建当前版本时用普通 `./gradlew build`。
- 构建前先运行项目的架构/格式检查任务（如 `checkArchitecture`）——多版本项目常把它挂进 `check`。

## 验证

- 改动条件编译块后，至少构建目标版本验证通过；如果可能，构建相邻的另一个版本，确认两个分支都正确。
- 留意 `/*? if ... */` 条件写反导致的"两个版本都走了同一分支"问题：切版本后重新构建是最可靠的验证。
