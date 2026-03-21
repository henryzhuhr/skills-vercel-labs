# 创建和发布 Skill 完整指南

本指南帮助你创建、发布和维护一个可以被其他人安装的 Agent Skill。

---

## 目录

1. [什么是 Skill](#什么是-skill)
2. [创建 Skill](#创建-skill)
3. [发布到 GitHub](#发布到-github)
4. [其他人如何安装](#其他人如何安装)
5. [更新你的 Skill](#更新你的-skill)
6. [用户如何更新 Skill](#用户如何更新-skill)
7. [提高曝光度](#提高曝光度)
8. [最佳实践](#最佳实践)
9. [常见问题](#常见问题)

---

## 什么是 Skill

Skill 是模块化的指令包，用于扩展 AI 编程助手的能力。每个 Skill 由一个 `SKILL.md` 文件定义，包含：

- **YAML Frontmatter**：定义 skill 的名称和描述
- **Markdown 内容**：详细的指令和使用说明

Skill 可以让 AI 助手执行特定任务，如代码审查、测试生成、部署流程等。

---

## 创建 Skill

### 方法一：使用 CLI 初始化（推荐）

```bash
# 初始化一个新 skill
npx skills init my-skill-name
```

这会创建如下结构：

```
my-skill-name/
└── SKILL.md
```

### 方法二：手动创建

创建目录和 `SKILL.md` 文件：

```bash
mkdir my-skill-name
cd my-skill-name
touch SKILL.md
```

### SKILL.md 文件格式

```markdown
---
name: my-skill-name
description: 简短描述你的 skill 做什么，一句话说明核心功能
---

# My Skill Name

## 概述

详细说明你的 skill 提供什么功能，解决什么问题。

## 何时使用

描述使用场景，例如：
- 当用户需要 X 时
- 当项目需要 Y 功能时

## 使用步骤

1. 第一步做什么
2. 第二步做什么
3. ...

## 示例

提供使用示例或输出示例。

## 注意事项

列出任何限制、前提条件或已知问题。
```

### Frontmatter 字段说明

| 字段 | 必填 | 说明 |
|------|------|------|
| `name` | 是 | 唯一标识符，小写字母和连字符，如 `my-skill` |
| `description` | 是 | 简短描述，用于技能列表展示 |
| `metadata.internal` | 否 | 设为 `true` 隐藏技能，仅内部使用 |

---

## 发布到 GitHub

其他人通过 GitHub 仓库安装 skill，你需要将代码发布到 GitHub。

### 选择仓库结构

#### 方式 A：单 Skill 仓库

一个仓库只包含一个 skill：

```
my-react-skill/           ← 仓库名
├── README.md
└── SKILL.md              ← name: my-react-skill
```

**适用场景**：独立的、功能单一的 skill

**安装命令**：
```bash
npx skills add username/my-react-skill
```

#### 方式 B：多 Skill 仓库（推荐）

一个仓库包含多个 skill：

```
agent-skills/             ← 仓库名
├── README.md
└── skills/
    ├── frontend-design/    ← skill 1
    │   └── SKILL.md        ← name: frontend-design
    ├── pr-review/          ← skill 2
    │   └── SKILL.md        ← name: pr-review
    └── test-generator/     ← skill 3
        └── SKILL.md        ← name: test-generator
```

**适用场景**：一系列相关的 skills，便于管理和分发

**安装命令**：
```bash
# 安装特定 skill
npx skills add username/agent-skills --skill frontend-design

# 安装所有 skills
npx skills add username/agent-skills --all
```

### 发布步骤

```bash
# 1. 创建本地 skill 目录
npx skills init my-skill
cd my-skill

# 2. 编辑 SKILL.md 内容
# 使用你喜欢的编辑器完善内容

# 3. 初始化 git 并提交
git init
git add .
git commit -m "Initial commit: my-skill"

# 4. 在 GitHub 创建仓库（需要安装 GitHub CLI）
gh repo create your-username/my-skill --public --source=. --push

# 或者手动在 GitHub 创建仓库后推送
git remote add origin https://github.com/your-username/my-skill.git
git branch -M main
git push -u origin main
```

---

## 其他人如何安装

用户可以使用以下命令安装你的 skill：

### 安装单 Skill 仓库

```bash
# 使用 GitHub  shorthand
npx skills add your-username/my-skill

# 使用完整 URL
npx skills add https://github.com/your-username/my-skill

# 全局安装
npx skills add your-username/my-skill -g
```

### 安装多 Skill 仓库中的特定 skill

```bash
# 列出可用 skills
npx skills add your-username/agent-skills --list

# 安装特定 skill
npx skills add your-username/agent-skills --skill frontend-design

# 安装多个 skills
npx skills add your-username/agent-skills --skill frontend-design --skill pr-review
```

### 安装选项说明

运行安装命令时，系统会提示你选择：

**1. 选择 Agent**
```
◇ Which agents do you want to install to?
│ Amp, Cline, Codex, Cursor, Gemini CLI, GitHub Copilot, Kimi Code CLI,
│ OpenCode, Warp, Claude Code
```
选择要安装到的目标 Agent（可多选）。

**2. 安装范围**（仅当 `-g` 未指定时）
```
◇ Installation scope
│ ● Project  - Install in current directory (committed with your project)
│ ○ Global   - Install in home directory (available across all projects)
```
- **Project**：技能安装在当前项目的 `./.agents/skills/` 目录，随项目一起提交到 git
- **Global**：技能安装到用户主目录，所有项目可用

**3. 安装方式**（当选择的 Agent 使用不同目录时）
```
◇ Installation method
│ ● Symlink (Recommended) - Single source of truth, easy updates
│ ○ Copy to all agents    - Independent copies for each agent
```
- **Symlink**：创建软链接，单一来源，更新方便（推荐）
- **Copy**：为每个 Agent 复制独立文件

### 常用标志

| 短模式 | 长模式 | 说明 |
|--------|--------|------|
| `-y` | `--yes` | 跳过所有确认提示 |
| `-g` | `--global` | 安装到全局目录（跳过范围选择） |
| `-a` | `--agent <name>` | 指定目标 Agent，跳过选择 |
| `-s` | `--skill <name>` | 指定要安装的 skill |
| `-l` | `--list` | 仅列出 skills，不安装 |
| `--copy` | - | 强制使用 copy 模式（跳过方式选择） |
| `--all` | - | 安装所有 skills 到所有 Agents |

```bash
# 跳过所有提示，快速安装
npx skills add your-username/agent-skills --skill frontend-design --agent claude-code --yes

# 安装到全局，不询问
npx skills add your-username/agent-skills --skill frontend-design -g -y

# 使用 copy 模式，不询问
npx skills add your-username/agent-skills --skill frontend-design --copy -y
```

---

## 更新你的 Skill

### 发布更新（作者）

```bash
# 1. 修改 SKILL.md 内容
# 2. 提交并推送
git add .
git commit -m "Update: improved instructions for X"
git push
```

**推送后，已安装的用户可以通过 `npx skills update` 获取更新。**

### 版本管理机制

> **重要：** Skills CLI 不使用传统的语义化版本（如 v1.0.0），而是使用 **Git SHA 哈希值** 来追踪版本。

| 机制 | 说明 |
|------|------|
| **自动版本追踪** | 每次 `git push` 后，skill 文件夹的 Git tree SHA 会自动更新 |
| **更新检测** | 用户运行 `npx skills check` 时，CLI 会比较本地 hash 和 GitHub 最新 hash |
| **无需手动管理** | 你不需要修改任何版本号，提交即发布 |

### 提交信息建议

虽然版本号由系统自动管理，但建议使用清晰的提交信息方便用户了解更新内容：

```bash
# 功能更新
git commit -m "Add TypeScript strict mode support"

# 修复问题
git commit -m "Fix: correct installation path for Windows users"

# 文档改进
git commit -m "docs: add more examples to troubleshooting section"

# 重大变更
git commit -m "BREAKING: change default behavior for X"
```

---

## 用户如何更新 Skill

### 检查是否有可用更新

```bash
# 检查所有已安装 skill 的更新
npx skills check

# 检查特定 skill 的更新
npx skills check my-skill-name
```

输出示例：
```
✓ frontend-design     已是最新版本 (v1.2.0)
✗ pr-review          有可用更新 (v1.0.1 → v1.1.0)
✗ test-generator     有可用更新 (v2.0.0 → v2.1.0)
```

### 执行更新

```bash
# 更新所有已安装的 skill
npx skills update

# 更新特定的 skill
npx skills update my-skill-name

# 更新全局安装的 skill
npx skills update -g

# 更新项目级别的 skill
npx skills update --project
```

### 自动更新（推荐用户）

在项目 `.claude/settings.json` 或其他 agent 配置中添加 hooks，实现自动检查更新：

```json
{
  "hooks": {
    "BeforeTool": {
      "if": "command -v npx && npx skills check 2>/dev/null | grep -q '有可用更新'",
      "run": "echo 'Skill 更新可用！运行 npx skills update 获取更新'"
    }
  }
}
```

### 更新频率建议

| 用户类型 | 建议频率 |
|----------|----------|
| 普通用户 | 每周运行一次 `npx skills update` |
| 重度用户 | 每天或每次开始新项目前检查 |
| 团队项目 | 在 CI/CD 中集成更新检查 |

### 故障排查

**问题：更新后 skill 没有变化？**

```bash
# 1. 确认更新已成功推送到 GitHub
# 2. 检查本地 skill 来源
npx skills list

# 3. 强制重新安装
npx skills remove my-skill-name
npx skills add username/my-skill-name
```

**问题：更新失败？**

```bash
# 检查网络连接
# 确认 GitHub 可访问

# 清除缓存重试
rm -rf ~/.skills-cache  # macOS/Linux
```

---

## 提高曝光度

### 1. 优化技能信息

- **名称**：简洁、易记、包含关键词（如 `react`, `test`, `pr`）
- **描述**：清楚说明功能，包含使用场景
- **内容**：结构清晰，有示例

### 2. 完善 GitHub 仓库

- 添加详细的 `README.md`
- 添加使用示例和截图
- 使用合适的标签（topics）
- 保持活跃更新

### 3. 分享到社区

- 在相关论坛、Discord、Reddit 分享
- 写博客介绍你的 skill
- 在 Twitter/X、LinkedIn 分享

### 4. Skills.sh 排行榜

你的 skill 会自动收录到 https://skills.sh

**排名因素**：
- 安装次数
- GitHub stars
- 活跃度

---

## 最佳实践

### SKILL.md 编写

- **标题清晰**：使用一级标题写明 skill 名称
- **场景明确**：清楚说明何时使用该 skill
- **步骤具体**：提供可执行的步骤，而非抽象概念
- **示例丰富**：提供输入输出示例
- **边界清晰**：说明不适用的场景

### 命名规范

- 使用小写字母和连字符：`my-skill-name`
- 名称要有描述性：`react-performance` 优于 `react-skill`
- 避免通用名称：`utils` 太模糊，`code-optimizer` 更具体

### 仓库管理

- 保持仓库整洁，只包含必要文件
- 添加 `.gitignore` 排除临时文件
- 使用 `README.md` 提供额外文档
- 启用 GitHub Issues 收集反馈

---

## 常见问题

### Q: Skill 没有出现在 skills.sh 上？

**A:** 确保：
1. GitHub 仓库是公开的
2. `SKILL.md` 有正确的 `name` 和 `description`
3. 等待几分钟让系统抓取更新

### Q: 用户反馈安装失败？

**A:** 检查：
1. `SKILL.md` 文件是否在仓库根目录或 `skills/` 目录下
2. Frontmatter YAML 格式是否正确
3. 名称是否只有小写字母和连字符

### Q: 可以删除已发布的 skill 吗？

**A:** 可以，但建议：
1. 在仓库中说明已废弃
2. 通知已安装用户
3. 提供替代方案（如有）

### Q: 支持私有仓库吗？

**A:** 不支持。skills.sh 生态基于公开仓库，私有 skill 只能手动复制文件。

### Q: 如何收集用户反馈？

**A:** 推荐方式：
1. 在 GitHub 开启 Issues
2. 在 `SKILL.md` 底部添加反馈链接
3. 创建讨论区（Discussions）

---

## 快速开始模板

复制以下内容开始：

### 单 Skill 仓库模板

```bash
# 创建目录
mkdir my-skill && cd my-skill

# 创建 SKILL.md
cat > SKILL.md << 'EOF'
---
name: my-skill
description: 一句话描述你的 skill 功能
---

# My Skill

## 概述

详细说明...

## 何时使用

- 场景 1
- 场景 2

## 步骤

1. ...
2. ...
EOF

# 初始化 git
git init
git add .
git commit -m "Initial commit"

# 推送到 GitHub
gh repo create your-username/my-skill --public --source=. --push
```

---

## 相关资源

- [Skills CLI README](../README.md)
- [Agent Skills 规范](https://agentskills.io)
- [Skills.sh 排行榜](https://skills.sh)
