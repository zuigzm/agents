# AGENTS.md

> Repository-level instructions for AI Coding Agents.
>
> 本文件定义本仓库中 AI Coding Agent 的长期工作规则。
>
> 适用于 Codex、Claude Code、OpenCode、GitHub Copilot Agent，以及其他能够读取仓库、修改文件、执行终端命令和使用外部工具的开发 Agent。
>
> 本文件只保存**长期稳定规则**。
>
> 项目当前状态、当前任务、长期知识和具体操作方法分别由其他文件负责，不应全部堆积在本文件中。

---

# 1. Repository Knowledge Model

本项目使用以下知识体系：





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

.agents/skills/
→ 某类工作具体如何执行

README.md
→ 项目介绍、运行和快速入口





Agent 不应依赖历史聊天记录才能理解项目。

理想状态：





git clone
↓
read AGENTS.md
↓
read CONTEXT.md
↓
read TASK.md
↓
read wiki/INDEX.md
↓
读取相关知识
↓
继续开发





---

# 2. Instruction Priority

发生规则冲突时，优先级如下：





1. 用户当前明确指令
2. 根目录 AGENTS.md
3. 当前工作目录更具体的 AGENTS.md
4. TASK.md
5. CONTEXT.md
6. wiki/ 中的当前有效架构知识
7. .agents/skills/ 中的执行规范
8. README / CONTRIBUTING
9. 当前代码行为
10. Agent 默认行为





安全规则不能被低优先级规则覆盖。

如果：





代码
≠
Wiki





不得直接假定任何一方正确。

应先确认：

* Wiki 是否已经过时；
* 代码是否属于旧实现；
* 是否存在 Migration / Compatibility Layer；
* 当前 TASK 是否正在改变该设计。

然后再修改。

禁止创建第三套不受管理的新行为。

---

# 3. Agent Startup Protocol

每次新的 Agent Session 开始时，按照以下顺序执行。

## 3.1 Read AGENTS.md

首先读取：

AGENTS.md

如果目标目录存在更具体的：

*/AGENTS.md

也必须读取。

子目录 AGENTS.md 只约束其目录范围。

子规则可以更具体，但不得绕过根目录安全规则。

## 3.2 Read CONTEXT.md

如果存在 CONTEXT.md，必须读取。

重点确认：

* 当前开发阶段；
* 当前架构；
* 已启用能力；
* Legacy / New Pipeline 状态；
* Feature Flags；
* Migration 状态；
* 当前限制；
* 已知风险。

CONTEXT.md 是项目当前状态快照，不是任务计划，也不是历史日志。

## 3.3 Read TASK.md

如果存在 TASK.md，必须读取。

确认：

Goal、Current Phase、Completed、Pending、Scope、Out of Scope、Verification、Next Step。

不要重复已经完成的工作，不要擅自扩大任务范围。

## 3.4 Read Wiki Index

读取 wiki/INDEX.md；如果不存在，则读取 wiki/README.md。

根据当前任务定位相关知识，不要默认加载整个 Wiki。

## 3.5 Inspect Git State

执行或等价检查：

git status

必要时检查 git diff 和 git diff --stat。

确认当前分支、未提交修改、其他 Agent / 用户的工作，以及最近提交与当前任务的关系。

## 3.6 Inspect Relevant Code

按以下顺序探索：

入口 → 调用链 → 领域逻辑 → 状态 → 持久化 → 外部依赖 → 测试。

只读取与当前任务相关的代码，不要无目的扫描整个仓库。

---

# 4. Core Development Principles

## 4.1 Understand Before Modify

修改之前必须理解：

Current Behavior、Expected Behavior、Gap、Likely Root Cause、Affected Modules、Compatibility Risk。

禁止仅根据错误消息、文件名或函数名猜测后直接修改。

## 4.2 Root Cause First

修复问题优先处理根因，避免不断堆叠特殊判断。

优先分析：

Input → Parser → Normalizer → Domain → Application → State → Persistence → Output。

确认问题发生在哪一层。

## 4.3 Minimal Correct Change

默认原则：完成目标所需的最小正确修改。

不要因为局部问题重写整个模块、更换技术栈、大规模格式化、升级所有依赖、顺手修改无关 UI，或顺手重构其他模块。

最小修改不等于临时 Hack；如果根因需要架构调整，应解决根因。

## 4.4 Preserve Existing Behavior

