# 08-测试专题：让 Claude Code 编写、运行和修复测试

本章目标：让 Claude Code 先识别项目测试框架，再按现有风格补充回归测试、运行测试并修复失败。

## 使用场景

适合补测试覆盖、为 bug 增加回归测试、为新功能写验收测试、理解已有测试结构，或让 Claude Code 帮你把失败测试修到合理通过。测试任务的重点不是“让绿灯亮”，而是确认测试表达的行为正确。

高风险提醒：运行测试可能写临时文件、访问数据库、启动服务、联网或依赖环境变量。不要让 Claude Code 擅自安装依赖、删除测试数据、修改权限、提交代码、读取密钥或把真实凭据写进测试。

普通终端命令：

```bash
cd /path/to/project
claude
```

Claude Code 对话输入：

```markdown
请先识别这个项目的测试框架和测试约定，不要写代码。
请查看 package.json、测试目录、CI 配置和已有测试样例。
输出：测试框架、命名约定、常用命令、测试数据策略、我应该补测试的位置。
```

## 推荐提示词模板

Claude Code 对话输入：

```markdown
请帮我为这个行为补测试。

目标行为：
[写清楚要验证的行为]

流程：
1. 先只读识别测试框架、断言风格和测试目录
2. 找 2 到 3 个最相似的已有测试作为风格参考
3. 给出测试计划，不要先改
4. 等我确认后，新增或修改最少测试文件
5. 运行最小相关测试
6. 如果失败，先解释失败原因，再修复

约束：
- 不要改生产代码，除非测试暴露真实 bug 且我确认
- 不要安装依赖、联网、删除文件、改权限、提交代码或处理密钥
- 测试中不要使用真实 token、真实账号或生产地址
```

确认后：

Claude Code 对话输入：

```markdown
请按测试计划实现。优先复用现有测试工具和 fixture，不要引入新测试库。完成后运行最小测试命令。
```

## 错误示范

Claude Code 对话输入：

```markdown
帮我把覆盖率提到 90%。
```

问题：目标过大，容易产生低质量测试或大范围修改。

Claude Code 对话输入：

```markdown
测试失败了，你随便改到通过就行。
```

问题：可能修改测试期望来迎合错误行为，而不是修复代码。

普通终端命令：

```bash
npm install some-new-test-library && npm test
```

问题：安装依赖是高风险动作，必须先确认必要性。多数项目应复用已有测试框架。

## 正确示范

Claude Code 对话输入：

```markdown
请为“邮箱为空时注册失败”补一个回归测试。
限制范围：优先只改 tests/register.test.ts。
请先查看附近测试风格，输出测试计划，不要立刻编辑。
```

Claude Code 对话输入：

```markdown
计划可以。请只新增这个测试用例，不要重构测试文件。运行注册相关测试即可。
```

普通终端命令：

```bash
npm test -- tests/register.test.ts
```

如果项目使用 Python：

普通终端命令：

```bash
pytest tests/test_register.py
```

如果项目使用 PowerShell 环境：

PowerShell 普通终端命令：

```powershell
npm test -- tests/register.test.ts
```

## 练习任务

1. 选择一个已有函数或接口，找出它目前缺少的边界测试。
2. 让 Claude Code 先识别测试框架和相似测试。
3. 要求它输出测试计划，不要先改。
4. 确认后补一个最小回归测试。
5. 运行相关测试。
6. 如果测试失败，让 Claude Code 判断是测试写错、环境问题，还是生产代码有 bug。

进阶练习：让 Claude Code 列出 5 个边界条件，但只实现其中最重要的 1 个。

## 验收标准

- Claude Code 先识别了测试框架、目录、命名和断言风格。
- 新测试复用现有工具、fixture 和风格，没有随意引入依赖。
- 测试验证的是清晰业务行为，不只是覆盖行数。
- 运行了最小相关测试，并记录命令和结果。
- 测试失败时有原因分析，没有直接乱改期望。
- 没有使用真实密钥、真实账号、生产地址或未经确认的联网动作。

## 官方依据

- Anthropic Claude Code Common workflows，访问日期：2026-04-26，https://code.claude.com/docs/en/common-workflows
- Anthropic Claude Code Permissions，访问日期：2026-04-26，https://code.claude.com/docs/en/permissions
- Anthropic Claude Code Security，访问日期：2026-04-26，https://code.claude.com/docs/en/security
- Anthropic Claude Code CLI reference，访问日期：2026-04-26，https://code.claude.com/docs/en/cli-reference

