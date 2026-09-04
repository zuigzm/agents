# AGENTS.md

> 仓库级 AI Coding Agent 工作规则。
>
> 本文件是本仓库的主要规则文件，默认使用中文。仅保存长期稳定规则；项目当前状态、当前任务、长期知识和具体执行流程分别由其他文件负责。

---

## 1. 仓库知识体系

- AGENTS.md：Agent 应该如何工作
- AGENTS_EN.md：本文件的英文对应版本
- CONTEXT.md：项目当前处于什么状态
- TASK.md：当前正在执行什么任务
- wiki/：项目长期知识、架构、模块和决策
- tasks/：任务历史与详细任务文档
- .agents/skills/：某类工作具体如何执行
- README.md：项目介绍、运行和快速入口

Agent 不应依赖历史聊天记录才能理解项目。理想流程是：git clone → 读取 AGENTS.md → 读取 CONTEXT.md → 读取 TASK.md → 读取 wiki/INDEX.md → 读取相关知识 → 继续开发。

## 2. 指令优先级

规则冲突时按以下顺序处理：

1. 用户当前明确指令
2. 根目录 AGENTS.md
3. 当前工作目录中更具体的 AGENTS.md
4. TASK.md
5. CONTEXT.md
6. wiki/ 中当前有效的架构知识
7. .agents/skills/ 中的执行规范
8. README.md / CONTRIBUTING.md
9. 当前代码行为
10. Agent 默认行为

安全规则不能被低优先级规则覆盖。

如果代码与 Wiki 不一致，不得直接假定任何一方正确。应先确认 Wiki 是否过时、代码是否属于旧实现、是否存在 Migration / Compatibility Layer，以及当前 TASK 是否正在改变该设计。禁止创建第三套不受管理的新行为。

## 3. Agent 启动协议

每次新的 Agent Session 开始时：

1. 读取根目录 AGENTS.md。
2. 读取目标目录及其父路径中适用的更具体 AGENTS.md。
3. 如果存在，读取 CONTEXT.md，确认开发阶段、架构、Legacy / New Pipeline、Feature Flags、Migration 状态、限制和风险。
4. 如果存在，读取 TASK.md，确认 Goal、Current Phase、Completed、Pending、Scope、Out of Scope、Verification 和 Next Step。
5. 读取 wiki/INDEX.md；不存在时读取 wiki/README.md，再按任务读取相关 Wiki，不要默认加载整个 Wiki。
6. 检查 git status、git branch --show-current、git log --oneline -10，必要时检查 git diff 和 git diff --stat。
7. 按“入口 → 调用链 → 领域逻辑 → 状态 → 持久化 → 外部依赖 → 测试”的顺序检查相关代码。

## 4. 核心开发原则

### 4.1 修改前先理解

修改前必须明确 Current Behavior、Expected Behavior、Gap、Likely Root Cause、Affected Modules 和 Compatibility Risk。禁止仅根据错误消息、文件名或函数名猜测后直接修改。

### 4.2 优先修复根因

优先分析完整链路：Input → Parser → Normalizer → Domain → Application → State → Persistence → Output。避免通过连续增加 if 和特殊判断堆叠临时补丁，确认问题实际发生在哪一层。

### 4.3 最小正确修改

完成目标所需的最小正确修改是默认原则。不要因为局部问题重写整个模块、更换技术栈、大规模格式化、升级所有依赖、修改无关 UI 或重构无关模块。最小修改不等于临时 Hack；如果根因需要架构调整，应解决根因。

### 4.4 保持既有行为

除非任务明确要求 Breaking Change，否则尽量保持 API、数据、配置、用户工作流和运行时兼容性。新增核心链路时，优先保留 Legacy Pipeline，通过明确机制切换到 New Pipeline，不要立即删除旧链路。

### 4.5 分离重构与行为变化

大型改动推荐分阶段：

1. 保持行为的重构
2. 引入新架构
3. 改变行为
4. 移除兼容层

不要一次同时重构、改业务、改数据库、升级依赖和改变部署。

## 5. 范围管理

开发开始前明确 IN SCOPE 和 OUT OF SCOPE。新发现的问题分类为：

- BLOCKING：阻塞当前任务，可以处理。
- RELATED：相关但不阻塞，记录到 TASK.md 或新任务。
- UNRELATED：无关，不要修改。

## 6. 架构规则

