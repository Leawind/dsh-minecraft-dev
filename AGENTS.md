# dsh-minecraft-dev 仓库指南

本仓库为 DeepSeek Harness 提供 **Minecraft 模组开发** Agent 预设，通过 GitHub 仓库分发（带 `dsh-plugin` topic），供其他开发者复制到自己的 `~/.dsh/.agent-presets/` 使用。

## 目录结构

```
agent-presets/minecraft-dev/        # 预设本体（唯一需要维护的内容）
├── agent.cordis.yml                # 组合文件：persona + 工具行 + 技能目录配置
├── preset.yml                      # 元数据：name / description
└── skills/                         # 按需加载的技能手册
    └── <skill_name>/SKILL.md
```

## 预设内容编写原则（核心经验）

预设是**面向所有人的通用分发物**，内容编写遵守以下原则：

### 不含本机特定内容

- 禁止出现绝对路径（`/home/steve/...` 等）、个人仓库名、个人目录组织约定
- 禁止把个人工作流习惯写成规定动作
- 判断标准：内容必须对任何使用者的环境都成立

### 不写显而易见或重复的内容

- 删除"即使不说，代理也极大概率知道"的常识（如"分阶段提交"、"复用原版逻辑"、"排查问题要看源码"）
- 同一规则只保留一处：persona 只放"不说就容易错"的核心约束，详细内容放对应技能
- 只保留"不说就不知道 / 不说就容易错"的信息增量
- 判断标准：删掉这条内容，代理的行为质量是否会下降

### 技能命名与内容一致

- 技能名（目录名 + frontmatter `name`）必须精确描述内容，不泛化
- 例：只讲 `me.modmuss50.mod-publish-plugin` 插件的技能命名为 `mod-publish-plugin`，不叫 `mod-publishing`（"模组发布"是领域名，内容是特定方法时名实不符）

### Persona 精简

- persona 只保留 3 类内容：核心兼容性约束、作者的独特偏好、技能加载指引
- 技术细节一律下沉到技能，不在 persona 重复

## 技能（SKILL.md）格式

- 每个技能一个目录，目录名 = 技能名
- 文件为 `SKILL.md`，带 YAML frontmatter：`name` + `description`
- `description` 以 "Use when..." 开头，说明何时加载该技能
- 技术事实（枚举值、版本号、API 行为）以官方源码为准，不凭记忆编写
- 可以附上相关源码仓库或官方文档的URL

## 修改流程

1. 编辑仓库中的 `agent-presets/minecraft-dev/`（仓库是权威源）
2. 同步到本机安装版：`cp -r agent-presets/minecraft-dev/. ~/.dsh/.agent-presets/minecraft-dev/`
3. **必须验证**：通过动态插件调用 `agentPresets.standingKeyFor('minecraft-dev')`，确认组合可加载（若无效会抛出 Cannot find package / invalid config / did not activate 等错误）
4. 提交（见提交规范）

## 同步与分发

- 发布渠道：GitHub 仓库 + `dsh-plugin` topic
- 使用方式：用户把 `agent-presets/minecraft-dev` 复制到自己的 `~/.dsh/.agent-presets/`，新建会话选择"Minecraft 模组开发"预设

## 提交规范

提交信息使用 Conventional Commits，全部英文：

- `feat:` 新技能、新内容
- `refactor:` 内容结构调整（精简、改名、去重）
- `docs:` 仅文档/说明修改

示例：`docs: add modrinth environment configuration to skill`
