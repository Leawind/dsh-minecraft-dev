---
name: stonecutter-multiversion
description: Use when working on a Minecraft mod that supports multiple Minecraft versions via Stonecutter — switching the active version, writing conditional-compilation blocks, running version-specific builds, or understanding how the version matrix is configured.
---

# Stonecutter 多版本开发

Stonecutter 是一个 Gradle 插件，用一套源码为多个 Minecraft 版本生成构建产物。同一份代码中，版本差异通过条件编译块表达，构建时按目标版本切换。本文描述的是 Stonecutter 的实际行为——语法与配置形态是工具本身的规则，不以某项目为准；Elytra Trims（Stonecutter 作者 kikugie 的模组）、YACL、HCsCR 等真实项目（见 `docs/References.md`）用于验证各版本的实际写法。

## 先弄清楚项目结构

1. 读 `settings.gradle.kts` 和 `stonecutter.gradle.kts`，确认 Stonecutter 插件版本与激活方式。插件 id 恒为 `dev.kikugie.stonecutter`，实测版本：0.8.2（YACL lts）、0.9.2（Elytra Trims 3.x）、0.9.7（Elytra Trims 4.x、HCsCR）：
   - 0.8–0.9.2 时代：`kotlinController = true` + `centralScript = "build.gradle.kts"`，版本内嵌在 settings 的 `shared { ... }` 里。
   - 0.9.7 时代：可用 `stonecutter active "<版本>-<加载器>"` 或 `active file(...)`，版本清单外置（见下）。
   - 直接指定：`stonecutter active "26.3-fabric"`（HCsCR、Elytra Trims 4.x）。
   - 指向文件：`stonecutter active file("versions/current")`（YACL，`versions/current` 里写当前版本名）。
2. 版本矩阵的存放位置随 Stonecutter 版本而变，**不要假设存在 `versions.properties`（已过时）**，按实际情况找：
   - `versions.json`（Elytra Trims 0.9.x）：`{"vcs": "26.1-fabric", "versions": ["26.1-fabric:26.1", "26.1-neoforge:26.1", ...]}`，格式为 `版本-加载器:MC版本`。
   - `versions/<版本>-<加载器>/gradle.properties`（YACL）：每个版本一个目录，里面是 `mcVersion=...`、`deps.fabricApi=...`、`meta.mcDep=~1.21.11`、`pub.stableMC=...` 等属性。
   - `stonecutter.properties.toml`（Elytra Trims 4.x）：版本无关属性放顶层，版本相关属性放 `["26.1"]`、`["26.2"]` 小节。
3. 加载器差异可能拆成多个构建文件：Elytra Trims 用 `mapBuilds { _, node -> "build.${node.project.substringAfterLast('-')}.gradle.kts" }`，即 `build.fabric.gradle.kts` / `build.neoforge.gradle.kts` 各管一个加载器。
4. 确认是否叠加了其他工具：modstitch（多加载器，见下文）、`stonecutter handlers { inherit(...) }` 声明要处理哪些文件类型。

## 条件编译块的写法

### 语法

**当前版本对应的代码保持裸写，其他版本的代码包在 `/*  */` 里**，条件用 `/*? if ... */` 或 `//? if ... {` 开头。Elytra Trims 的真实例子（激活版本为 fabric）：

```java
//? if fabric {
import dev.kikugie.elytratrims.api.ETCommonInitializer;
// 裸写的 fabric 代码，当前版本（fabric）会保留
//?}
```

```java
//? if neoforge {
/*import dev.kikugie.elytratrims.api.ETCommonInitializer;
import net.neoforged.fml.common.Mod;

@Mod("elytratrims")
public class NeoforgeEntrypoint { ... }
*///?}
```