- 每个能力必须属于明确模块。不要把业务逻辑持续堆入 utils、helpers、common、misc、manager 或 service。
- 尊重依赖方向，避免循环依赖。跨模块调用优先通过 Public API、Interface、Facade、Module Export 或 Contract，避免深层 internal 路径引用。
- 复杂业务优先保持 domain/、application/、infrastructure/、interface/ 或项目已有等价结构。
- 基本方向是 Interface → Application → Domain；Infrastructure 实现 Domain / Application 所需接口。
- 核心业务规则不应绑定 HTTP、数据库 ORM、具体 AI Provider、Queue 或 UI Framework。
- 每类信息尽量只有一个 Source of Truth：规则在 AGENTS.md，状态在 CONTEXT.md，任务在 TASK.md，长期知识在 wiki/，决策在 wiki/decisions/，数据库定义在 Schema / Migration，API 合同在 OpenAPI / Shared Types，Prompt 在 Prompt Registry，可复用流程在 .agents/skills/。

## 7. 兼容性与迁移

高风险能力优先渐进迁移：Old → Compatibility Layer → New。

核心新链路、高风险重构、新 Provider、Runtime、Storage、Parser 或认证流程优先考虑 Feature Flag。Feature Flag 必须有明确默认值、行为、测试、回滚方式和未来移除计划。

数据库修改必须考虑 Existing Data、Nullability、Default、Backward Compatibility、Migration Cost、Rollback、Index Cost 和 Application Compatibility。正式 Schema 修改必须使用项目规定的 Migration 机制，不得为了快速修复重置生产数据库。

## 8. 安全规则

### 8.1 保护用户数据

未经用户明确授权，不得执行可能导致数据丢失的操作，包括但不限于：rm -rf、DROP DATABASE、DROP TABLE、TRUNCATE、prisma migrate reset、docker volume rm、docker compose down -v 及等价操作。

### 8.2 保护现有工作区修改

未经确认，禁止执行 git reset --hard、git clean -fd、git clean -fdx、git restore .、git checkout -- .。

### 8.3 禁止未经授权 Force Push

禁止自行执行 git push --force 或 git push --force-with-lease。

### 8.4 保护 Secret

不得输出真实 Secret，不得把 API Key、Password、OAuth Secret、Token、Cookie、Private Key 或 Production Credential 写入文档、日志或 Git。必要时使用 ***REDACTED***。

### 8.5 高风险操作

数据库删除、数据库 Reset、Volume 删除、Production Mutation、Credential Mutation、Force Push 和 Large Irreversible Migration 必须特别谨慎。不可逆风险操作应先取得用户明确授权。

## 9. 外部系统与 AI

LLM、HTTP API、Database、Queue、Webhook、MCP、CLI、Git Provider 和 Cloud Service 均视为不可靠依赖。

- 长期外部调用必须有合理 Timeout。不要无限增加 Timeout 掩盖根因。
- 只对 timeout、429、502、503、504、临时网络错误和临时 Provider 错误进行有限重试。
- 通常不要重试 validation error、permission error、authentication error、invalid business state 或 invalid schema。
- Retry 必须具备 max attempts、backoff 和 timeout，禁止无限重试。
- LLM 负责语义理解、生成、规划、分类、推理、总结和排序。
- 确定性代码负责 validation、authorization、state、persistence、business invariants、format conversion 和 security。
- 不要让 LLM 决定是否允许删除数据、权限是否成立、数据库是否完整或安全规则是否可以绕过。
- 结构化 AI 输出应经过：LLM Raw Output → Parser → Normalizer → Validator → Domain Object。不要直接把原始 LLM JSON 当作领域模型。

## 10. 任务管理

根目录 TASK.md 只描述当前 Workspace 正在做什么，不要无限积累历史任务。复杂任务推荐使用 tasks/active/、tasks/completed/ 和 tasks/archive/。

复杂任务每完成一个明确阶段，应更新 Progress、Verification 和 Next Step。推荐字段包括 Goal、Current Phase、Scope、Current Behavior、Expected Behavior、Findings、Execution Plan、Progress、Verification、Risks 和 Next Step。

## 11. CONTEXT.md 管理

CONTEXT.md 保存项目当前状态，适合记录 Current Architecture、Development Stage、Active Runtime、Migration Status、Legacy Compatibility Status、Feature Flags、Known Constraints、Known Issues 和 Recent Important Decisions。

不要在其中记录详细任务步骤、大量历史修改、临时 Debug 日志或每日开发日志。当开发阶段、默认 Runtime、Legacy、Migration、Feature Flag 默认值、核心能力稳定性或重大限制发生变化时，检查是否需要更新。

## 12. Wiki 管理

长期知识统一进入 wiki/。推荐按需使用 architecture/、domain/、modules/、workflows/、api/、database/、ai/、infrastructure/、deployment/、development/、decisions/、debugging/、glossary/ 和 archive/。

wiki/INDEX.md 面向 Agent 快速检索，wiki/README.md 面向开发者介绍知识库。Wiki 应记录架构、模块责任、领域概念、核心流程、状态机、数据所有权、恢复策略、数据库设计、API 原则、Prompt 架构、基础设施、部署、ADR、已确认的调试知识和术语表。

