# 项目级技能更新当前不受 `skills check` / `skills update` 支持

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