- 比较条件形如 `if >=1.21.11`、`if <1.20.2`、`if fabric`（加载器常量，配合 `stonecutter parameters { constants { match(loader, "fabric", "neoforge") } }`）。
- **单行选择器**：`/*? if <=1.20.4*/getOrThrow(false) {}` + `/*? if >1.20.4*//*orThrow*/`——非当前分支的代码放在注释里（Elytra Trims 的 Util.kt 范例）。
- **多分支**：`//?} elif <1.20.6 {` 链式判断（HCsCR）。
- **文本替换块**（HCsCR）：`//~ if <条件> '原文' -> '替换' { ... //~}` 在源码里直接做文本替换。
- 嵌套时，符合当前条件的代码仍用 `/* */`，被注释的其他版本代码用 `/^ ^/` 表示内层条件（参考项目自身的既有写法，保持一致）。
- 需要 else / else-if 时，尽量用 `>=` 条件，不要用 `<` 或 `<=`。
- 方法体中出现条件编译且需要注释说明原因时，把原因写在该方法的 Javadoc 中，不要写在方法体里。
- 合并相邻条件编译块时，`*/` 后紧跟 `/*?`，中间不要留空白。
- 多版本差异应尽量下沉封装（例如到一个 bridge/适配层），避免业务代码到处散落版本分支——具体分层看项目架构约定。
- **JSON 也能条件编译**：HCsCR 在 `stonecutter.gradle.kts` 里 `handlers { inherit("java", "json") }`，mixin 配置等 JSON 文件里用同样的条件注释增删条目。

### 构建脚本里的版本判断

`stonecutter.gradle.kts` 里可以用版本条件控制参数：

- 老写法（3.x 时代）：`if (stonecutter.eval(mcVersion, "1.20.6")) { ... }`。
- 新写法（Elytra Trims 4.x）：`current.parsed >= "26.3"` 直接比较，配合 `swaps { }` / `replacements { string(...) { replace(...) } }` 做全局替换（如按版本替换 accessWidener 头、包名迁移）。

## 版本号拼接

**构建版本号用 SemVer build metadata**：`version = "${mod.version}+${deps.minecraft}"`（Elytra Trims 的 `build.fabric.gradle.kts`），产物形如 `4.9.0-beta.5+26.3`。`mod.version` 是模组自身版本（放 `stonecutter.properties.toml` 顶层），`deps.minecraft` 是当前激活的 MC 版本。

## 多加载器：Stonecutter + modstitch

Stonecutter 只处理版本差异；加载器差异（Fabric/NeoForge 元数据、入口）用 **modstitch** 配合：

- 每个版本目录的 `gradle.properties` 里写 `modstitch.platform=fabric-loom-remap`（或 neoforge 对应值），modstitch 据此生成对应加载器的 loom 配置（YACL 的做法）。
- **`modstitch.ct` 是加载器无关的 class tweaker**，格式为 `accessWidener v1 named` + `accessible class ...` / `extendable method ...`（YACL 的 modstitch.ct 是真实范例）。modstitch 按 `modstitch.ct → .classTweaker → accesstransformer.cfg` 顺序在项目链中自动查找，转换后设置 `loom.accessWidenerPath`，构建脚本里无需声明。
- 不用 modstitch 时，Elytra Trims 直接在 loom 里处理：`loom { accessWidenerPath = sc.process(file("...classtweaker"), "build/processed/...") }`。

## 切换版本与构建

- 切换激活版本前，先读项目的 AGENTS.md 或文档确认命令（常见为直接改 `stonecutter.gradle.kts` / `versions/current`，或 `./gradlew` 的 stonecutter 相关 task）。
- **CI 按实例激活**：Elytra Trims 的 CI 用 jq 从 `versions.json` 生成矩阵，设置 `MATRIX_INSTANCE` / `MATRIX_VERSION` 环境变量后，settings 里 `version(matrix, version)` 按实例激活，逐项目 `./gradlew :<项目>:build`。
- 多版本全量构建通常很慢：用后台任务运行，例如 `./gradlew buildAndCollect`（项目定义了该 task 时，Elytra Trims 用它收集各版本产物），不要阻塞对话。
- 只构建当前版本时用普通 `./gradlew build`。
- 构建前先运行项目的架构/格式检查任务（如 `checkArchitecture`）——多版本项目常把它挂进 `check`。

## 验证

- 改动条件编译块后，至少构建目标版本验证通过；如果可能，构建相邻的另一个版本，确认两个分支都正确。
- 留意 `/*? if ... */` 条件写反导致的"两个版本都走了同一分支"问题：切版本后重新构建是最可靠的验证。
- 切版本后若编译报"找不到符号/类"，先检查该符号是否在条件块外裸写（应包进对应版本的条件块）。
