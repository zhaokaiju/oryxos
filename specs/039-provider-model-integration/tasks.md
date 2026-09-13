---
description: "Task list for Provider model integration"
---

# Tasks: Provider 模型统一接入

**Input**: Design documents from `/specs/039-provider-model-integration/`

## Phase 1: Setup

- [ ] T001 Configure Provider module dependencies and shared Java 21 settings in `pom.xml`
- [ ] T002 [P] Add provider environment-variable placeholders and safe defaults in `config/application.yml.example`
- [ ] T003 [P] Add Flyway migration for provider configuration fields and indexes in `oryxos-storage/src/main/resources/db/migration/V_provider.sql`

## Phase 2: Foundational

- [ ] T004 Create shared model request, response, call ID and error types in `oryxos-core/src/main/java/**/model`
- [ ] T005 [P] Implement credential reference resolver and redaction utility in `oryxos-core/src/main/java/**/security/CredentialResolver.java`
- [ ] T006 [P] Implement provider repository transaction boundaries in `oryxos-storage/src/main/java/**/ProviderRepository.java`
- [ ] T007 Configure synchronous provider call execution on virtual threads in `oryxos-boot/src/main/java/**/ProviderExecutionConfig.java`

## Phase 3: User Story 1 - 按名称选择模型供应商 (Priority: P1)

**Goal**: Multiple enabled providers can be selected deterministically by name and return a uniform response.

**Independent Test**: Configure two providers, call each by name, and verify no cross-provider routing and stable errors.

- [ ] T008 [P] [US1] Define Provider entity validation for unique safe names and connection fields in `oryxos-provider/src/main/java/**/ProviderConfig.java`
- [ ] T009 [P] [US1] Define adapter contract for protocol-to-uniform request/response conversion in `oryxos-provider/src/main/java/**/ProviderAdapter.java`
- [ ] T010 [US1] Implement explicit provider-name registry and refresh operation in `oryxos-provider/src/main/java/**/ProviderRegistry.java`
- [ ] T011 [US1] Implement ProviderService routing, call ID generation and uniform response mapping in `oryxos-provider/src/main/java/**/ProviderService.java`
- [ ] T012 [US1] Add provider-specific adapters for supported model protocols in `oryxos-provider/src/main/java/**/adapters`
- [ ] T013 [US1] Ensure model responses containing tool calls are described but never auto-executed in `oryxos-provider/src/main/java/**/ProviderService.java`
- [ ] T014 [P] [US1] Add routing, unknown-provider, disabled-provider and uniform-response tests in `oryxos-provider/src/test/java/**/ProviderRoutingTest.java`
- [ ] T015 [P] [US1] Add adapter conversion tests for success, timeout, rate-limit and protocol errors in `oryxos-provider/src/test/java/**/ProviderAdapterTest.java`

## Phase 4: User Story 2 - 安全配置与运行时维护 (Priority: P2)

**Goal**: Operators can maintain provider configurations safely and changes affect subsequent calls.

**Independent Test**: Create, update, enable, disable and remove providers while checking redaction and refresh behavior.

- [ ] T016 [P] [US2] Implement provider CRUD service with name, URL and credential-reference validation in `oryxos-provider/src/main/java/**/ProviderAdminService.java`
- [ ] T017 [US2] Implement enable, disable, connection-test and safe-remove rules in `oryxos-provider/src/main/java/**/ProviderAdminService.java`
- [ ] T018 [US2] Refresh registry atomically after provider mutations in `oryxos-provider/src/main/java/**/ProviderRegistry.java`
- [ ] T019 [P] [US2] Expose provider CRUD and lifecycle endpoints from `oryxos-web/src/main/java/**/ProviderController.java`
- [ ] T020 [P] [US2] Expose provider list, set, enable, disable and test commands in `oryxos-cli/src/main/java/**/ProviderCommand.java`
- [ ] T021 [US2] Add provider request validation and redacted response mapping in `oryxos-web/src/main/java/**/ProviderDtoMapper.java`
- [ ] T022 [US2] Add administration, concurrent-update and credential-redaction tests in `oryxos-provider/src/test/java/**/ProviderAdminServiceTest.java`
- [ ] T023 [US2] Write model-call audit events with call ID and redaction hooks in `oryxos-storage/src/main/java/**/ModelAuditWriter.java`

## Phase 5: Polish & Cross-Cutting Concerns

- [ ] T024 [P] Add provider routing, failure and configuration metrics in `oryxos-provider/src/main/java/**/ProviderMetrics.java`
- [ ] T025 [P] Document provider setup, credential references and switching in `specs/039-provider-model-integration/quickstart.md`
- [ ] T026 Run provider unit/integration tests and `mvn verify`; fix findings in affected module files
- [ ] T027 Validate all acceptance scenarios and record evidence in `specs/039-provider-model-integration/checklists/requirements.md`

## Dependencies & Execution Order

- Setup T001-T003 precedes Foundational T004-T007.
- US1 T008-T015 depends on Foundational completion.
- US2 T016-T023 depends on the ProviderService and registry from US1.
- Polish T024-T027 follows both user stories.

## Parallel Opportunities

- T002-T003 and T005-T006 can run in parallel.
- US1 T008-T009 and T014-T015 can run in parallel where files do not overlap.
- US2 T019-T020 and T022 can run in parallel after service contracts are stable.

## Implementation Strategy

### MVP First

Complete Setup, Foundational and US1; validate deterministic two-provider routing and uniform responses.

### Incremental Delivery

Add US2 lifecycle management and secure credential handling, then metrics, documentation and full verification.
