# 项目级技能更新当前不受 `skills check` / `skills update` 支持

## 问题描述

我使用下面的命令将技能按照到我的项目里

```bash
npx skills add henryzhuhr/skills --copy --agent claude-code --skill git-commit-helper -y
```

在 `.claude/skills/git-commit-helper` 目录下成功安装了技能，并且在项目根目录生成了 `skills-lock.json`，内容如下：

```json
{
  "version": 1,
  "skills": {
    "git-worktree-helper": {
      "source": "henryzhuhr/skills",
      "sourceType": "github",
      "computedHash": "55df5a20eea4d26a11df6e6668bf6807f9b450819cd8c9b63292691c18a00d38"
    }
  }
}
```

但是当我用 `npx skills check` 或 `npx skills update` 时，技能的更新检查和更新都没有生效。

我觉得这很奇怪，因为项目的目录下明明有 `skills-lock.json` 记录了这个技能的信息，按理说 `check` 和 `update` 应该能根据这个锁文件来检查和更新技能才对。

## 结论

当前仓库**不支持项目级远端技能**的统一更新检查和更新。

更准确地说：

- `skills check` 和 `skills update` 只处理**全局安装**的技能
- 项目级 `skills-lock.json` 目前只用于：
  - `experimental_install` 按 source 恢复项目技能
  - `experimental_sync` 对 `node_modules` 技能做基于内容哈希的同步/重装

## 代码依据

### 1. `check` / `update` 只读取全局锁文件

`src/cli.ts` 里的 `getSkillLockPath()` 明确返回：

- `$XDG_STATE_HOME/skills/.skill-lock.json`
- 或 `~/.agents/.skill-lock.json`

对应位置：

- `src/cli.ts:299`
- `src/cli.ts:307`

`runCheck()` 和 `runUpdate()` 都直接调用这个本地定义的 `readSkillLock()`，没有读取项目目录下的 `skills-lock.json`：

- `src/cli.ts:365`
- `src/cli.ts:475`

### 2. 项目级锁文件是另一套结构，且元数据不足以支持现有更新机制

项目级锁文件定义在 `src/local-lock.ts`，文件名为 `skills-lock.json`：

- `src/local-lock.ts:5`
- `src/local-lock.ts:45`

单个项目级 skill entry 只保存：

- `source`
- `sourceType`
- `computedHash`

定义位置：

- `src/local-lock.ts:15`

这和全局更新逻辑依赖的数据不一致。现有 `update` 逻辑依赖至少这些字段：

- `sourceUrl`
- `skillPath`
- `skillFolderHash`

定义位置：

- `src/cli.ts:283`
- `src/skill-lock.ts:14`

也就是说，即使把 `skills-lock.json` 读进来，当前字段也不够支撑现有 GitHub 检查更新和重装逻辑。

### 3. 项目级锁文件没有接入 `check` / `update`

`readLocalLock()` 的使用点只出现在：

- `src/add.ts`
- `src/install.ts`
- `src/sync.ts`

没有出现在 `check` / `update` 路径里。

相关位置：

- `src/add.ts:801`
- `src/add.ts:1535`
- `src/install.ts:17`
- `src/sync.ts:163`

### 4. 项目级目前支持的是“恢复”和“sync”，不是远端更新

`experimental_install`：

- 从项目 `skills-lock.json` 读取条目
- 按 `source` 分组
- 再调用 `runAdd()` 重新安装

实现位置：

- `src/install.ts:17`
- `src/install.ts:61`

`experimental_sync`：

- 只针对 `node_modules` 发现的技能
- 重新计算当前技能目录哈希 `computedHash`
- 与 `skills-lock.json` 比较，不一致则重装

实现位置：

- `src/sync.ts:162`
- `src/sync.ts:175`
- `src/sync.ts:348`

## 影响

- 项目级通过 `skills add <source>` 安装的远端技能，当前没有对应的 `skills check` / `skills update`
- 项目级 `skills-lock.json` 更像“项目恢复清单”，不是完整的“可更新锁文件”
- 只有 `node_modules` 来源的项目级技能，能通过 `experimental_sync` 获得类似更新能力

## GitHub 上的相关 Issue / PR

以下状态基于 2026-03-22 在 GitHub 上查看的结果。

### 1. PR #544

- 链接: <https://github.com/vercel-labs/skills/pull/544>
- 标题: `fix: update project-local skills from skills-lock`
- 状态: Open
- 时间: 2026-03-08 打开

