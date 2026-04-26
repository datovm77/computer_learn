# 必备编程基础：Git、diff、commit 与回退

Claude Code 可以协助你理解 Git 状态、生成变更摘要、运行检查，甚至准备提交。但 Git 是代码安全边界，提交和回退都可能影响团队协作。你必须先看懂 diff，再决定是否提交。

⚠️ 提交代码、回退代码、删除文件、批量重构、改权限、安装依赖、联网拉取内容、处理密钥都需要明确确认。不要让 Claude Code 在你没看 diff 前直接 commit。

## 使用场景

你会在这些场景用到 Git：

- 开始任务前确认工作区是否干净。
- Claude Code 修改后查看哪些文件变了。
- 审查每个文件的 diff。
- 运行测试后准备提交。
- 发现方向错了，决定手动回退或让 Claude Code 说明回退方案。

普通终端命令：

```bash
git status
git diff
git diff --stat
git log --oneline -5
```

核心概念：

- 工作区：你当前文件系统里的实际文件。
- 暂存区：准备进入下一次提交的变更。
- commit：一次带说明的历史快照。
- diff：当前变更和某个基准之间的差异。
- 回退：撤销变更，风险取决于是否已提交、是否已推送、是否影响别人。

## 推荐提示词模板

Claude Code 对话输入：

```markdown
请检查当前 Git 状态并解释，不要修改文件。

请输出：
- 当前分支
- 是否有未提交变更
- 变更文件列表
- 哪些变更可能不是本任务产生的
- 下一步建议
```

Claude Code 对话输入：

```markdown
请根据当前 diff 做代码审查，不要修改文件，不要提交。

请重点检查：
- 是否超出任务范围
- 是否有删除文件
- 是否有密钥或凭证
- 是否有权限变更
- 是否需要运行测试
```

Claude Code 对话输入：

```markdown
请为当前变更草拟 commit message，但不要执行 git commit。

输出格式：
- 标题行
- 正文要点
- 验证命令
- 剩余风险
```

## 错误示范

Claude Code 对话输入：

```markdown
改完直接提交。
```

哪里不对：

- 没有先检查 `git diff`。
- 没有运行测试或说明为什么不运行。
- 提交代码是高风险动作，需要确认。

普通终端命令：

```bash
git reset --hard
```

哪里不对：

- 这会丢弃未提交变更。
- 如果你不确定这些变更是谁做的，可能删除别人或其他 agent 的工作。
- 回退前应该先看 `git status` 和 `git diff`，必要时备份。

Claude Code 对话输入：

```markdown
把所有报错都修了，能删就删。
```

哪里不对：

- 删除文件是高风险动作。
- “所有报错”范围不清。
- 容易引入大范围不可审查变更。

## 正确示范

普通终端命令：

```bash
git status
git diff --stat
git diff
```

Claude Code 对话输入：

```markdown
请审查当前 diff，不要修改文件。

请判断：
1. 变更是否只涉及本任务范围。
2. 是否包含删除文件、权限变化、密钥、依赖安装或联网相关变更。
3. 是否需要补充测试。
4. 是否可以进入提交前检查。
```

Claude Code 对话输入：

```markdown
请执行提交前检查计划，但每一步运行命令前先告诉我命令和目的。

限制：
- 不安装依赖，除非我确认。
- 不提交代码，除非我确认。
- 不回退代码，除非我确认。
- 不处理真实密钥。
```

普通终端命令：

```bash
git add README.md
git commit -m "docs: clarify local development steps"
```

说明：只有在你确认 diff、验证结果和提交信息后，才执行提交命令。

## 回退风险

回退不是一个动作，而是一组风险不同的操作：

- 未暂存变更：可能通过编辑器、补丁或 Git 命令撤销。
- 已暂存未提交：需要先取消暂存或重新选择提交内容。
- 已提交未推送：可能用新提交修正，或在个人分支上改写历史。
- 已推送共享分支：通常不要改写历史，应新提交修复。

Claude Code 对话输入：

```markdown
请帮我分析如何安全撤销当前任务的变更。不要执行任何回退命令。

请先说明：
- 哪些文件会受影响
- 是否有其他人的变更
- 可以选择哪些回退方案
- 每个方案的风险
```

## 练习任务

1. 找一个练习仓库。
2. 让 Claude Code 做一个只改文档的小修改。
3. 用普通终端命令查看 `git status`、`git diff --stat`、`git diff`。
4. 让 Claude Code 解释 diff。
5. 让 Claude Code 草拟 commit message，但不要提交。

普通终端命令：

```bash
git status
git diff --stat
git diff
```

Claude Code 对话输入：

```markdown
请解释当前 diff。请按文件列出：改了什么、为什么改、是否符合任务范围、需要如何验证。不要提交代码。
```

## 验收标准

学完本章，你应该能：

- 能用 `git status` 判断工作区状态。
- 能用 `git diff` 查看 Claude Code 修改了什么。
- 能在提交前发现越界修改、删除文件、权限变化、密钥和依赖变更。
- 能要求 Claude Code 只草拟 commit message，而不是直接提交。
- 能解释为什么回退命令需要谨慎，尤其是在有其他 agent 或用户变更时。

## 官方依据

访问日期：2026-04-26。

- Common workflows：<https://code.claude.com/docs/en/common-workflows>
- CLI reference：<https://code.claude.com/docs/en/cli-reference>
- Permissions：<https://code.claude.com/docs/en/permissions>
- Security：<https://code.claude.com/docs/en/security>

