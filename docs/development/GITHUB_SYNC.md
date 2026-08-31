# GitHub 同步规范

## 触发语句

当用户明确说“同步到 GitHub”时，视为授权执行当前项目的一次完整同步检查、提交、远程合并和推送。该授权不包含强制推送、删除远程分支、提交私有数据或绕过失败测试。

## 同步目标

- 远程仓库：`https://github.com/ethanwalkerwater/jsjy-calendar.git`
- 主分支：`main`
- 远程名称：`origin`
- 合并策略：保留历史的 merge，不使用 rebase 改写已共享历史

## 每次同步流程

1. 检查当前分支、工作区、暂存区和未跟踪文件。
2. 确认真实 CSV、密钥、`.env`、备份及工具日志未进入 Git。
3. 获取远程最新状态：`git fetch origin --prune`。
4. 汇总从上一次远程同步点到当前的提交、文件差异、文档决定和验证结果。
5. 对当前修改运行适用检查；测试失败时不推送，并报告失败。
6. 将本地未提交修改整理为边界清楚的提交，不把无关工作混在一起。
7. 如果 `origin/main` 有新提交，先合并远程：
   - 无冲突时完成 merge；
   - 内容冲突时理解双方意图后解决；
   - 涉及产品、领域、权限、费用、数据删除或架构决策的冲突必须交由用户确认；
   - 禁止直接使用全量 `ours` 或 `theirs` 覆盖。
8. 再次运行受影响检查。
9. 推送到 `origin/main`。
10. 返回同步提交范围、最终提交哈希、测试结果和遗留风险。

## Commit / Merge Message 规则

标题使用简洁的 Conventional Commit 风格，例如：

```text
feat: add deterministic scheduling constraints
fix: prevent stale schedule publication
docs: refine migration and agent workflow
sync: publish scheduling MVP progress
```

同步时的最后一个 commit 或 merge message 必须概括从上一次 GitHub 同步点到当前的全部有效进展，正文按实际内容选用以下栏目：

```text
Progress since <上次同步提交>:
- Added: 新增能力
- Changed: 行为或结构变化
- Fixed: 修复的问题
- Decisions: 已确认的产品或架构决定
- Validation: 测试、检查和人工验证
- Docs: 文档变化

Remote integration:
- 合并的远程变化及冲突处理

Residual risks:
- 尚未解决但不阻塞本次同步的问题
```

没有内容的栏目不写。消息必须描述实际变化，不能只写“update”“sync files”或“misc changes”。

## 安全边界

- 永不执行 `git push --force` 或 `--force-with-lease`，除非用户针对具体情况再次明确授权。
- 永不提交 `data/private/`、生产数据库、备份、访问 token、API 密钥或 `.env`。
- 不因同步而自动修改产品需求、业务规则或生产数据。
- 工作区存在来源不明的修改时先停止并说明。
- 远程冲突无法确定正确意图时先询问，不猜测。
- 推送前保持 `main` 可运行；存在已知阻塞测试失败时不推送。

## 追溯方式

每次同步以同步前的 `origin/main` 为基线，通过以下范围收集进展：

```text
git log <同步前的 origin/main>..HEAD
git diff <同步前的 origin/main>...HEAD
git status --short
```

同步完成后记录新的 `origin/main` 提交哈希。后续同步从该远程提交继续比较，无需维护额外状态文件。
