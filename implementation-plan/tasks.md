# BackOfficePilot — Task Breakdown

Each task below maps to a story. Tasks are ordered by dependency. For each task, the developer should follow the SPEC → BDD → TDD → CODE workflow.

---

## Task 1: STORY-A3-001 — Bootstrap Repo & Dev Environment

**Priority:** P0 (blocks everything)
**Estimated effort:** 1 day
**Dependencies:** None

### What to build
- Initialize monorepo with Python backend (`src/`) and Next.js frontend (`web/`)
- Set up `uv` for Python package management (Python 3.12)
- Set up `pnpm` for frontend package management
- Configure GitHub Actions CI pipeline (lint, type-check, test)
- Configure pre-commit hooks (ruff, mypy, prettier)
- Create `Makefile` with common commands (`make test`, `make lint`, `make traceability`)
- Set up pytest + pytest-bdd + hypothesis in `tests/`
- Create `.github/workflows/ci.yml`

### Acceptance criteria
- `make test` runs (even if no tests yet)
- `make lint` passes on empty project
- CI pipeline triggers on push and PR
- Directory structure matches the spec layout

### Key files to create
```
pyproject.toml, uv.lock
web/package.json, web/tsconfig.json
Makefile
.github/workflows/ci.yml
.pre-commit-config.yaml
src/__init__.py, src/domain/__init__.py, src/application/__init__.py
src/adapters/__init__.py, src/api/__init__.py
tests/conftest.py
```

---

## Task 2: STORY-A3-002 — CDK UAT Infrastructure Stacks

**Priority:** P0 (blocks deployment)
**Estimated effort:** 2 days
**Dependencies:** Task 1

### What to build
- CDK app in `infra/` with Python
- **Network stack:** VPC (public + private subnets), NAT Gateway, security groups
- **Data stack:** DynamoDB single-table, S3 buckets (runs, audit with Object Lock, policy bundles), OpenSearch domain
- **Security stack:** KMS keys, Secrets Manager paths, IAM role templates (`PerCustomerIamRole` construct)
- **AgentCore stack:** AgentCore Runtime, Gateway, Memory provisioning
- **Scheduling stack:** EventBridge Scheduler rules
- **Auth stack:** Cognito User Pool + app client

### Acceptance criteria
- `cdk synth` produces valid CloudFormation
- `cdk deploy --all` provisions UAT environment
- DDB table created with correct key schema
- S3 audit bucket has Object Lock enabled in compliance mode
- IAM roles scoped per-customer prefix

### Key files to create
```
infra/app.py
infra/stacks/network_stack.py
infra/stacks/data_stack.py
infra/stacks/security_stack.py
infra/stacks/agentcore_stack.py
infra/stacks/auth_stack.py
infra/constructs/per_customer_iam_role.py
```

---

## Task 3: STORY-A3-003 — FastAPI Control Plane + OpenAPI Codegen

**Priority:** P0
**Estimated effort:** 2 days
**Dependencies:** Task 1, Task 2

### What to build
- FastAPI application with Mangum handler (Lambda adapter)
- OpenAPI 3.1 spec at `specs/30-architecture/34-api-contract/openapi.yaml`
- API routes:
  - `GET /customers/{id}/workflows` — list workflows
  - `GET /customers/{id}/runs` — list runs (paginated, filterable)
  - `GET /runs/{id}` — run detail
  - `GET /runs/{id}/steps` — list steps
  - `GET /runs/{id}/audit` — audit events
  - `GET /runs/{id}/cost` — cost breakdown
  - `POST /customers/{id}/workflows/{wf}/runs` — trigger manual run
  - `POST /runs/{id}/abort` — abort run
  - `PUT /customers/{id}/workflows/{wf}` — upsert workflow
  - `GET /customers/{id}/escalations` — open escalations
  - `POST /escalations/{id}/resolve` — resolve escalation