不要记录临时修改日志、临时 TODO、未验证猜测、完整 Git Changelog 或大量短期代码行号。

## 13. 架构决策

重大技术决策保存到 wiki/decisions/，推荐 ADR 字段：Status、Date、Context、Options、Decision、Reasons、Consequences、Migration 和 Revisit Conditions。

不要删除失效的重要 ADR。应标记为 Superseded，并链接到新的决策。

## 14. 调试知识

复杂、重复、定位成本高的问题可进入 wiki/debugging/，记录 Symptoms、Facts、Hypotheses、Root Cause、Fix、Verification、Prevention 和 Related Components。

必须区分 FACT 与 HYPOTHESIS；只有经过验证的结论才能写入 Root Cause。

## 15. Skills

具体操作方法放入 .agents/skills/。Skill 描述某类工作怎么做；Wiki 描述当前项目怎么工作，二者不得混淆。

## 16. 验证

不能仅以“代码已经修改”作为完成标准。优先执行与修改范围最相关的验证：Parser → parser tests，API → API tests，Workflow → integration tests，Types → typecheck，Build system → build。根据项目实际情况选择 lint、typecheck、unit test、integration test 和 build。

没有执行的测试不得声明 Passed。未验证时必须写明 Not Verified 及原因。测试失败时区分 current change、existing failure、environment failure 和 external dependency failure，不得为让结果变绿而隐藏失败。

## 17. Git 工作流

修改前至少检查 git status 和 git diff。提交前检查 git status、git diff 和 git diff --stat，确认没有 Secret、Debug 垃圾、无关修改、意外生成文件或未知用户修改。

一个 Commit 应代表一个逻辑单元，推荐前缀：feat:、fix:、refactor:、docs:、test:、chore:。复杂任务推荐按“Phase → Verify → Commit”推进。

默认 commit != push。是否 Push 由用户要求、Workspace Policy、CI 流程和 Repository Policy 决定。

## 18. 多 Agent 协作

尽量按模块、测试和文档分工，避免多个 Agent 同时修改同一核心文件。共享 Workspace 时不得覆盖其他 Agent 修改；修改前查看 Git 状态，明确修改范围，合并前统一 Review。

## 19. 工具使用

工具可用不等于操作已授权。Docker 可用不代表允许删除 Volume，Git 可用不代表允许 Force Push，数据库可访问不代表允许 Reset Database。

## 20. 自主执行

普通开发任务应自主完成：Investigate → Plan → Implement → Verify → Update Task → Update Knowledge if needed。

以下情况必须停止并请求明确授权：可能删除用户数据、Production Mutation、Credential Mutation、Force Push、Irreversible Migration、覆盖未知用户修改，或目标操作对象不明确且风险高。普通实现选择应根据 Existing Architecture、Maintainability、Compatibility、Simplicity 和 Testability 自行决定。

## 21. 知识更新协议

复杂任务完成前进行 Knowledge Impact Review，检查 Architecture、Domain、Module Responsibility、Workflow、State Machine、Database、API Contract、Runtime、Migration、Feature Flag 和重要 Root Cause 是否变化，再决定是否更新 TASK.md、CONTEXT.md、wiki/ 或 ADR。不要机械地每次修改所有文档。

## 22. 任务完成检查

- [ ] Goal completed
- [ ] Scope controlled
- [ ] Root cause addressed
- [ ] Existing behavior preserved where required
- [ ] Architecture boundaries respected
- [ ] Error handling considered
- [ ] Compatibility considered
- [ ] Relevant tests executed
- [ ] Git diff reviewed
- [ ] TASK.md updated
- [ ] CONTEXT.md reviewed
- [ ] Wiki impact reviewed
- [ ] No secrets added
- [ ] No unrelated changes

## 23. 完成报告

完成任务时提供简洁报告，包含 What Changed、Why、Main Files、Verification、Compatibility、Risks、Not Verified 和 Next Step。如果任务未全部完成，必须明确当前状态；不要声称“全部解决”，除非确实完成并验证。

## 24. 推荐仓库结构

推荐按需使用：AGENTS.md、AGENTS_EN.md、CONTEXT.md、TASK.md、README.md、CONTRIBUTING.md、wiki/、tasks/、.agents/skills/、.github/ 和 src/。项目可以裁剪结构，不要为了形式创建无意义目录。

## 25. 标准 Session 流程

START → 读取 AGENTS.md / AGENTS_EN.md → 读取 CONTEXT.md → 读取 TASK.md → 读取 wiki/INDEX.md → 定位相关知识 → 检查 Git → 检查相关代码 → 确认当前行为 → 定位根因 / 设计 → 实施一个完整阶段 → 执行针对性验证 → 检查 Git diff → 更新 TASK.md → 检查 CONTEXT 与 Wiki 影响 → 按需提交 → 下一阶段 / 完成。

## 26. 最终原则

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
