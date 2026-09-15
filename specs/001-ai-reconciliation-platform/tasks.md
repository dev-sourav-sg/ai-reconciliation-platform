# Tasks: AI Reconciliation Platform

**Input**: Design documents from `/specs/001-ai-reconciliation-platform/`

**Prerequisites**: plan.md (required), spec.md (required for user stories), research.md, data-model.md, contracts/

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Establish the backend, AI, and deployment scaffolding required for the reconciliation feature.

- [ ] T001 [P] Initialize Java backend build and module structure in `backend/pom.xml`, `backend/src/main/java/`, and `backend/src/test/java/`
- [ ] T002 [P] Initialize Python AI service scaffolding in `ai/requirements.txt`, `ai/src/`, and `ai/tests/`
- [ ] T003 [P] Add Docker and deployment configuration in `docker-compose.yml`, `backend/Dockerfile`, `ai/Dockerfile`, and `k8s/`
- [ ] T004 Create shared configuration and environment templates in `backend/src/main/resources/application.yml`, `ai/.env.example`, and `backend/.env.example`

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Define the canonical business model, repository boundaries, and security/audit primitives before story implementation.

- [ ] T005 Create canonical transaction and lifecycle domain models in `backend/src/main/java/com/example/reconciliation/model/TransactionRecord.java` and `LifecycleEvent.java`
- [ ] T006 Create reconciliation outcome and mismatch reason models in `backend/src/main/java/com/example/reconciliation/model/ReconciliationOutcome.java` and `MismatchReason.java`
- [ ] T007 Create processor completeness snapshot and operational metric models in `backend/src/main/java/com/example/reconciliation/model/ProcessorDataSnapshot.java` and `OperationalMetric.java`
- [ ] T008 Implement repository interfaces and persistence adapters in `backend/src/main/java/com/example/reconciliation/repository/`
- [ ] T009 Implement shared authorization, audit, and correlation primitives in `backend/src/main/java/com/example/reconciliation/security/` and `backend/src/main/java/com/example/reconciliation/audit/`
- [ ] T010 Define the external API Gateway entry point and ingress policy in `gateway/`, `docker-compose.yml`, and `k8s/ingress.yaml`
- [ ] T011 Implement service-to-service authenticated identity and encrypted transport controls in `backend/src/main/java/com/example/reconciliation/security/ServiceIdentitySecurity.java`, `backend/src/main/java/com/example/reconciliation/config/SecurityConfig.java`, and `backend/src/main/resources/application.yml`
- [ ] T012 Configure LocalStack-based local cloud emulation and test integration wiring in `docker-compose.yml`, `localstack/`, and `backend/src/test/java/com/example/reconciliation/integration/LocalstackIntegrationTest.java`
- [ ] T013 Define NoSQL-oriented persistence design aligned with access patterns in `backend/src/main/java/com/example/reconciliation/config/PersistenceConfiguration.java` and `backend/src/main/java/com/example/reconciliation/repository/`
- [ ] T014 Define Kafka event-driven workflow foundations, including event contracts, idempotency, retry, and DLQ or equivalent failure handling in `backend/src/main/java/com/example/reconciliation/events/`, `backend/src/main/resources/kafka/`, and `backend/src/main/java/com/example/reconciliation/service/RetryPolicy.java`
- [ ] T015 Define Kubernetes deployment, readiness, health, and service communication requirements in `k8s/deployment.yaml`, `k8s/service.yaml`, and `k8s/healthcheck.yaml`
- [ ] T016 [P] Validate the OpenAPI contract and data model constraints against the approved requirements in `specs/001-ai-reconciliation-platform/contracts/reconciliation-api.yaml` and `specs/001-ai-reconciliation-platform/data-model.md`

## Phase 3: User Story 1 - Reconcile transactions to RECONCILED or UNRECONCILED status

**Story Goal**: Provide deterministic financial classification for every in-scope transaction with explicit mismatch reasons preserved in the output.

