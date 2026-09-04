# AI Coding Agent Repository Instructions

这是一个面向 AI Coding Agent 的通用仓库规则包，用于指导 Codex、Claude Code、OpenCode、GitHub Copilot Agent 以及其他能够读取仓库并修改代码的开发 Agent。

它不提供具体业务代码，而是提供一套可复用、可审查、可持续维护的 Agent 工作规范，使 Agent 能够在长期演进的软件项目中安全、准确、连续、可验证地工作。

## 这套规则是做什么的？

规则主要用于指导以下行为：

- Agent 开始工作前应该读取哪些项目资料；
- 如何理解当前代码、任务、架构和长期知识；
- 如何确认当前行为、预期行为、差距和根因；
- 如何控制任务范围，避免修改无关内容；
- 如何保持模块边界、依赖方向和兼容性；
- 如何渐进迁移 Legacy Pipeline、Runtime、Provider、Storage 或 Parser；
- 哪些数据库、Git、工作区和 Secret 操作必须禁止或先请求授权；
- 如何处理外部 API、LLM、超时、重试和结构化 AI 输出；
- 如何执行针对性测试、检查 Git diff 和报告未验证内容；
- 如何在多 Agent 协作时保护其他用户或 Agent 的未提交修改；
- 如何维护 TASK、CONTEXT、Wiki 和架构决策。

核心原则：

> 先理解，再修改；修根因，不堆补丁；数据安全高于开发速度。

## 文件说明

| 文件 | 用途 |
| --- | --- |
| AGENTS.md | 中文主版本。大多数 Agent 会自动读取项目根目录中的该文件。 |
| AGENTS_EN.md | 全英文对应版本，适合英文团队或英文工作环境。 |
| README.md | 规则用途、安装方法和使用说明。 |

AGENTS.md 是默认的权威入口。AGENTS_EN.md 与中文版本保持同一套规则，但不同 Agent 对英文文件名的自动发现约定不同，因此英文版本通常需要显式指定，或复制为目标工具支持的规则文件名。

## Agent 安装方式

将下面这句话直接复制给目标 AI Agent：

> 请从 https://github.com/zuigzm/agents 获取最新的 AGENTS.md 和 AGENTS_EN.md，检查当前项目根目录是否已有规则文件及未提交修改，确认不会覆盖用户内容后，将规则安装到当前项目根目录，并验证安装结果；除非我明确要求，不要自动提交、推送或修改其他项目文件。

如果只需要中文主规则，可使用：

> 请从 https://github.com/zuigzm/agents 获取最新的 AGENTS.md，检查当前项目根目录是否已有规则文件及未提交修改，确认不会覆盖用户内容后，将 AGENTS.md 安装到当前项目根目录并验证安装结果；除非我明确要求，不要自动提交、推送或修改其他项目文件。

## 不同 Agent 的使用方式

### Codex、Claude Code、OpenCode 和通用 Coding Agent

将 AGENTS.md 放在项目根目录。启动 Agent 后，它通常会自动读取该文件，并将规则应用于整个项目。

如果在子目录中放置更具体的 AGENTS.md，该文件通常只约束对应目录范围。子目录规则可以补充局部要求，但不能绕过根目录安全规则。

### GitHub Copilot Agent

将 AGENTS.md 放在仓库根目录并提交到 Git。具体生效方式可能取决于 Copilot Agent、IDE 和仓库配置。如果当前环境不自动识别 AGENTS.md，请在项目说明或 Agent 自定义指令中显式引用它。

### 使用英文规则

AGENTS_EN.md 是英文版本，但不是所有 Agent 都会自动识别这个文件名。可以采用以下方式：

1. 保留中文 AGENTS.md 作为默认规则，并在启动提示中要求 Agent 同时读取 AGENTS_EN.md；
2. 在英文团队项目中，将 AGENTS_EN.md 复制为该工具约定的规则文件名；
3. 将英文内容合并到项目现有的 Agent 指令文件中。

不要在同一个项目中让中文和英文文件包含冲突规则。发生差异时，以项目明确指定的主规则文件和用户当前指令为准。

## 与项目上下文文件配合使用

这套规则将不同类型的信息分开管理：

    AGENTS.md
    → Agent 应该如何工作

    CONTEXT.md
    → 项目当前处于什么状态

    TASK.md
    → 当前正在执行什么任务

    wiki/
    → 项目长期知识、架构、模块和决策

    tasks/
    → 任务历史与详细任务文档

如果目标项目还没有这些文件，可以按实际需要逐步创建，不需要一次性创建全部目录。

推荐启动顺序：读取 AGENTS.md；读取适用的更具体 AGENTS.md；读取 CONTEXT.md；读取 TASK.md；读取 wiki/INDEX.md 或 wiki/README.md；检查 Git 状态；检查相关代码和测试。

## 安装后的验证

macOS、Linux 或 Git Bash：

    test -f AGENTS.md && echo "AGENTS.md installed"
    test -f AGENTS_EN.md && echo "AGENTS_EN.md installed"
    head -n 8 AGENTS.md
    head -n 8 AGENTS_EN.md

Windows PowerShell：

    Test-Path .\AGENTS.md
    Test-Path .\AGENTS_EN.md
    Get-Content .\AGENTS.md -TotalCount 8
    Get-Content .\AGENTS_EN.md -TotalCount 8

还应确认：

- 文件位于目标项目根目录；
- 文件没有被 .gitignore 忽略；
- 文件已经提交到 Git；
- 没有覆盖目标项目原有的 AGENTS.md；
- 如果原项目已有规则文件，应先人工合并；
- 中文和英文规则没有冲突；
- 没有写入 Secret、Token、密码等敏感信息。

## 更新规则

如果使用了本地规则仓库，可以先获取最新版本：

    git -C /path/to/agents pull --ff-only

更新到目标项目时，建议先检查差异：

    git diff --no-index /path/to/your-project/AGENTS.md /path/to/agents/AGENTS.md || true
    git diff --no-index /path/to/your-project/AGENTS_EN.md /path/to/agents/AGENTS_EN.md || true

确认没有项目特有规则被覆盖后，再复制或合并文件。规则文件应随目标项目一起进行代码审查和版本管理。

## 注意事项

- 安装规则不会自动修改项目代码、数据库、部署配置或凭据。
- AGENTS.md 是指导文件，不是权限系统；Agent 仍必须遵守用户授权、项目策略和运行环境安全限制。
- 规则文件不能替代测试、代码审查、权限控制、数据库迁移和 Secret 管理。
- 不要把临时任务日志、未验证猜测或真实凭据写入规则文件。
- 如果项目已有组织级、平台级或用户级规则，应遵守实际生效的优先级。

## 维护约定

修改规则时应保持中文主版本和 AGENTS_EN.md 的规则一致。新增、删除或改变重要规则时，应同步更新两个版本，并在提交说明中写清楚变化原因。