- TypeScript client generation from OpenAPI spec → `web/lib/_generated/`
- JWT validation middleware (Cognito issuer, audience, expiry)
- Tenant context middleware (extract `customer_id` from JWT, enforce on all queries)
- Problem response format (RFC 9457) for errors

### Acceptance criteria
- OpenAPI spec validates
- TypeScript client generates without errors
- JWT middleware rejects invalid/expired tokens with 401
- Cross-tenant access returns 403
- All endpoints return proper Problem responses on error

### Key files to create
```
src/api/main.py (FastAPI app)
src/api/middleware/auth.py
src/api/middleware/tenant_context.py
src/api/routers/runs.py
src/api/routers/workflows.py
src/api/routers/escalations.py
specs/30-architecture/34-api-contract/openapi.yaml
scripts/generate_client.sh
```

---

## Task 4: STORY-A3-004 — Next.js Dashboard Scaffold + Cognito Auth

**Priority:** P0
**Estimated effort:** 2 days
**Dependencies:** Task 1, Task 3

### What to build
- Next.js 14 App Router project in `web/`
- Tailwind CSS + shadcn/ui setup with brand palette (navy, ice, gold, teal, coral)
- Cognito authentication flow (hosted UI → JWT cookie)
- Next.js middleware for auth validation
- App Router layout structure:
  - `(auth)/login` — login page
  - `(customer)/` — customer-scoped pages (run list, escalations, cost, workflows)
  - `(operator)/` — operator pages
- TanStack Query provider setup
- SSE utility hook for live updates
- Skeleton/loading states (never spinner-only)

### Acceptance criteria
- Login flow works with Cognito
- Authenticated routes redirect to login if no valid session
- Customer routes enforce customer scope
- Brand palette applied consistently
- Lighthouse accessibility score ≥ 90

### Key files to create
```
web/app/layout.tsx
web/app/(auth)/login/page.tsx
web/app/(customer)/layout.tsx
web/app/(customer)/page.tsx (dashboard home)
web/middleware.ts
web/lib/auth.ts
web/lib/sse.ts
web/tailwind.config.ts (brand colors)
web/components/ui/ (shadcn/ui components)
```

---

## Task 5: STORY-A3-005 — Domain Models + Property-Based Tests

**Priority:** P0
**Estimated effort:** 1.5 days
**Dependencies:** Task 1

### What to build
- Pydantic v2 models in `src/domain/`:
  - `Workflow` — parsed from YAML, validates structure
  - `Plan` — Reasoner output: steps, success criteria, reasoning trace
  - `Step` — unit of executor work, status transitions
  - `Run` — execution lifecycle, status machine
  - `Exception` — classification result
  - `Escalation` — routing + SLA tracking
  - `AuditEvent` — hash-chained record
  - `CostMeter` — per-run cost breakdown
- Port interfaces (ABCs) in `src/domain/ports.py`:
  - `ReasonerPort`, `ExecutorPort`, `MemoryPort`
  - `EventBusPort`, `AuditLogPort`, `SecretPort`
- Domain events: `PlanCreated`, `StepExecuted`, `ExceptionRaised`, `RunCompleted`, `RunAborted`, `CostMetered`, `EscalationOpened`, `EscalationResolved`, `SlaBreached`
- Hypothesis property-based tests for all models

### Acceptance criteria
- All models validate correctly (reject invalid data)
- Status transitions are enforced (e.g., cannot go from `succeeded` to `running`)
- Hypothesis finds no invariant violations over 1000+ examples
- Domain layer has ZERO AWS imports
- Port interfaces are abstract (no implementation)

### Key files to create
```
src/domain/workflow.py
src/domain/plan.py
src/domain/step.py
src/domain/run.py
src/domain/exception_classifier.py
src/domain/escalation.py
src/domain/audit_event.py
src/domain/cost_meter.py
src/domain/events.py
src/domain/ports.py
tests/domain/test_models.py (hypothesis)
tests/domain/test_status_transitions.py
```

