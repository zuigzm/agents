# Agent Instructions

> Repository-level rules for AI coding agents.
>
> This is the complete English counterpart of the primary Chinese AGENTS.md. The Chinese file is the default repository instruction file. Both files contain the same long-term operating rules.

---

## 1. Repository Knowledge Model

- AGENTS.md: how an agent should work
- AGENTS_EN.md: the English counterpart of the primary rules
- CONTEXT.md: the current project state
- TASK.md: the task currently in progress
- wiki/: long-term project knowledge, architecture, modules, and decisions
- tasks/: task history and detailed task documents
- .agents/skills/: procedures for specific types of work
- README.md: project overview, setup, and quick entry points

Agents must not depend on historical chat messages to understand the project. The ideal flow is: git clone → read AGENTS.md → read CONTEXT.md → read TASK.md → read wiki/INDEX.md → read task-relevant knowledge → continue development.

## 2. Instruction Priority

Resolve conflicts in this order:

1. The user's current explicit instruction
2. Root-level AGENTS.md
3. A more specific AGENTS.md in the current working directory or a child directory
4. TASK.md
5. CONTEXT.md
6. Current architecture knowledge in wiki/
7. Procedures in .agents/skills/
8. README.md / CONTRIBUTING.md
9. Current code behavior
10. The agent's default behavior

Safety rules always override lower-priority rules.

When code and Wiki disagree, do not assume either one is correct. First determine whether the Wiki is stale, the code is a legacy implementation, a Migration / Compatibility Layer exists, or the current TASK is changing the design. Do not create a third unmanaged behavior.

## 3. Agent Startup Protocol

At the start of every new Agent Session:

1. Read the root AGENTS.md.
2. Read any more specific applicable AGENTS.md files.
3. If present, read CONTEXT.md and confirm the development stage, architecture, Legacy / New Pipeline status, Feature Flags, Migration status, constraints, and risks.
4. If present, read TASK.md and confirm Goal, Current Phase, Completed, Pending, Scope, Out of Scope, Verification, and Next Step.
5. Read wiki/INDEX.md; if absent, read wiki/README.md. Then read only the Wiki knowledge relevant to the task.
6. Inspect git status, git branch --show-current, and git log --oneline -10. Inspect git diff and git diff --stat when necessary.
7. Inspect relevant code in this order: entry point → call chain → domain logic → state → persistence → external dependencies → tests.

## 4. Core Development Principles

### 4.1 Understand Before Modifying

Before modifying code, establish Current Behavior, Expected Behavior, Gap, Likely Root Cause, Affected Modules, and Compatibility Risk. Do not change code based only on an error message, filename, or function name.

### 4.2 Fix Root Causes First

Analyze the complete path: Input → Parser → Normalizer → Domain → Application → State → Persistence → Output. Avoid accumulating temporary patches through repeated if statements and special cases. Identify the actual layer where the problem occurs.

### 4.3 Minimal Correct Change

Make the smallest correct change needed to achieve the goal. Do not rewrite a module, replace the technology stack, perform broad formatting, upgrade every dependency, alter unrelated UI, or refactor unrelated modules because of a local issue. Minimal does not mean a temporary hack; address architectural causes when necessary.

### 4.4 Preserve Existing Behavior

Unless a Breaking Change is explicitly requested, preserve API, data, configuration, user workflow, and runtime compatibility as far as practical. For a new core pipeline, prefer retaining the Legacy Pipeline with an explicit switching mechanism instead of removing it immediately.

### 4.5 Separate Refactoring from Behavior Changes

For large changes, use these phases: behavior-preserving refactor; introduce the new architecture; change behavior; remove the compatibility layer. Do not combine refactoring, business changes, database changes, dependency upgrades, and deployment changes in one unstructured step.

## 5. Scope Management

Define IN SCOPE and OUT OF SCOPE before implementation. Classify newly discovered issues as follows:

- BLOCKING: blocks the current task and may be addressed now.
- RELATED: related but non-blocking; record it in TASK.md or a new task.
- UNRELATED: do not modify it.

## 6. Architecture Rules

- Every capability must belong to a clear module. Do not continuously place business logic in utils, helpers, common, misc, manager, or service.
- Respect dependency direction and avoid circular dependencies. Prefer Public APIs, Interfaces, Facades, Module Exports, or Contracts for cross-module calls; avoid deep internal imports.
- For complex business logic, preserve domain/, application/, infrastructure/, and interface/ boundaries, or the project's established equivalent.
- The basic direction is Interface → Application → Domain; Infrastructure implements interfaces required by Domain / Application.
- Core business rules must not be coupled to HTTP, a database ORM, a specific AI Provider, Queue, or UI Framework.
- Keep one Source of Truth for each kind of information: rules in AGENTS.md, state in CONTEXT.md, the active task in TASK.md, long-term knowledge in wiki/, decisions in wiki/decisions/, database definitions in Schema / Migration, API contracts in OpenAPI / Shared Types, prompts in a Prompt Registry, and reusable procedures in .agents/skills/.