除非任务明确要求 Breaking Change，否则必须尽量保持 API、数据、配置、用户工作流和运行时兼容性。

新增核心链路时优先保留 Legacy Pipeline，并通过明确机制切换到 New Pipeline，而不是立即删除旧链路。

## 4.5 Separate Refactor From Behavior Change

大型改动推荐分阶段：

1. Behavior-preserving refactor
2. Introduce new architecture
3. Change behavior
4. Remove compatibility layer

不要一次同时重构、改业务、改数据库、升级依赖和改变部署。

---

# 5. Scope Management

开始开发前，应明确 IN SCOPE 和 OUT OF SCOPE。

开发过程中发现其他问题时分类为 BLOCKING、RELATED 或 UNRELATED。

### BLOCKING

阻塞当前任务，可以处理。

### RELATED

与当前问题相关但不阻塞，记录到 TASK.md 或新任务。

### UNRELATED

不要修改。

---

# 6. Architecture Rules

## 6.1 Respect Module Ownership

每个能力必须属于明确模块。避免把大量业务逻辑不断放入 utils、helpers、common、misc、manager 或 service，应能回答“这个能力属于哪个领域或技术模块？”

## 6.2 Respect Dependency Direction

避免形成循环依赖。跨模块调用优先通过 Public API、Interface、Facade、Module Export 或 Contract，避免深层 internal 路径引用。

## 6.3 Domain Before Infrastructure

复杂业务推荐保持 domain/application/infrastructure/interface 或项目现有的等价架构。

基本原则：Interface → Application → Domain。Infrastructure 实现 Domain / Application 所需接口。

核心业务规则不应绑定 HTTP、数据库 ORM、具体 AI Provider、Queue 或 UI Framework。

## 6.4 Stable Source of Truth

每类信息尽量保持唯一 Source of Truth：Agent Rules → AGENTS.md；Current Project State → CONTEXT.md；Current Task → TASK.md；Project Knowledge → wiki/；Architecture Decisions → wiki/decisions/；Database Definition → ORM Schema / Migration；Exact API Contract → OpenAPI / Shared Types；Prompt Definition → Prompt Registry；Reusable Agent Procedures → .agents/skills/。

不要维护多个互相漂移的副本。

---

# 7. Compatibility and Migration

## 7.1 Prefer Incremental Migration

高风险能力应优先渐进迁移：Old → Compatibility Layer → New。

## 7.2 Feature Flags

核心新链路、高风险重构、新 Provider、Runtime、Storage、Parser 或认证流程，优先考虑 Feature Flag。

Feature Flag 必须有明确默认值、行为、测试、回滚方式和未来移除计划。

## 7.3 Database Safety

数据库修改必须考虑 Existing Data、Nullability、Default、Backward Compatibility、Migration Cost、Rollback、Index Cost 和 Application Compatibility。

正式 Schema 修改应使用项目规定的 Migration 机制，不得为了快速修复直接重置生产数据库。

---

# 8. Safety Rules

本节优先级高于普通开发规则。

## 8.1 Protect User Data

未经用户明确授权，不得执行可能导致数据丢失的操作，包括 rm -rf、DROP DATABASE、DROP TABLE、TRUNCATE、prisma migrate reset、docker volume rm、docker compose down -v 及等价操作。

## 8.2 Protect Existing Workspace Changes

禁止未经确认执行 git reset --hard、git clean -fd、git clean -fdx、git restore .、git checkout -- .。

## 8.3 Never Force Push Without Explicit Authorization

禁止自行执行 git push --force 或 git push --force-with-lease。

## 8.4 Protect Secrets

不得输出真实 Secret、把 Secret 写入文档或日志，或提交 Git。必要时使用 ***REDACTED***。

## 8.5 High-risk Operations

数据库删除、数据库 Reset、Volume 删除、Production Mutation、Credential Mutation、Force Push 和 Large Irreversible Migration 必须特别谨慎；不可逆风险操作应先取得用户明确授权。

---

# 9. External Systems and AI

项目中的外部系统均视为不可靠依赖，包括 LLM、HTTP API、Database、Queue、Webhook、MCP、CLI、Git Provider 和 Cloud Service。

## 9.1 Timeout

所有长期外部调用应有合理 Timeout。修改 Timeout 前先判断真正原因，不要无限增加 Timeout 掩盖根因。

## 9.2 Retry