---

## Task 6: STORY-A3-006 — Workflow YAML Validator

**Priority:** P1
**Estimated effort:** 1.5 days
**Dependencies:** Task 5

### What to build
- JSON Schema at `specs/30-architecture/34-api-contract/schemas/workflow.schema.json`
- Pydantic `WorkflowDefinition` model that doubles as runtime parser
- Validation pipeline:
  1. YAML parse (PyYAML safe loader)
  2. JSON Schema validate
  3. Pydantic model construction
  4. Cross-field validation:
     - Every `step.system` references a declared `systems[].id`
     - Every `inputs:` references a prior `step.id`
     - Every `policy_documents[]` exists in customer's policy bundle
     - `budget.max_cost_per_run_usd` ≤ customer plan ceiling
- Structured error list response (not single message)
- Semver bump computation from diff against prior version

### Acceptance criteria
- Invalid YAML returns 400 with all validation errors listed
- Valid YAML produces a `WorkflowDefinition` model
- Cross-field references validated (dangling system/step refs rejected)
- Version bump computed correctly (patch/minor/major)

### Key files to create
```
src/domain/workflow_validator.py
specs/30-architecture/34-api-contract/schemas/workflow.schema.json
tests/domain/test_workflow_validator.py
features/BDD-A3-006.feature
```

---

## Task 7: STORY-A3-007 — Bedrock Opus Reasoner Adapter

**Priority:** P1
**Estimated effort:** 2 days
**Dependencies:** Task 5, Task 2

### What to build
- `BedrockOpusReasoner` implementing `ReasonerPort`
- Three capabilities:
  1. **Plan** — Given workflow + policy bundle, produce a step-by-step plan
  2. **Reconcile** — Given step result + policy, determine match/exception/retry
  3. **Classify** — Given ambiguous result, classify exception category + severity
- Prompt engineering:
  - Policy bundle loaded as cached prompt prefix
  - Structured output (JSON mode) for plan and reconcile responses
- Token usage tracking (input/output tokens reported for cost meter)
- Error handling: retry with backoff on throttling, circuit breaker on persistent failure

### Acceptance criteria
- Plan generation produces valid `Plan` model from workflow definition
- Reconcile returns confidence score + decision (match/exception/retry)
- Exception classification maps to registered categories
- Token counts reported accurately for cost metering
- Throttling handled gracefully (exponential backoff)

### Key files to create
```
src/adapters/bedrock/opus_reasoner.py
src/adapters/bedrock/prompts/ (plan.txt, reconcile.txt, classify.txt)
tests/adapters/bedrock/test_opus_reasoner.py
features/BDD-A3-007.feature
```

---

## Task 8: STORY-A3-008 — Nova Act Executor Adapter

**Priority:** P1
**Estimated effort:** 2 days
**Dependencies:** Task 5, Task 2

### What to build
- `NovaActExecutor` implementing `ExecutorPort`
- Capabilities:
  - Navigate to URL, authenticate with stored credentials
  - Execute intent-based actions (click, type, download, upload)
  - Capture screenshots at each action
  - Report observations (what was seen on screen)
  - Handle timeouts and element-not-found errors
- Credential retrieval from Secrets Manager (via `SecretPort`)
- Step result packaging: observations, actions taken, screenshots, duration

### Acceptance criteria
- Executor can navigate to a URL and perform actions
- Credentials retrieved securely from Secrets Manager
- Screenshots captured and attached to step result
- Timeout handling: step fails gracefully after configured timeout
- Element-not-found: reported as step failure (triggers reconcile retry)

### Key files to create
```
src/adapters/novaact/executor.py
src/adapters/novaact/action_mapper.py
tests/adapters/novaact/test_executor.py
features/BDD-A3-008.feature
```

---

## Task 9: STORY-A3-009 — AgentCore Browser Executor (Fallback)

