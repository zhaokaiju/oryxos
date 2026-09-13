---
description: "Executable task list for OryxOS Agent OS core runtime"
---

# Tasks: OryxOS Agent OS 核心运行时

**Input**: Design documents from `/specs/029-oryxos-core/`

**Organization**: Tasks are grouped by the five user stories in `spec.md`.

## Phase 1: Setup

- [ ] T001 Create the nine Maven module declarations in `pom.xml`
- [ ] T002 [P] Configure Java 21, Spring Boot and shared dependency versions in `pom.xml`
- [ ] T003 [P] Add baseline application configuration and environment placeholders in `config/application.yml.example`
- [ ] T004 [P] Configure Spotless, Checkstyle, SpotBugs and dependency checks in `pom.xml`

## Phase 2: Foundational

- [ ] T005 Create core abstractions for messages, profiles, tools and results in `oryxos-core/src/main/java/**/core`
- [ ] T006 Create Flyway migration baseline for sessions, memories, providers and audit tables in `oryxos-storage/src/main/resources/db/migration`
- [ ] T007 [P] Implement shared error model, trace identifiers and structured logging in `oryxos-core/src/main/java/**/error`
- [ ] T008 [P] Implement environment and secret reference validation in `oryxos-cli/src/main/java/**/config`
- [ ] T009 [P] Configure virtual-thread task execution and synchronous request boundaries in `oryxos-boot/src/main/java/**/config`
- [ ] T010 Implement persistence repositories and transaction boundaries for core entities in `oryxos-storage/src/main/java/**/repository`

**Checkpoint**: Shared contracts, migrations, error handling and persistence are ready before story work.

## Phase 3: User Story 1 - 统一接入大语言模型 (Priority: P1)

**Goal**: Route requests to named providers through a uniform model interface.

**Independent Test**: Configure two providers, select each by name, and verify uniform responses and safe credential errors.

- [ ] T011 [P] [US1] Implement provider configuration entity and validation in `oryxos-provider/src/main/java/**/ProviderConfig.java`
- [ ] T012 [P] [US1] Implement explicit provider-name to ChatModel registry in `oryxos-provider/src/main/java/**/ProviderRegistry.java`
- [ ] T013 [US1] Implement uniform model request and response adapter in `oryxos-provider/src/main/java/**/ProviderService.java`
- [ ] T014 [US1] Disable automatic model/tool execution and eager provider auto-configuration in `oryxos-boot/src/main/resources/application.yml`
- [ ] T015 [US1] Persist provider CRUD operations through `oryxos-storage/src/main/java/**/ProviderRepository.java`
- [ ] T016 [US1] Add provider routing and missing-credential validation tests in `oryxos-provider/src/test/java/**/ProviderServiceTest.java`

## Phase 4: User Story 2 - 可控的 ReAct 工具循环 (Priority: P1)

**Goal**: Execute a bounded, observable think-act-observe loop controlled by OryxOS.

**Independent Test**: Run a weather-style task requiring one tool call and verify tool result injection, termination and iteration limits.

- [ ] T017 [P] [US2] Implement prompt assembly for instructions, memory, history and tool metadata in `oryxos-core/src/main/java/**/PromptBuilder.java`
- [ ] T018 [P] [US2] Implement tool registry and schema exposure in `oryxos-tool/src/main/java/**/ToolRegistry.java`
- [ ] T019 [US2] Implement synchronous tool dispatch and retry classification in `oryxos-core/src/main/java/**/ToolExecutor.java`
- [ ] T020 [US2] Implement bounded ReAct loop with explicit final-answer and failure termination in `oryxos-core/src/main/java/**/ReActLoop.java`
- [ ] T021 [US2] Implement session lifecycle and message accumulation in `oryxos-core/src/main/java/**/SessionManager.java`
- [ ] T022 [US2] Implement CLI chat command and workspace initialization in `oryxos-cli/src/main/java/**/ChatCommand.java`
- [ ] T023 [US2] Add ReAct loop integration validation in `oryxos-core/src/test/java/**/ReActLoopTest.java`

## Phase 5: User Story 3 - 跨会话记忆与 Skill 能力复用 (Priority: P2)

**Goal**: Persist useful memory and expose safely bound Skill metadata with on-demand content loading.

**Independent Test**: Save a preference, recall it in a new session, and validate a good and bad Skill link.

