# Claude Code 工作方式与核心概念

Claude Code 是运行在开发环境中的编程助理。你用自然语言描述任务，它会读取项目、规划步骤、编辑文件、运行命令，并在需要时请求权限。进阶使用的关键不是“让它随便做”，而是理解它看到了什么、能调用什么工具、哪些动作需要你确认。

本章只保留环境自检和常见不可用原因，不展开安装教程。

高风险提醒：涉及删除文件、批量重构、修改权限、安装依赖、提交代码、联网访问、处理密钥时，要要求 Claude Code 先说明计划、影响范围和回滚方案，并等待你确认。

## 使用场景

适合以下任务：

- 让 Claude Code 阅读项目并解释结构。
- 让 Claude Code 修改少量文件并展示 diff。
- 让 Claude Code 运行测试、格式化、lint 等检查命令。
- 用 `/help` 查看可用能力，用 `/doctor` 检查环境，用 `/permissions` 管理权限。

Claude Code 的核心心智模型：

- 对话输入是任务说明，不是 shell。
- 普通终端命令由 shell 执行，例如 `pwd`、`git status`、`claude --version`。
- Claude Code 可能调用工具读取文件、编辑文件、运行命令。
- 权限决定它能否执行某些命令或访问某些资源。
- 你负责确认范围、审查 diff 和决定是否提交。

## 环境自检

普通终端命令：

```bash
claude --version
claude doctor
pwd
```

Windows PowerShell 普通终端命令：

```powershell
claude --version
claude doctor
Get-Location
```

进入项目目录后再启动：

普通终端命令：

```bash
cd /path/to/your-project
pwd
claude
```

Claude Code 对话输入：

```markdown
/help
```

Claude Code 对话输入：

```markdown
/doctor
```

Claude Code 对话输入：

```markdown
/permissions
```

常见不可用原因：

- 当前终端找不到 `claude` 命令，通常是 PATH 没配置好或终端没有重启。
- 没有登录账号，或当前账号没有可用订阅、Console 额度、企业权限或第三方提供方配置。
- 当前目录不是项目根目录，Claude Code 读到的上下文不完整。
- Git、Node、Python、包管理器或项目依赖缺失，导致检查命令无法运行。
- 权限设置阻止了文件写入、命令执行、联网或访问某些路径。
- 公司代理、防火墙、证书或网络策略导致无法连接服务。

## 推荐提示词模板

Claude Code 对话输入：

```markdown
请先不要修改文件。请检查当前 Claude Code 会话的工作目录和项目上下文。

请输出：
1. 当前目录是否像项目根目录。
2. 你能看到的关键文件。
3. 你建议我先运行哪些普通终端命令验证环境。
4. 有哪些权限或安全风险需要确认。
```

Claude Code 对话输入：

```markdown
请解释你准备如何完成这个任务。执行任何文件修改、安装依赖、联网、删除文件、改权限、提交代码前，都必须先停下来让我确认。

任务：阅读当前项目并找出测试命令。
限制：只读分析，不修改文件。
```

Claude Code 对话输入：

```markdown
请帮我判断当前报错是否来自 Claude Code 环境、项目依赖、权限设置还是网络连接。请先列排查顺序，再告诉我每一步要运行的普通终端命令。
```

## 错误示范

Claude Code 对话输入：

```markdown
你自己看着办，能跑起来就行，缺什么就装什么。
```

问题：

- “缺什么就装什么”允许安装依赖，属于高风险动作。
- 没有说明项目目录、包管理器、版本限制。
- 没有要求先解释计划和影响。

Claude Code 对话输入：

```markdown
所有权限都给你，直接修。
```

问题：

- 权限放得过宽。
- 没有限定是否可以联网、删除文件或改权限。
- 不利于审查和回滚。

## 正确示范

Claude Code 对话输入：

```markdown
请只读检查当前项目是否具备运行条件，不要修改文件。

请检查：
- 当前目录
- 项目语言和依赖文件
- 可能的测试命令
- Claude Code 权限是否足够完成只读分析

输出：
- 发现的问题
- 建议我手动执行的普通终端命令
- 哪些动作需要额外确认
```

普通终端命令：

```bash
pwd
git status
ls
claude --version
claude doctor
```

Claude Code 对话输入：

```markdown
请基于 `claude doctor` 的结果解释问题来源。除非我明确同意，不要修改配置文件、不要安装依赖、不要联网下载任何内容。
```

## 练习任务

1. 在一个练习项目根目录运行环境自检命令。
2. 启动 Claude Code 后输入 `/help`，观察可用命令。
3. 输入 `/doctor`，检查环境状态。
4. 输入 `/permissions`，查看权限相关设置。
5. 让 Claude Code 只读分析项目结构，并指出它不能确定的信息。

普通终端命令：

```bash
claude --version
claude doctor
pwd
git status
```

Claude Code 对话输入：

```markdown
请用“确定 / 不确定 / 需要我确认”三类整理你对当前项目的判断。不要修改文件。
```

## 验收标准

完成本章后，你应该能做到：

- 能区分普通终端命令和 Claude Code 对话输入。
- 能使用 `claude --version`、`claude doctor`、`pwd` 或 `Get-Location` 做基础自检。
- 能确认 Claude Code 是否在正确项目目录中运行。
- 能使用 `/help`、`/doctor`、`/permissions` 获取帮助、诊断环境和理解权限。
- 能列出 Claude Code 不可用的常见原因。
- 能在高风险动作前要求 Claude Code 先暂停并等待确认。

## 官方依据

访问日期：2026-04-26。

- Claude Code Overview：<https://code.claude.com/docs/en/overview>
- Quickstart：<https://code.claude.com/docs/en/quickstart>
- CLI reference：<https://code.claude.com/docs/en/cli-reference>
- Commands：<https://code.claude.com/docs/en/commands>
- Permissions：<https://code.claude.com/docs/en/permissions>
- Security：<https://code.claude.com/docs/en/security>