## 7. Compatibility and Migration

Prefer incremental migration for high-risk capabilities: Old → Compatibility Layer → New.

Consider Feature Flags for core new pipelines, high-risk refactors, new Providers, Runtimes, Storage, Parsers, and authentication flows. Every Feature Flag must have a defined default, behavior, test coverage, rollback path, and removal plan.

Database changes must account for Existing Data, Nullability, Default, Backward Compatibility, Migration Cost, Rollback, Index Cost, and Application Compatibility. Use the repository's approved Migration mechanism for formal Schema changes. Never reset a production database as a quick fix.

## 8. Safety Rules

### 8.1 Protect User Data

Without explicit user authorization, do not perform operations that may cause data loss, including rm -rf, DROP DATABASE, DROP TABLE, TRUNCATE, prisma migrate reset, docker volume rm, docker compose down -v, or equivalent operations.

### 8.2 Protect Existing Workspace Changes

Without confirmation, do not run git reset --hard, git clean -fd, git clean -fdx, git restore ., or git checkout -- .

### 8.3 No Unauthorized Force Push

Never independently run git push --force or git push --force-with-lease.

### 8.4 Protect Secrets

Never expose real Secrets or write API Keys, Passwords, OAuth Secrets, Tokens, Cookies, Private Keys, or Production Credentials to documentation, logs, or Git. Use ***REDACTED*** when necessary.

### 8.5 High-Risk Operations

Database deletion, database reset, volume deletion, Production Mutation, Credential Mutation, Force Push, and Large Irreversible Migration require special caution. Obtain explicit authorization before an irreversible high-risk operation.

## 9. External Systems and AI

Treat LLMs, HTTP APIs, Databases, Queues, Webhooks, MCP, CLIs, Git Providers, and Cloud Services as unreliable dependencies.

- Long-running external calls require reasonable timeouts. Do not increase timeouts indefinitely to hide the root cause.
- Retry only transient failures such as timeout, 429, 502, 503, 504, temporary network failures, and temporary Provider failures, with limits.
- Normally do not retry validation errors, permission errors, authentication errors, invalid business states, or invalid schemas.
- Retries must have max attempts, backoff, and timeout. Never retry forever.
- LLMs may handle semantic understanding, generation, planning, classification, reasoning, summarization, and ranking.
- Deterministic code must handle validation, authorization, state, persistence, business invariants, format conversion, and security.
- Do not let an LLM decide whether data may be deleted, whether authorization is valid, whether a database is complete, or whether security rules may be bypassed.
- Structured AI output should follow LLM Raw Output → Parser → Normalizer → Validator → Domain Object. Never treat raw LLM JSON as a domain model directly.

## 10. Task Management

The root TASK.md describes only what the current Workspace is doing; it must not become an unlimited history log. For complex work, use tasks/active/, tasks/completed/, and tasks/archive/.

After each meaningful phase, update Progress, Verification, and Next Step. Recommended fields include Goal, Current Phase, Scope, Current Behavior, Expected Behavior, Findings, Execution Plan, Progress, Verification, Risks, and Next Step.

## 11. CONTEXT.md Management

CONTEXT.md stores the current project state. It may contain Current Architecture, Development Stage, Active Runtime, Migration Status, Legacy Compatibility Status, Feature Flags, Known Constraints, Known Issues, and Recent Important Decisions.

Do not use it for detailed task steps, extensive modification history, temporary debug logs, or daily development notes. Review it when the development stage, default Runtime, Legacy status, Migration status, Feature Flag defaults, capability maturity, or major constraints change.

## 12. Wiki Management

Store long-term knowledge in wiki/. Use architecture/, domain/, modules/, workflows/, api/, database/, ai/, infrastructure/, deployment/, development/, decisions/, debugging/, glossary/, and archive/ as needed.

wiki/INDEX.md is for fast agent lookup; wiki/README.md introduces the knowledge base to developers. Wiki content should cover architecture, module responsibilities, domain concepts, core workflows, state machines, data ownership, recovery strategy, database design, API principles, prompt architecture, infrastructure, deployment, ADRs, confirmed debugging knowledge, and terminology.

Do not record temporary modification logs, temporary TODOs, unverified guesses, a complete Git changelog, or large amounts of short-lived line-specific information.

## 13. Architecture Decisions

Store major technical decisions in wiki/decisions/. Recommended ADR fields are Status, Date, Context, Options, Decision, Reasons, Consequences, Migration, and Revisit Conditions.

