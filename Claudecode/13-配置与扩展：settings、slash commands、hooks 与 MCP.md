# 13-配置与扩展：settings、commands / skills、hooks 与 MCP

Claude Code 的进阶能力很大一部分来自配置与扩展：`settings` 管全局和项目行为，commands / skills 把常用流程变成可复用命令，hooks 在特定生命周期自动执行检查，MCP 把外部工具和数据源接入 Claude Code。它们能提升效率，也会扩大权限边界。

高风险提醒：配置与扩展常常涉及命令执行、联网、外部系统、权限控制和密钥。任何会删除文件、批量重构、改权限、安装依赖、提交代码、联网或处理密钥的配置，都必须先做只读评审，并清楚说明触发条件。

## 使用场景

settings 适合：

- 配置默认模型、权限、环境变量、工具策略、项目级规则。
- 固化团队约定，例如禁止某些命令、指定允许的工具。
- 区分用户级配置和项目级配置。

commands / skills 适合：

- 把重复提示词封装成命令，例如 `/review-diff`、`/write-tests`、`/release-note`。
- 让团队成员用同一套审查和实现流程。
- 降低长提示词复制错误。
- 使用 skills 承载更完整的流程、说明和支持文件。

说明：`/help`、`/doctor`、`/permissions` 这类是 Claude Code 内置 commands；你自己创建的可复用命令目前推荐用 skills，例如 `.claude/skills/review-diff/SKILL.md`。旧的 `.claude/commands/review-diff.md` 仍可工作，但新内容优先按 skills 组织。

hooks 适合：

- 在工具调用前后自动检查风险。
- 对写入、命令执行、提交前流程做提醒或阻断。
- 自动运行格式检查、审查脚本或日志记录。

MCP 适合：

- 接入外部系统，例如 issue tracker、数据库只读查询、文档库、浏览器或内部工具。
- 给 Claude Code 提供结构化上下文，而不是手工复制粘贴。
- 将团队已有工具作为受控能力开放给 Agent。

不建议扩展的情况：

- 你还不清楚扩展会获得什么权限。
- MCP 服务需要生产密钥但没有最小权限账号。
- hooks 会自动执行破坏性命令。
- slash command 包含“直接提交”“自动删除”“自动安装依赖”等动作。

## 推荐提示词模板

Claude Code 对话输入：

```markdown
请只读分析当前项目是否适合加入 Claude Code settings。
要求：
- 不修改文件。
- 不安装依赖。
- 不联网。
- 输出：建议配置项、风险、是否适合放在用户级或项目级。
高风险动作包括删除文件、批量重构、改权限、安装依赖、提交代码、联网、处理密钥。
```

Claude Code 对话输入：

```markdown
请帮我设计一个可通过 `/review-diff` 调用的 skill。
目标：审查当前 git diff。
要求：
- 默认只读。
- 不修改文件。
- 不提交代码。
- 检查密钥、删除文件、权限变化、依赖安装、批量重构。
- 输出中文 Markdown。
先给出 `SKILL.md` 内容草案，不要写入文件。
```

Claude Code 对话输入：

```markdown
请评估是否应该为本项目添加 hooks。
要求：
- 只读分析。
- 列出适合的触发点。
- 每个 hook 都说明风险。
- 不要创建或修改配置文件。
```

Claude Code 对话输入：

```markdown
请为接入一个只读文档 MCP 设计安全方案。
要求：
- 不包含真实密钥。
- 说明最小权限原则。
- 说明联网审核点。
- 说明如何验证 MCP 只提供只读能力。
```

普通终端命令：

```bash
claude --help
```

用途：查看当前安装版本支持的 CLI 参数，避免照抄过期配置。

## 错误示范

Claude Code 对话输入：

```markdown
帮我加一个 hook，每次运行命令前自动 chmod -R 777 .
```

问题：

- 改权限是高风险动作。
- `chmod -R 777 .` 会破坏安全边界。
- 自动触发会放大事故范围。

Claude Code 对话输入：

```markdown
创建一个 MCP，连接生产数据库，给 Claude 完整读写权限。
```

问题：

- 生产数据库和写权限风险极高。
- 缺少只读账号、审计和查询限制。
- 可能暴露隐私数据或密钥。

Claude Code 对话输入：

```markdown
做一个 /fix-all 命令，自动重构、安装依赖、删除无用文件并提交。
```

问题：

- 把多个高风险动作放入一个快捷命令。
- 无法可靠验收。
- 缺少人工确认和测试。

json 错误示例：

```json
{
  "permissions": {
    "allow": ["Bash(*)"]
  }
}
```

问题：

- 过宽的命令权限会让危险命令绕过人工确认。
- 应按命令、目录和任务最小化授权。

## 正确示范

settings 思路示例：

json：