**Independent Test Criteria**: A reconciliation run for a defined period returns every transaction with either `RECONCILED` or `UNRECONCILED`, preserves material mismatches, and retains the transaction in the final output.

- [ ] T017 [US1] Implement the deterministic reconciliation engine in `backend/src/main/java/com/example/reconciliation/service/ReconciliationEngine.java`
- [ ] T018 [US1] Implement material mismatch detection for transaction ID, MID, amount, and action type in `backend/src/main/java/com/example/reconciliation/service/MismatchClassifier.java`
- [ ] T019 [US1] Implement score-based fallback logic and threshold evaluation in `backend/src/main/java/com/example/reconciliation/service/MatchingScoreEvaluator.java`
- [ ] T020 [US1] Add reconciliation run orchestration and status tracking in `backend/src/main/java/com/example/reconciliation/service/ReconciliationRunService.java`
- [ ] T021 [US1] Expose the async reconciliation-run API in `backend/src/main/java/com/example/reconciliation/api/ReconciliationController.java`
- [ ] T022 [US1] Add reconciliation outcome integration validation in `backend/src/test/java/com/example/reconciliation/ReconciliationEngineIT.java`

## Phase 4: User Story 2 - Investigate a transaction from a reconciliation analyst perspective

**Story Goal**: Surface a transaction-centric investigation view with evidence, differing fields, mismatch reasons, and lifecycle details.

**Independent Test Criteria**: An analyst can look up a transaction ID and review the reconciliation status, source details, matching explanation, mismatch reasons, and available lifecycle history.

- [ ] T023 [US2] Implement transaction detail assembly and evidence lookup in `backend/src/main/java/com/example/reconciliation/service/TransactionDetailService.java`
- [ ] T024 [US2] Implement lifecycle aggregation for reconciliation, funding, settlement, disputes, and chargebacks in `backend/src/main/java/com/example/reconciliation/service/LifecycleService.java`
- [ ] T025 [US2] Expose transaction detail and lifecycle endpoints in `backend/src/main/java/com/example/reconciliation/api/TransactionController.java`
- [ ] T026 [US2] Add analyst investigation validation in `backend/src/test/java/com/example/reconciliation/TransactionInvestigationIT.java`

## Phase 5: User Story 3 - Support Merchant Support and merchant self-service lookup

**Story Goal**: Enforce least-privilege access for Merchant Support and merchant self-service transaction visibility.

**Independent Test Criteria**: Authorized users can look up only their permitted transactions and a denied access request is recorded and rejected.

- [ ] T027 [US3] Implement merchant-scoped authorization logic in `backend/src/main/java/com/example/reconciliation/security/MerchantAccessPolicy.java`
- [ ] T028 [US3] Implement merchant transaction lookup and lifecycle access in `backend/src/main/java/com/example/reconciliation/service/MerchantLookupService.java`
- [ ] T029 [US3] Expose merchant and support access endpoints in `backend/src/main/java/com/example/reconciliation/api/MerchantAccessController.java`
- [ ] T030 [US3] Add authorization and denial-path tests in `backend/src/test/java/com/example/reconciliation/MerchantAccessIT.java`

## Phase 6: User Story 4 - Monitor processor-data completeness and business performance

**Story Goal**: Detect completeness gaps and surface business metrics by processor and reporting period.

**Independent Test Criteria**: The system can compare expected vs received populations, raise an alert for completeness gaps, and show reconciled vs unreconciled metrics and SLA status.

- [ ] T031 [US4] Implement processor completeness comparison logic in `backend/src/main/java/com/example/reconciliation/service/ProcessorCompletenessService.java`
- [ ] T032 [US4] Implement operational metric aggregation in `backend/src/main/java/com/example/reconciliation/service/ReconciliationMetricsService.java`
- [ ] T033 [US4] Expose completeness and metrics endpoints in `backend/src/main/java/com/example/reconciliation/api/OperationsController.java`
- [ ] T034 [US4] Add completeness and metrics validation in `backend/src/test/java/com/example/reconciliation/OperationsMetricsIT.java`

