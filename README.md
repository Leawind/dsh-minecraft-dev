# dsh-minecraft-dev

一个面向 Minecraft 模组开发的 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness) Agent 预设。

在 DSH 中新建会话时选择 **Minecraft 模组开发** 预设，代理会获得模组开发实践约定，并可按需加载多版本开发、Mixin 规范、模组发布、兼容性排查等技能。

## 功能

- **知识注入**：persona 注入作者的经验约定（Mixin 禁用 `@Redirect`、文档正面描述、技能加载指引）
- **技能手册**（按需加载）：
  | 技能                         | 用途                                                                               |
  | ---------------------------- | ---------------------------------------------------------------------------------- |
  | `mod-ecosystem-overview`    | 模组开发生态速览：加载器选择、发布平台、CI 发布、文档发布、源码管理与许可证                                           |
  | `stonecutter-multiversion`   | Stonecutter 多版本开发：条件编译写法、版本切换、多版本构建                         |
  | `mixin-development`          | Mixin 开发：注入点选择、兼容性安全实践、崩溃修复                                   |
  | `mod-publish-plugin`         | 使用 mod-publish-plugin 发布到 Modrinth / CurseForge：插件配置、发布流程、故障排查 |
  | `compat-troubleshooting`     | 模组兼容性排查：查看参考模组源码、定位冲突点、验证修复                             |
  | `reference-mod-architecture` | 参考开源模组架构设计：多加载器/模块划分/跨平台抽象等模式的原理、实例与选型         |

## 安装

### 克隆仓库后复制

```sh
git clone https://github.com/Leawind/dsh-minecraft-dev.git
mkdir -p ~/.dsh/.agent-presets
cp -r dsh-minecraft-dev/agent-presets/minecraft-dev ~/.dsh/.agent-presets/
```

## 使用

- 新建会话时选择预设即可，无需其他配置
- 遇到多版本改动时，代理会自动加载 `stonecutter-multiversion` 技能
- 项目根目录的 `AGENTS.md` 仍然会优先注入（会话工作区指令），与本预设的通用规则互补

## 自定义

预设基于 DSH 的 `standard` 预设派生（继承其工具链）。如需调整：

1. 编辑 `~/.dsh/.agent-presets/minecraft-dev/agent.cordis.yml` 中的 persona 段落
2. 在 `skills/` 下增删技能目录（每个技能是一个目录 + 一份 `SKILL.md`）
3. 修改后重启会话生效

> 注意：不要直接修改 DSH 安装目录中的官方预设（`standard` / `code` / `minimal` / `cordis`），升级会被覆盖；自定义请始终基于本仓库的副本。