```json
{
  "permissions": {
    "deny": [
      "Bash(rm -rf:*)",
      "Bash(chmod -R:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

说明：这是权限策略示意，具体字段和匹配方式应以当前 Claude Code 官方文档和本机版本为准。配置前先只读审查，避免误阻断正常命令或误放行危险命令。

skill 草案：

markdown：

```markdown
---
name: review-diff
description: 只读审查当前 git diff，重点检查安全风险、测试缺口和提交前问题。
---

# Review Diff

请只读审查当前 git diff。

禁止：
- 修改文件
- 删除文件
- 安装依赖
- 改权限
- 联网
- 提交或回退代码
- 输出或保存密钥

重点检查：
- 密钥、token、证书
- 删除文件
- 权限变化
- 批量重构
- 测试缺口

输出：
1. 阻塞问题
2. 非阻塞建议
3. 建议测试命令
4. 是否需要人工确认
```

落地方式：

- 推荐：保存为 `.claude/skills/review-diff/SKILL.md`，之后可通过 `/review-diff` 调用，也可让 Claude 在相关场景自动使用。
- 兼容：保存为 `.claude/commands/review-diff.md` 仍可创建同名命令，但新建流程优先使用 skills。

hook 设计示例：

json：

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Review risky Bash commands before allowing execution.'"
          }
        ]
      }
    ]
  }
}
```

说明：示例只做提示，不执行破坏性动作。真实项目中应让 hook 调用轻量、可审计、失败行为明确的脚本。

MCP 安全接入清单：

markdown：

```markdown
MCP 接入前检查：
- 是否必须联网。
- 是否使用最小权限账号。
- 是否避免生产写权限。
- 是否会读取密钥、用户隐私或商业敏感数据。
- 是否有日志审计。
- 是否能在项目级禁用。
- 是否能限制到特定 Agent 使用。
```

普通终端命令：

```bash
git diff -- .claude/
```

用途：审查项目级 Claude Code 配置是否引入高风险权限、hook 或 MCP。

## 练习任务

练习 1：settings 评估

Claude Code 对话输入：

```markdown
请只读检查当前项目是否已有 .claude 配置。
输出：
- 发现的配置文件。
- 可能影响权限的字段。
- 是否存在危险命令或密钥。
不要修改文件。
```

练习 2：skill 设计

Claude Code 对话输入：

```markdown
请设计一个可通过 /test-plan 调用的 skill，用于根据当前 diff 生成测试计划。
要求：
- 不运行测试。
- 不修改文件。
- 输出手动测试和自动测试建议。
先输出 SKILL.md 草案，不要写入文件。
```

练习 3：hooks 风险评审

Claude Code 对话输入：

```markdown
请评审一个计划中的 hook：在每次 Bash 工具调用前检查命令是否包含 rm、chmod、git reset、npm install。
请说明：
- 能降低什么风险。
- 可能误报什么。
- 是否应该阻断还是提醒。
```

练习 4：MCP 最小权限设计

Claude Code 对话输入：

```markdown
请为一个只读 issue tracker MCP 写接入方案。
要求：
- 不包含真实 token。
- 限制只读。
- 说明如何撤销访问。
- 说明联网审核点。
```

## 验收标准

完成本章后，你应该能够：

- 区分 settings、commands / skills、hooks、MCP 的适用场景。
- 判断配置应该放在用户级、项目级还是团队管理级。
- 编写默认只读的 skill 草案，并知道旧 `.claude/commands/` 的兼容位置。
- 识别 hooks 中自动执行危险命令的风险。
- 为 MCP 接入设计最小权限、联网审核和密钥保护方案。
- 在修改 `.claude/` 配置前先审查 diff。
- 对删除文件、批量重构、改权限、安装依赖、提交代码、联网、处理密钥设置人工确认点。

## 官方依据

- Anthropic Claude Code Docs: Claude Code settings，访问日期：2026-04-26，https://code.claude.com/docs/en/settings
- Anthropic Claude Code Docs: Commands，访问日期：2026-04-26，https://code.claude.com/docs/en/commands
- Anthropic Claude Code Docs: Extend Claude with skills，访问日期：2026-04-26，https://code.claude.com/docs/en/skills
- Anthropic Claude Code Docs: Hooks reference，访问日期：2026-04-26，https://code.claude.com/docs/en/hooks
- Anthropic Claude Code Docs: Connect Claude Code to tools via MCP，访问日期：2026-04-26，https://code.claude.com/docs/en/mcp
- Anthropic Claude Code Docs: Configure permissions，访问日期：2026-04-26，https://code.claude.com/docs/en/permissions
- Anthropic Claude Code Docs: Security，访问日期：2026-04-26，https://code.claude.com/docs/en/security

要点摘录：

- settings 用于控制 Claude Code 的配置、权限和行为。
- commands 控制 Claude Code 会话；skills 可复用常见提示词和工作流，并可创建自定义 `/name` 调用入口。
- hooks 可在生命周期事件中触发命令，但需要谨慎设计权限和失败行为。
- MCP 用于连接外部工具和数据源，应使用最小权限和明确的访问边界。