## Phase 7: User Story 5 - Manage exceptions and audit investigation history

**Story Goal**: Keep unreconciled investigations traceable with evidence, notes, and reviewer decisions.

**Independent Test Criteria**: Reviewers can open a case, record actions, and preserve the full audit history and disposition path.

- [ ] T035 [US5] Create investigation-case and audit-history domain models in `backend/src/main/java/com/example/reconciliation/model/InvestigationCase.java` and `AuditEntry.java`
- [ ] T036 [US5] Implement case creation, action recording, and disposition logic in `backend/src/main/java/com/example/reconciliation/service/CaseManagementService.java`
- [ ] T037 [US5] Expose case list, case detail, and case-action endpoints in `backend/src/main/java/com/example/reconciliation/api/CaseController.java`
- [ ] T038 [US5] Add case audit trail validation in `backend/src/test/java/com/example/reconciliation/CaseManagementIT.java`

## Phase 8: User Story 6 - Use governed AI read-only assistance

**Story Goal**: Allow natural-language investigation support while preventing AI from becoming the authority for financial truth.

**Independent Test Criteria**: The AI converts a request into a constrained structured search intent, enforces permission checks, and rejects unrestricted query behavior.

- [ ] T039 [US6] Define AI search intent schema and bounded transformation in `ai/src/reconciliation_ai/search/StructuredSearchMapper.py`
- [ ] T040 [US6] Implement read-only governed-search logic with authorization checks in `ai/src/reconciliation_ai/search/GovernedSearchService.py`
- [ ] T041 [US6] Expose the `/ai/search` API surface and integrate it with the backend in `backend/src/main/java/com/example/reconciliation/api/AiSearchController.java`
- [ ] T042 [US6] Add AI evaluation and injection-rejection tests in `ai/tests/test_governed_search.py`

## Phase 9: Polish & Cross-Cutting Concerns

**Purpose**: Finalize resilience, observability, and deployment readiness before validation.

- [ ] T043 [P] Add request correlation, structured logging, metrics, and health checks across `backend/src/main/java/com/example/reconciliation/monitoring/` and `ai/src/reconciliation_ai/telemetry/`
- [ ] T044 [P] Add event-driven resilience hardening and release validation for asynchronous workflow recovery in `backend/src/main/resources/kafka/`, `backend/src/main/java/com/example/reconciliation/events/`, and `specs/001-ai-reconciliation-platform/quickstart.md`

## Dependencies

- User Story 1 must be complete before User Stories 2, 3, 4, 5, and 6 can be fully validated.
- User Story 2 depends on the reconciliation outcome model and transaction record model created in Phase 2.
- User Story 3 depends on the authorization primitives created in Phase 2 and the transaction view created in User Story 2.
- User Story 4 depends on the completeness comparison model and metrics layer created in Phase 2 and the reconciliation-result outputs from User Story 1.
- User Story 5 depends on the transaction outcome, case model, and audit primitives created in Phase 2.
- User Story 6 depends on the investigation evidence and authorization model established in User Stories 2 and 3.

## Parallel Execution Examples

- Setup tasks `T001` through `T004` can be completed in parallel because they touch distinct scaffolding areas.
- Model creation tasks `T005` through `T010` can be completed in parallel after the architecture is approved.
- User Story phases 3 through 8 are independent once the foundational models and authorization layer are in place; implementation teams can work on multiple stories concurrently when their APIs and data contracts are agreed.

## Suggested MVP Scope

The recommended MVP delivery scope is User Story 1 only, with foundational security and audit primitives in place. This gives the project a demonstrable end-to-end deterministic reconciliation capability before expanding to merchant access, completeness monitoring, case handling, and AI-supported investigation.

## Validation Checklist

- [ ] All tasks use the required checklist format: `- [ ] T### ...`
- [ ] Each task includes a concrete file path and a clear action.
- [ ] User story tasks are grouped by `US1` through `US6` in priority order.
- [ ] Setup and foundation tasks remain outside story labels.
- [ ] Final polish tasks are grouped under a cross-cutting phase.