**Priority:** P1
**Estimated effort:** 1.5 days
**Dependencies:** Task 5, Task 2

### What to build
- `AgentCoreBrowserExecutor` implementing `ExecutorPort`
- Same interface as Nova Act but uses AgentCore Browser (sandboxed Playwright)
- Used when:
  - Nova Act cannot reach a system (network/firewall)
  - Nova Act fails repeatedly on a specific step
  - Dev/test environments
- Fallback routing logic in `ExecutorRouter`

### Acceptance criteria
- Same `ExecutorPort` interface as Nova Act
- ExecutorRouter falls back to AgentCore Browser after Nova Act failure
- Browser sessions properly started and stopped
- Screenshots captured identically to Nova Act adapter

### Key files to create
```
src/adapters/agentcore/browser_executor.py
src/application/executor_router.py
tests/adapters/agentcore/test_browser_executor.py
features/BDD-A3-009.feature
```

---

## Task 10: STORY-A3-010 — PERC Loop Orchestrator + Cost Ceiling

**Priority:** P0 (core logic)
**Estimated effort:** 3 days
**Dependencies:** Task 5, Task 7, Task 8, Task 9

### What to build
- `PERCLoop` class in `src/application/perc_loop.py`
- State machine: Plan → Execute → Reconcile → Commit
- Between each step:
  - Check abort signal (DDB attribute poll)
  - Check cost ceiling (compare running tally to budget)
  - Emit domain events (`StepExecuted`, `ExceptionRaised`, etc.)
- Retry logic: on step failure, reconcile decides retry/escalate/abort
  - Max retries configurable (default 2), exponential backoff
  - Repeated exception → abort with `reason=repeated_exception`
- `CostCeilingEnforcer` in domain layer:
  - Soft ceiling: configured `max_cost_per_run_usd`
  - Hard ceiling: 2× configured (kill switch)
- `RunWorkflowUseCase` orchestrates the full lifecycle
- Property-based tests with hypothesis (arbitrary step/result sequences)

### Acceptance criteria
- 5-step workflow executes in order, emits `RunCompleted`
- Step failure triggers retry (up to max), then escalation
- Cost ceiling breach → `RunAborted` with `reason=cost_ceiling_exceeded`
- Abort signal honored between steps (no new steps dispatched)
- Hypothesis finds no invariant violations (no `RunCompleted` after `RunAborted`)
- Integration test runs full workflow against mock adapters

### Key files to create
```
src/application/perc_loop.py
src/application/run_workflow_use_case.py
src/domain/cost_ceiling.py
tests/application/test_perc_loop.py (hypothesis)
tests/application/test_run_workflow_use_case.py
features/BDD-A3-010.feature
```

---

## Task 11: STORY-A3-011 — S3 Hash-Chain Audit Writer

**Priority:** P1
**Estimated effort:** 1.5 days
**Dependencies:** Task 5, Task 2

### What to build
- `S3HashChainAuditLog` implementing `AuditLogPort`
- Algorithm:
  - Canonicalize event (sorted keys, UTF-8, no whitespace, ISO-8601)
  - Compute `this_hash = SHA-256(prev_hash + "\n" + payload)`
  - Write to DDB (PK: `RUN#{run_id}`, SK: `AUDIT#{sequence:08d}`)
  - Write to S3 with Object Lock retention
- On `RunCompleted`/`RunAborted`: package full stream as `audit.jsonl`
- Sequence counter: DDB atomic increment per run
- Verification script: `scripts/verify_audit.py`
- Async write pattern: orchestrator does not block on audit write

### Acceptance criteria
- Every domain event produces an audit record
- Hash chain is verifiable (recompute matches stored hashes)
- S3 objects have Object Lock retention applied
- Out-of-order events detected and alerted
- `scripts/verify_audit.py` independently verifies any run's audit chain