Do not delete important obsolete ADRs. Mark them Superseded and link to the replacement decision.

## 14. Debugging Knowledge

Document complex, recurring, or expensive-to-locate problems in wiki/debugging/ with Symptoms, Facts, Hypotheses, Root Cause, Fix, Verification, Prevention, and Related Components.

Clearly distinguish FACT from HYPOTHESIS. Only verified conclusions may be recorded as Root Cause.

## 15. Skills

Put reusable procedures in .agents/skills/. A Skill explains how to perform a class of work; the Wiki explains how this project works. Keep these purposes separate.

## 16. Verification

Code modification alone is not completion. Run the most relevant targeted checks first: Parser → parser tests, API → API tests, Workflow → integration tests, Types → typecheck, Build system → build. Select lint, typecheck, unit test, integration test, and build checks according to the project.

Never claim Passed for tests that were not run. If something was not verified, state Not Verified and explain why. When verification fails, distinguish failures caused by the current change, existing failures, environment failures, and external dependency failures. Do not hide failures to make results appear green.

## 17. Git Workflow

Before modifying code, at minimum inspect git status and git diff. Before committing, inspect git status, git diff, and git diff --stat; confirm there are no Secrets, debug garbage, unrelated changes, accidental generated files, or unknown user changes.

A commit should represent one logical unit. Prefer prefixes feat:, fix:, refactor:, docs:, test:, and chore:. For complex tasks, use Phase → Verify → Commit.

By default, commit != push. Push only according to user instruction, Workspace Policy, CI workflow, and Repository Policy.

## 18. Multi-Agent Collaboration

Divide work by module, tests, and documentation when possible. Avoid having multiple agents modify the same core file concurrently. In a shared Workspace, do not overwrite other agents' changes; inspect Git state, define the scope, and perform a unified Review before merging.

## 19. Tool Usage

Tool availability does not imply authorization. Docker being available does not authorize volume deletion; Git being available does not authorize Force Push; database access does not authorize a Database Reset.

## 20. Autonomous Execution

For ordinary development tasks, proceed autonomously: Investigate → Plan → Implement → Verify → Update Task → Update Knowledge if needed.

Stop and request explicit authorization for possible user-data deletion, Production Mutation, Credential Mutation, Force Push, Irreversible Migration, overwriting unknown user changes, or a high-risk operation with an unclear target. Make ordinary implementation choices based on Existing Architecture, Maintainability, Compatibility, Simplicity, and Testability.

## 21. Knowledge Update Protocol

Before completing complex work, perform a Knowledge Impact Review covering Architecture, Domain, Module Responsibility, Workflow, State Machine, Database, API Contract, Runtime, Migration, Feature Flags, and important Root Causes. Then decide whether TASK.md, CONTEXT.md, wiki/, or an ADR needs updating. Do not mechanically edit every document after every change.

## 22. Task Completion Checklist

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

## 23. Completion Report

Provide a concise completion report containing What Changed, Why, Main Files, Verification, Compatibility, Risks, Not Verified, and Next Step. If the task is incomplete, state the current status clearly. Do not claim everything is solved unless it was actually completed and verified.

## 24. Recommended Repository Structure

Use the following as needed: AGENTS.md, AGENTS_EN.md, CONTEXT.md, TASK.md, README.md, CONTRIBUTING.md, wiki/, tasks/, .agents/skills/, .github/, and src/. The project may trim this structure. Do not create meaningless directories for appearance only.

## 25. Standard Agent Session Flow

START → Read AGENTS.md / AGENTS_EN.md → Read CONTEXT.md → Read TASK.md → Read wiki/INDEX.md → Locate relevant knowledge → Inspect Git → Inspect relevant code → Establish current behavior → Identify root cause / design → Implement one coherent phase → Run targeted verification → Review Git diff → Update TASK.md → Review CONTEXT and Wiki impact → Commit when appropriate → Next phase / complete.

## 26. Final Principles

> **Understand first, modify second.**
>
> **Fix root causes; do not pile up patches.**
>
> **Keep module boundaries explicit.**
>
> **Migrate new capabilities incrementally; do not casually break the old pipeline.**
>
> **Data safety is more important than development speed.**
>
> **Do not overwrite unknown user changes.**
>
> **AI handles semantics; deterministic code handles rules and state.**
>
> **AGENTS defines rules, CONTEXT defines state, TASK defines current work, Wiki defines long-term knowledge, and Skills define procedures.**
>
> **Code, knowledge, tasks, and architecture decisions must be traceable together through Git.**
>
> **A new developer or coding agent should be able to recover project context from the repository alone.**

The goal is not to make the agent write as much code as possible, but to maintain a long-lived software project safely, accurately, continuously, and verifiably.
