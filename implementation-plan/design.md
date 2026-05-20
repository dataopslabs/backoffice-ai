# BackOfficePilot — Architecture & Design

## System Architecture Overview

BackOfficePilot follows **hexagonal architecture** (ports & adapters) with **Domain-Driven Design** (light). The system is event-driven internally, API-first externally, and configuration-as-code for workflows.

---

## C4 Level 1 — System Context

```
┌─────────────────────────────────────────────────────────────────────┐
│                        BackOfficePilot                               │
│         Multi-Model Agent Platform (BFSI Back-Office)               │
└─────────────────────────────────────────────────────────────────────┘
        ▲               ▲               ▲               ▲
        │               │               │               │
   Ops Analyst    BSA Officer      CFO/COO        BOP Operator
   (exceptions)  (audit/policy)   (cost/savings)  (deploy/config)

External Dependencies:
  • Amazon Bedrock (Claude Opus 4.6) — Reasoner
  • Amazon Nova Act — Primary Executor (UI automation)
  • AWS AgentCore (6 services) — Runtime substrate
  • Customer Systems (ERP, Banking Portal, CRM, Slack/Teams)
  • CloudWatch GenAI Observability — Metrics/traces
```

---

## C4 Level 2 — Containers (Deployable Units)

| Container | Technology | Responsibility |
|-----------|-----------|----------------|
| **Next.js Dashboard** | Next.js 14 App Router, TS, Tailwind, shadcn/ui | Customer-facing UI |
| **Control Plane API** | FastAPI on Lambda behind API Gateway | CRUD API, workflow management, signed URLs |
| **EventBridge Scheduler** | AWS EventBridge | Cron-triggered workflow runs |
| **Orchestrator Agent** | Python on AgentCore Runtime | PERC loop execution |
| **AgentCore Gateway** | AWS managed | Tool surface for agents |
| **AgentCore Memory** | AWS managed | Session + long-term context |
| **AgentCore Registry** | AWS managed | Agent version management |
| **DynamoDB** | Single-table design | Run state, cost rollups, escalation timers |
| **S3 + Object Lock** | WORM storage | Immutable run bundles, audit logs |
| **OpenSearch** | AWS managed | Run history search |
| **Secrets Manager** | AWS managed | Customer credentials (rotated) |
| **KMS** | AWS managed | Customer-managed encryption keys |

---

## C4 Level 3 — Orchestrator Internal Components

The orchestrator is the core of the system. It follows hexagonal architecture:

```
┌─────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR AGENT                             │
│                                                                   │
│  ┌─── API Layer ───────────────────────────────────────────────┐ │
│  │  WorkflowRunController (thin FastAPI router)                 │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│  ┌─── Application Layer ───────────────────────────────────────┐ │
│  │  RunWorkflowUseCase                                          │ │
│  │  PERCLoop (Plan → Execute → Reconcile → Commit)             │ │
│  │  ExecutorRouter                                              │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│  ┌─── Domain Layer (PURE — no AWS knowledge) ──────────────────┐ │
│  │  Workflow, Plan, Step, Exception models                      │ │
│  │  ExceptionClassifier                                         │ │
│  │  PolicyApplier                                               │ │
│  │  CostCeilingEnforcer                                         │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│  ┌─── Ports (interfaces) ──────────────────────────────────────┐ │
│  │  ReasonerPort, ExecutorPort, MemoryPort                     │ │
│  │  EventBusPort, AuditLogPort, SecretPort                     │ │
│  └──────────────────────────────────────────────────────────────┘ │
│                              │                                    │
│  ┌─── Adapters (implementations) ─────────────────────────────┐ │
│  │  BedrockOpusReasoner, NovaActExecutor                       │ │
│  │  AgentCoreBrowserExecutor (fallback)                        │ │
│  │  AgentCoreMemoryAdapter, EventBridgeBus                     │ │
│  │  S3HashChainAuditLog, SecretsManagerAdapter                 │ │
│  └──────────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────────┘
```

---

## PERC Loop (Core Orchestration Pattern)