适合 Retry：timeout、429、502、503、504、temporary network failure、temporary provider failure。

通常不应 Retry：validation error、permission error、authentication error、invalid business state、invalid schema。

Retry 应具备 max attempts、backoff 和 timeout，禁止无限重试。

## 9.3 LLM Boundary

LLM 负责 semantic understanding、generation、planning、classification、reasoning、summarization 和 ranking。

确定性代码负责 validation、authorization、state、persistence、business invariants、format conversion 和 security。

不要让 LLM 决定是否允许删除数据、权限是否成立、数据库是否完整或安全规则是否绕过。

## 9.4 Structured AI Output

推荐流程：LLM Raw Output → Parser → Normalizer → Validator → Domain Object。

不要直接把原始 LLM JSON 当作领域模型。

---

# 10. Task Management

根目录 TASK.md 只描述当前 Workspace 正在做什么，不要无限积累历史任务。

复杂任务推荐使用 tasks/active、tasks/completed 和 tasks/archive。根 TASK.md 可以作为当前任务摘要和入口。

推荐 TASK.md 结构：Goal、Current Phase、Scope、Current Behavior、Expected Behavior、Findings、Execution Plan、Progress、Verification、Risks、Next Step。

复杂任务每完成一个明确阶段，应更新 Progress、Verification 和 Next Step。

---

# 11. CONTEXT.md Management

CONTEXT.md 保存项目当前状态，适合记录 Current Architecture、Development Stage、Active Runtime、Migration Status、Legacy Compatibility Status、Feature Flags、Known Constraints、Known Issues 和 Recent Important Decisions。

不适合记录详细任务步骤、大量历史修改、临时 Debug 日志或每日开发日志。

当开发阶段、默认 Runtime、Legacy、Migration、Feature Flag 默认值、核心能力稳定性或重大限制发生变化时，检查是否更新 CONTEXT.md。

---

# 12. Wiki Knowledge Management

项目长期知识统一进入 wiki/，推荐使用 README.md、INDEX.md、architecture/、domain/、modules/、workflows/、api/、database/、ai/、infrastructure/、deployment/、development/、decisions/、debugging/、glossary/ 和 archive/（按需裁剪）。

wiki/INDEX.md 面向 Agent 快速检索；wiki/README.md 面向开发者介绍知识库和推荐阅读顺序。

适合 Wiki 的内容包括 Architecture、Module Responsibilities、Domain Concepts、Core Workflows、State Machines、Data Ownership、Recovery Strategy、Database Design、API Design Principles、Prompt Architecture、Infrastructure、Deployment、ADR、Confirmed Debugging Knowledge 和 Glossary。

不要记录临时修改日志、临时 TODO、未验证猜测、完整 Git Changelog 或不稳定的具体代码行号。

---

# 13. Architecture Decisions

重大技术决策保存到 wiki/decisions/，推荐 ADR 格式：Status、Date、Context、Options、Decision、Reasons、Consequences、Migration、Revisit Conditions。

不要删除失效的重要 ADR，应标记为 Superseded 并链接到新 ADR。

---

# 14. Debugging Knowledge

复杂、重复、定位成本高的问题可以进入 wiki/debugging/，记录 Symptoms、Facts、Hypotheses、Root Cause、Fix、Verification、Prevention 和 Related Components。

必须区分 FACT 和 HYPOTHESIS；只有经过验证的结论才能写入 Root Cause。

---

# 15. Skills

具体操作方法放入 .agents/skills/。Skill 描述某类工作怎么做；Wiki 描述当前项目本身怎么工作，二者不得混淆。

---

# 16. Verification

不能仅以“代码已经修改”作为完成标准。

## 16.1 Targeted Verification First

根据修改范围执行最相关验证：Parser → parser tests；API → API tests；Workflow → integration tests；Types → typecheck；Build system → build。

## 16.2 Typical Verification

根据项目实际情况选择 lint、typecheck、unit test、integration test 和 build。

## 16.3 Never Fake Verification

没有执行的测试不得声明 Passed；未验证时明确写 Not Verified 并说明原因。

## 16.4 Failed Verification

测试失败时区分 current change、existing failure、environment failure 和 external dependency failure，不得为让结果变绿而隐藏失败。

---

# 17. Git Workflow

## 17.1 Before Modify

至少检查 git status 和 git diff。

## 17.2 Before Commit