### Key files to create
```
src/adapters/audit/s3_hash_chain.py
scripts/verify_audit.py
tests/adapters/audit/test_s3_hash_chain.py
features/BDD-A3-011.feature
```

---

## Task 12: STORY-A3-012 — Cost Meter Subscriber

**Priority:** P1
**Estimated effort:** 1.5 days
**Dependencies:** Task 5, Task 10

### What to build
- `CostMeterSubscriber` — event bus subscriber
- `config/pricing.yaml` — pricing source-of-truth (versioned)
- Per-run tally:
  - Compute `cost_delta` from event fields × pricing
  - Emit `CostMetered` event every 30s or every $0.10
  - Write incremental update to DDB `CostMeter` row
- `RollupAggregator` — background process for daily totals
- On `RunCompleted`/`RunAborted`: write final tally to S3 run bundle
- Record pricing version used per event (price changes don't alter history)

### Acceptance criteria
- Cost computed correctly from token counts and runtime duration
- `CostMetered` events emitted at configured intervals
- Final cost matches sum of all deltas
- Pricing version recorded per event
- Daily rollups computed for dashboard

### Key files to create
```
src/adapters/cost/cost_meter_subscriber.py
src/adapters/cost/rollup_aggregator.py
config/pricing.yaml
tests/adapters/cost/test_cost_meter.py
features/BDD-A3-012.feature
```

---

## Task 13: STORY-A3-013 — Slack Escalation Adapter

**Priority:** P1
**Estimated effort:** 1 day
**Dependencies:** Task 5, Task 10

### What to build
- `SlackEscalationAdapter` — sends escalation messages to Slack
- Message contents:
  - Workflow + run link to dashboard
  - Reasoner's narrative summary (one paragraph)
  - Step context (key fields)
  - Up to 3 screenshots
  - Buttons: Approve / Reject / Open in Dashboard
- Button interactions hit `resolveEscalation` API
- SLA timer integration (DDB TTL + Step Functions wait for breach notification)

### Acceptance criteria
- Escalation message posted to configured Slack channel
- Message includes narrative, context, and action buttons
- Approve/Reject buttons resolve the escalation via API
- SLA breach triggers follow-up notification

### Key files to create
```
src/adapters/slack/escalation_adapter.py
tests/adapters/slack/test_escalation_adapter.py
features/BDD-A3-013.feature
```

---

## Task 14: STORY-A3-014 — Dashboard Run List + Run Detail

**Priority:** P1
**Estimated effort:** 2 days
**Dependencies:** Task 4, Task 3

### What to build
- **Run list page** (`web/app/(customer)/page.tsx`):
  - Paginated list of 50 most recent runs
  - Filters: workflow, status, date range (no full page reload)
  - Status badges, cost summary per run
- **Run detail page** (`web/app/(customer)/runs/[runId]/page.tsx`):
  - Step-by-step timeline visualization
  - Audit event list
  - Cost meter breakdown (by component and by step)
  - "Download bundle" action (signed URL)
  - SSE for live updates on in-flight runs
- Server components for first paint, TanStack Query for hydration

### Acceptance criteria
- Run list loads with server-side rendering
- Filters update results without page reload
- Run detail shows timeline, audit, cost
- In-flight runs show live step progress via SSE
- Download bundle produces a time-limited signed URL

### Key files to create
```
web/app/(customer)/page.tsx
web/app/(customer)/runs/[runId]/page.tsx
web/components/runs/run-list.tsx
web/components/runs/run-timeline.tsx
web/components/runs/run-cost.tsx
web/components/runs/run-audit.tsx
```

---

## Task 15: STORY-A3-015 — Dashboard Exception Queue + Cost View

**Priority:** P1
**Estimated effort:** 2 days
**Dependencies:** Task 4, Task 3

### What to build
- **Exception queue** (`web/app/(customer)/escalations/page.tsx`):
  - Open escalations grouped by severity (high > medium > low)
  - Expandable cards with Reasoner narrative, step context, screenshots
  - Inline Approve / Reject / Edit actions
  - SLA breach badge (red) with breach duration
  - SSE for real-time updates
- **Cost view** (`web/app/(customer)/cost/page.tsx`):
  - Stacked bar chart (cost by workflow, 30 days) — Recharts
  - Per-agent drill-down with outlier annotations (>2σ)
  - "Savings vs labor" panel (configurable baseline, default $35/hr)
  - Current-day spend updates via SSE (within 60s)
  - Export CSV button

### Acceptance criteria
- Exception queue renders with correct severity ordering
- Approve/Reject/Edit actions work inline
- SLA breach highlighted with red badge
- Cost charts render with brand palette
- CSV export downloads correctly
- Live updates work via SSE

### Key files to create
```
web/app/(customer)/escalations/page.tsx
web/app/(customer)/cost/page.tsx
web/components/escalations/escalation-card.tsx
web/components/escalations/resolve-form.tsx
web/components/charts/cost-bar-chart.tsx
web/components/charts/savings-panel.tsx
```

---

## Task 16: STORY-A3-016 — Traceability Matrix Generator

**Priority:** P2
**Estimated effort:** 0.5 days
**Dependencies:** Task 1

### What to build
- `scripts/build_traceability.py`:
  1. Walk `/specs/**/*.md`, parse YAML frontmatter
  2. Walk `/features/**/*.feature`, parse Feature + scenario titles
  3. Walk `/tests/**/*.py`, collect test function names
  4. Cross-reference `covers-req`, `bdd`, `tests` fields
  5. Optionally consume `reports/junit.xml` for pass/fail
  6. Replace sections in `specs/00-meta/03-traceability-matrix.md`
  7. Exit non-zero if orphans/warnings found
- Makefile target: `make traceability`
- CI integration: runs on every push

### Acceptance criteria
- Generator produces all 5 sections (Req→Story, Story→BDD→Test, NFR, ADR, Orphans)
- Orphan detection works (stories without tests, features without stories)
- CI fails if orphans exist (after bootstrap phase)
- `make traceability` runs locally

### Key files to create
```
scripts/build_traceability.py
```

---

## Dependency Graph (Critical Path)

```
Task 1 (Bootstrap)
  ├── Task 2 (CDK Infra)
  │     ├── Task 3 (FastAPI + OpenAPI)
  │     │     ├── Task 4 (Next.js Dashboard)
  │     │     │     ├── Task 14 (Run List/Detail)
  │     │     │     └── Task 15 (Exceptions/Cost)
  │     │     └── Task 14, 15
  │     ├── Task 7 (Bedrock Reasoner)
  │     ├── Task 8 (Nova Act Executor)
  │     └── Task 9 (Browser Executor)
  ├── Task 5 (Domain Models)
  │     ├── Task 6 (Workflow Validator)
  │     ├── Task 7, 8, 9
  │     ├── Task 10 (PERC Loop) ← depends on 7, 8, 9
  │     │     ├── Task 11 (Audit Writer)
  │     │     ├── Task 12 (Cost Meter)
  │     │     └── Task 13 (Slack Escalation)
  │     └── Task 11, 12, 13
  └── Task 16 (Traceability)
```

**Critical path:** Task 1 → Task 5 → Task 7/8/9 → Task 10 → Task 11/12/13

---

## Definition of Done (Per Task)

- [ ] BDD feature file exists and covers all acceptance criteria
- [ ] Tests written BEFORE implementation (red → green → refactor)
- [ ] All tests pass (`make test`)
- [ ] Lint passes (`make lint`)
- [ ] Type checking passes (mypy for Python, tsc for TypeScript)
- [ ] Spec frontmatter updated: `status: done`, `version` bumped, `last-updated` set
- [ ] No domain logic in adapters; no AWS imports in domain
- [ ] Code reviewed (or self-reviewed against ADRs)