The orchestrator runs a **Plan → Execute → Reconcile → Commit** loop:

1. **Plan** — Reasoner (Claude Opus) analyzes the workflow definition + policy bundle and produces a step-by-step plan.
2. **Execute** — Executor (Nova Act primary, AgentCore Browser fallback) performs each step on the target system.
3. **Reconcile** — Reasoner evaluates step results against policy. If confidence < threshold → exception raised.
4. **Commit** — Results persisted, audit events emitted, cost meter updated.

Between each step: check abort signal, check cost ceiling, emit domain events.

---

## Data Model — Core Entities

| Entity | Primary Store | Description |
|--------|--------------|-------------|
| **Customer** | DynamoDB | BFSI institution, tier (pilot/platform/enterprise) |
| **Workflow** | DynamoDB | Versioned YAML definition of an automation procedure |
| **Agent** | DynamoDB | Deployed instance on AgentCore Runtime |
| **Run** | DDB (state) + S3 (bundle) | Single execution of a workflow |
| **Plan** | DDB + S3 | Reasoner's output: steps, success criteria, reasoning trace |
| **Step** | DDB + S3 | Unit of executor work within a run |
| **Exception** | DynamoDB | Condition the agent cannot resolve deterministically |
| **Escalation** | DynamoDB | Routing of exception to human reviewer with SLA |
| **AuditEvent** | S3 + Object Lock | Append-only, hash-chained state transition record |
| **CostMeter** | DDB + Athena | Per-run cost rollup (live + historical) |
| **PolicyBundle** | S3 + DDB pointer | Versioned markdown rules for the Reasoner |

### DynamoDB Single-Table Design

| Access Pattern | PK | SK |
|---------------|-----|-----|
| Get customer | `CUSTOMER#{id}` | `META` |
| List workflows | `CUSTOMER#{id}` | `WORKFLOW#{wf_id}` |
| Get run | `RUN#{run_id}` | `META` |
| List steps | `RUN#{run_id}` | `STEP#{seq}` |
| Open escalations (GSI) | `ESCQ#{customer_id}` | `OPEN#{opened_at}` |

---

## Tenant Isolation Design

Every resource is scoped by customer ID:

| Resource | Naming Pattern |
|----------|---------------|
| DDB partition keys | `CUSTOMER#{id}`, `RUN#{id}` |
| S3 paths | `s3://bop-<env>-{purpose}/cust={id}/...` |
| Secrets Manager | `cust/{id}/{system}/{secret_name}` |
| KMS aliases | `alias/bop-{env}-cust-{id}` |
| AgentCore Memory | `bop/{env}/cust/{id}` |
| Cognito app client | One per customer |

CDK construct `PerCustomerIamRole` generates IAM policies scoped to the customer's resources. Cross-tenant access is denied at the IAM layer.

---

## Workflow Definition Format (YAML)

```yaml
name: daily_reconciliation
triggers:
  cron: "0 13 * * MON-FRI"
  timezone: "UTC"
policy_documents:
  - policy/three_way_match.md
  - policy/journal_posting_rules.md
systems:
  - id: bank_portal
    type: web
    base_url: ${BANK_PORTAL_URL}
    credentials_secret: bank/creds
  - id: netsuite
    type: web
    base_url: ${NETSUITE_URL}
    credentials_secret: netsuite/creds
steps:
  - id: pull_bank_file
    system: bank_portal
    intent: "Download yesterday's transaction file"
    success: { file_present: true, min_rows: 20 }
    timeout_seconds: 60
  - id: match
    type: reason
    inputs: [pull_bank_file, pull_open_invoices]
  - id: escalate_exceptions
    type: escalate
    channel: slack
exception_categories:
  - amount_mismatch
  - missing_invoice
  - duplicate_payment
budget:
  max_cost_per_run_usd: 1.00
  max_duration_minutes: 15
escalation:
  channel: "#bop-reconciliation"
  reviewer_sla_minutes: 60
  fallback: email
```

---

## Cost Meter Design