检查 git status、git diff 和 git diff --stat，确认没有 Secret、Debug 垃圾、无关修改、意外生成文件或未知用户修改。

## 17.3 Commit Scope

一个 Commit 应代表一个逻辑单元，推荐使用 feat:、fix:、refactor:、docs:、test:、chore: 前缀。

## 17.4 Complex Tasks

推荐 Phase → Verify → Commit，避免积累大量无关修改后一次提交。

## 17.5 Commit Is Not Push

默认 commit != push。Push 根据用户要求、Workspace Policy、CI 流程和 Repository Policy 决定，不得自动 Force Push。

---

# 18. Multi-Agent Collaboration

多个 Agent 协作时尽量按模块、测试和文档分工，避免同时修改同一核心文件。

共享 Workspace 时不得覆盖其他 Agent 修改；修改前查看 Git 状态，明确范围，合并前统一 Review。

---

# 19. Tool Usage

工具可用不等于操作已授权。例如 Docker 可用不代表允许删除 Volume，Git 可用不代表允许 Force Push，数据库可访问不代表允许 Reset Database。

---

# 20. Autonomous Execution

普通开发任务应自主完成：Investigate → Plan → Implement → Verify → Update Task → Update Knowledge if needed。

以下情况需要明确授权或停止危险操作：可能删除用户数据、Production Mutation、Credential Mutation、Force Push、Irreversible Migration、覆盖未知用户修改，或目标操作对象不明确且风险高。

普通实现选择应根据 Existing Architecture、Maintainability、Compatibility、Simplicity 和 Testability 自行决定。

---

# 21. Knowledge Update Protocol

完成复杂任务前进行 Knowledge Impact Review，检查 Architecture、Domain、Module Responsibility、Workflow、State Machine、Database、API Contract、Runtime、Migration、Feature Flag 和重要 Root Cause 是否变化，然后决定是否更新 TASK.md、CONTEXT.md、wiki/ 或 ADR。

不要机械地每次修改所有文档。

---

# 22. Task Completion Protocol

任务完成前至少检查：

* Goal completed
* Scope controlled
* Root cause addressed
* Existing behavior preserved where required
* Architecture boundaries respected
* Error handling considered
* Compatibility considered
* Relevant tests executed
* Git diff reviewed
* TASK.md updated
* CONTEXT.md reviewed
* Wiki impact reviewed
* No secrets added
* No unrelated changes

---

# 23. Completion Report

完成任务时提供简洁报告，包含 What Changed、Why、Main Files、Verification、Compatibility、Risks、Not Verified 和 Next Step。

如果任务未全部完成，应明确当前状态。不要声称“全部解决”，除非确实完成并验证。

---

# 24. Recommended Repository Structure

推荐结构包括：

AGENTS.md、CONTEXT.md、TASK.md、README.md、CONTRIBUTING.md、wiki/、tasks/、.agents/skills/、.github/ 和 src/。项目可以根据需要裁剪，不要为了形式创建无意义目录。

---

# 25. Standard Agent Session Flow

START → Read AGENTS.md → Read CONTEXT.md → Read TASK.md → Read wiki/INDEX.md → Find Relevant Wiki → Inspect Git → Inspect Relevant Code → Establish Current Behavior → Identify Root Cause / Design → Implement One Coherent Phase → Run Targeted Verification → Review Git Diff → Update TASK.md → Review CONTEXT Impact → Review Wiki Impact → Commit When Appropriate → NEXT PHASE / COMPLETE。

---

# 26. Final Principles

> **先理解，再修改。**
>
> **修根因，不堆补丁。**
>
> **保持明确的模块边界。**
>
> **新能力优先渐进迁移，不轻易破坏旧链路。**
>
> **数据安全高于开发速度。**
>
> **不要覆盖未知的用户修改。**
>
> **AI 负责语义，确定性代码负责规则和状态。**
>
> **AGENTS 管规则，CONTEXT 管状态，TASK 管当前工作，Wiki 管长期知识，Skills 管执行方法。**
>
> **代码、知识、任务和架构决策必须能够通过 Git 一起追踪。**
>
> **任何新的开发者或 Coding Agent 都应该能够仅依赖仓库恢复项目上下文。**

最终目标不是让 Agent 尽可能多写代码，而是让 Agent 能够安全、准确、连续、可验证地维护一个长期演进的软件项目。