- [ ] T024 [P] [US3] Implement memory entity, repository and save/recall service in `oryxos-memory/src/main/java/**/MemoryService.java`
- [ ] T025 [P] [US3] Implement Agent directory and AGENT.md loader in `oryxos-core/src/main/java/**/AgentLoader.java`
- [ ] T026 [US3] Implement relative Skill link scanner, metadata parser and stable ordering in `oryxos-core/src/main/java/**/ContextLoader.java`
- [ ] T027 [US3] Inject memory and Skill metadata into prompt context in `oryxos-core/src/main/java/**/PromptBuilder.java`
- [ ] T028 [US3] Add memory recall and Skill-link validation tests in `oryxos-memory/src/test/java/**/MemoryServiceTest.java`

## Phase 6: User Story 4 - 安全工具与审计 (Priority: P2)

**Goal**: Enforce sandbox policy and record every model/tool outcome durably.

**Independent Test**: Exercise allowed and denied file, command, domain and symlink requests and query their audit records.

- [ ] T029 [P] [US4] Implement file, command and domain policy evaluation in `oryxos-tool/src/main/java/**/SandboxChecker.java`
- [ ] T030 [P] [US4] Implement real-path validation for existing and newly created symlink targets in `oryxos-tool/src/main/java/**/PathPolicy.java`
- [ ] T031 [US4] Implement built-in file, shell and HTTP tools with pre-execution sandbox enforcement in `oryxos-tool/src/main/java/**/builtin`
- [ ] T032 [US4] Implement LLM and tool audit writers with redaction in `oryxos-storage/src/main/java/**/AuditService.java`
- [ ] T033 [US4] Integrate audit writes into provider calls and tool execution in `oryxos-core/src/main/java/**/AuditInterceptor.java`
- [ ] T034 [US4] Add sandbox escape and audit durability tests in `oryxos-tool/src/test/java/**/SandboxCheckerTest.java`

## Phase 7: User Story 5 - Web、CLI 与多端点管理 (Priority: P3)

**Goal**: Expose shared core capabilities through REST and management commands.

**Independent Test**: Initialize, create an Agent, chat synchronously, inspect health/session/audit data through CLI and REST.

- [ ] T035 [P] [US5] Implement health, Agent and chat REST endpoints in `oryxos-web/src/main/java/**/AgentController.java`
- [ ] T036 [P] [US5] Implement provider, session, audit and schedule REST endpoints in `oryxos-web/src/main/java/**/ManagementController.java`
- [ ] T037 [US5] Implement uniform REST error responses and trace propagation in `oryxos-web/src/main/java/**/GlobalExceptionHandler.java`
- [ ] T038 [US5] Implement CLI Agent, provider and health subcommands in `oryxos-cli/src/main/java/**/commands`
- [ ] T039 [US5] Serve the Web management assets and configure API base paths in `oryxos-web/src/main/resources/static/admin`
- [ ] T040 [US5] Add REST and CLI end-to-end checks from `contracts/` and `quickstart.md` in `oryxos-web/src/test/java/**/CoreApiE2ETest.java`

## Phase 8: Polish & Cross-Cutting Concerns

- [ ] T041 [P] Document deployment, environment variables and recovery in `docs/quickstart.md`
- [ ] T042 [P] Add structured metrics and health indicators for model, tool, session and audit operations in `oryxos-core/src/main/java/**/observability`
- [ ] T043 Run `mvn verify` and fix formatting, static analysis and dependency findings in affected module files
- [ ] T044 Validate all scenarios in `specs/029-oryxos-core/quickstart.md` and record results in `specs/029-oryxos-core/checklists/requirements.md`

## Dependencies & Execution Order

- Setup (T001-T004) precedes Foundational (T005-T010).
- US1 (T011-T016) precedes US2 (T017-T023).
- US3 (T024-T028) and US4 (T029-T034) both depend on US2 and can proceed in parallel.
- US5 (T035-T040) depends on US1-US4.
- Polish (T041-T044) follows all selected stories.

## Parallel Opportunities

- Setup T002-T004 and foundational T007-T009 can run in parallel.
- US1 T011-T012; US2 T017-T018; US3 T024-T025; US4 T029-T030; and US5 T035-T036 are parallel groups when their prerequisites are complete.
- US3 and US4 are the largest independent story-level parallel tracks.

## Implementation Strategy

### MVP First

Complete Setup, Foundational, US1 and US2, then validate the weather-style ReAct scenario.

### Incremental Delivery

Add US3 for memory and Skill reuse, US4 for sandbox and audit, then US5 for Web and management operations.

### Definition of Done

Every completed story passes its independent test, all tool/model paths are auditable, and `mvn verify` is green.