```
EventBus → CostMeterSubscriber → pricing.yaml → RunTally → CostCeilingEnforcer
                                                    ↓
                                              RollupAggregator → DDB (CostMeter/CostDaily)
                                                                      ↓
                                                              Customer Dashboard
```

- Pricing source-of-truth: `config/pricing.yaml` (versioned, PR-required for changes)
- Per-run tally updated every 30s or every $0.10
- Cost ceiling: soft (configured) + hard (2× configured as kill switch)
- Weekly reconciliation job compares synthetic cost to AWS billing

---

## Audit Log Design (Hash Chain + WORM)

```python
def write_audit_event(event, prev_hash):
    payload = canonicalize(event)  # deterministic JSON, sorted keys
    content = f"{prev_hash}\n{payload}"
    this_hash = sha256(content.encode()).hexdigest()
    # Write to DDB (fast query) + S3 with Object Lock (tamper-evident)
```

- Each run has its own sequence counter (DDB atomic increment)
- On RunCompleted/RunAborted: full event stream packaged as `audit.jsonl`
- Verification script (`scripts/verify_audit.py`) recomputes chain independently

---

## Exception Routing Pipeline

```
PERCLoop.reconcile (low confidence) → ExceptionClassifier (pure domain)
    → ExceptionRouter → EventBus → EscalationOpener
        → Channel selection (Slack / Teams / Email)
        → SLA timer started (DDB TTL + Step Functions wait)
```

Severity defaults: `duplicate_payment` = high, `amount_mismatch > 1%` = medium, `unknown` = high.

---

## Dashboard Architecture

- **Framework:** Next.js 14 App Router, TypeScript strict mode
- **Styling:** Tailwind CSS + shadcn/ui (navy, ice, gold, teal, coral palette)
- **Data fetching:** React Server Components (first paint) + TanStack Query (hydration) + SSE (live updates)
- **Charts:** Recharts
- **Auth:** Cognito hosted UI → JWT in httpOnly cookie → Next.js middleware validates
- **Performance targets:** LCP < 1.5s, INP < 200ms, WCAG 2.2 AA

---

## Key Architecture Decision Records (ADRs)

| ADR | Decision |
|-----|----------|
| ADR-001 | FastAPI over Django (lighter, async, OpenAPI-native) |
| ADR-002 | Hexagonal architecture (domain purity, testability) |
| ADR-003 | Spec-driven development (SPEC → BDD → TDD → CODE) |
| ADR-004 | Multi-model orchestration (Reasoner + Executor split) |
| ADR-005 | AgentCore Runtime as host (serverless, managed) |
| ADR-006 | Event-driven internal bus (loose coupling, audit) |
| ADR-007 | pytest-bdd over Behave (better pytest integration) |
| ADR-008 | OpenAPI spec-first (contract-driven, codegen) |
| ADR-009 | VPC isolation prod/UAT (separate accounts, no shared data) |
| ADR-010 | CDK Python over Terraform (same language as backend) |
| ADR-011 | Nova Act primary, AgentCore Browser fallback |
| ADR-012 | Cost meter from observability events (event-sourced) |

---

## Deployment Architecture

- **Environments:** UAT and Prod in separate AWS accounts, separate VPCs
- **Infrastructure:** AWS CDK (Python), per-VPC stacks
- **Frontend:** Vercel or AWS Amplify (outside VPC, reaches control plane via API Gateway)
- **Agent hosting:** AgentCore Runtime (serverless)
- **CI/CD:** GitHub Actions → CDK deploy → AgentCore Registry push → canary rollout

---

## Technology Stack (Locked — ADR required to change)

| Layer | Choice |
|-------|--------|
| Frontend | Next.js 14 App Router, TypeScript, Tailwind, shadcn/ui |
| Backend | Python 3.12, FastAPI, Pydantic v2, uv (package manager) |
| BDD | pytest-bdd |
| TDD | pytest, hypothesis (property-based), Playwright (E2E) |
| Infra | AWS CDK (Python), per-VPC stacks |
| Frontend tooling | pnpm |
| Observability | OpenTelemetry SDK → AgentCore Observability → CloudWatch |