PR 描述里的目标比较明确：

- 让 `skills check` 在“本地技能已被项目 `skills-lock.json` 跟踪”时优先看项目级条目
- 让 `skills update` 对这些本地 source 重新计算哈希，并把有变化的技能重新安装到 `.agents/skills`
- 增加本地 add -> 修改 source -> check/update 的回归测试

这个 PR 看起来是在解决“**项目级、本地路径来源**的技能不能被 `check/update` 处理”的问题。

但它不是完整的“项目级远端技能更新”方案。结合当前仓库里 `LocalSkillLockEntry` 只保存 `source` / `sourceType` / `computedHash` 这一点来看，它更像是补齐 **local source** 的项目级更新能力，而不是补齐 GitHub / 远端 source 的项目级更新链路。

### 2. PR #637

- 链接: <https://github.com/vercel-labs/skills/pull/637>
- 标题: `fix: scope check/update to project skills when skills-lock.json exists`
- 状态: Open
- 时间: 2026-03-15 打开

这个 PR 的思路和 #544 不一样：

- 读取项目 `skills-lock.json`
- 用里面的 skill 名称去过滤全局锁文件中的条目
- 如果项目技能还没进全局锁，提示先跑 `npx skills install`

从 PR 描述看，它修的是“在项目目录里执行 `skills check` / `skills update` 时，不该去操作所有全局技能”这个问题。

这条路线仍然把**全局锁文件**当作实际更新来源，而不是让项目级 `skills-lock.json` 自己成为完整的更新来源。所以它解决的是“作用域错误”问题，不一定能覆盖本文上面说的“项目级远端技能元数据不足”问题。

### 3. Issue #690

- 链接: <https://github.com/vercel-labs/skills/issues/690>
- 标题: `[Feature]: Project-local skills are written to skills-lock.json, but check / update do not operate on them`
- 状态: Open
- 时间: 2026-03-19 打开

这是和本文最一致的一条 feature issue。标题本身就在说：

- 项目级技能已经写进 `skills-lock.json`
- 但 `check` / `update` 没有对它们生效

### 4. Issue #542

- 链接: <https://github.com/vercel-labs/skills/issues/542>
- 标题: `[Bug]: skills check/update ignore project-level skills-lock.json; global lock migration silently wipes tracked skills`
- 状态: Open
- 时间: 2026-03-08 打开

这是更早的一条直接相关 bug。它同时指出了两件事：

- `skills check` / `skills update` 忽略项目级 `skills-lock.json`
- 全局锁文件迁移时，旧数据可能被静默清空

PR #637 的描述里明确写了 `Fixes #542`，但截至 2026-03-22 该 PR 仍是 Open。

### 5. Issue #683

- 链接: <https://github.com/vercel-labs/skills/issues/683>
- 标题: `[Feature]: global restore/relink from ~/.agents/.skill-lock.json for canonical skills + agent links`
- 状态: Open
- 时间: 2026-03-18 打开

这条不是“项目级更新”本身，但它和本文相关，因为它也在讨论锁文件到底应该承担什么职责：

- 只是记录安装状态
- 还是也应该支持 restore / relink / update

它更偏向**全局锁文件的恢复能力**，可以当作相邻议题留档。

## 建议方向

如果要补项目级更新支持，至少需要两步：

1. 扩展 `LocalSkillLockEntry`
   - 增加 `sourceUrl`
   - 增加 `skillPath`
   - 对 GitHub 来源增加可比较的远端 hash 信息，或复用 `skillFolderHash`

2. 为项目级增加单独的 `check/update` 路径
   - 读取 `skills-lock.json`
   - 复用现有 GitHub folder hash 检查逻辑
   - 更新时走 project-scoped install，而不是 `-g`

## 备注

README 当前的 `skills check` / `skills update` 说明也没有区分全局和项目级：

- `README.md:131`

这会让用户以为项目级技能也在支持范围内。

另外，GitHub 上已经出现两条不同的修复思路：

- `#544` 更偏向给项目级 local source 单独补 check/update 能力
- `#637` 更偏向在有 `skills-lock.json` 时，把 check/update 的作用域限制到该项目，并继续依赖全局锁文件

这两条路线对应的产品语义并不完全一样，后续如果要正式实现，最好先统一“项目级锁文件到底是不是更新来源”这个设计。
