# BackOfficePilot — Requirements Summary

## What is BackOfficePilot?

A **multi-model agentic automation platform** for BFSI (Banking, Financial Services, Insurance) back-office operations. It automates repetitive, rule-heavy tasks (invoice reconciliation, KYC checks, loan operations) that today require humans to navigate legacy ERPs, banking portals, and lending CRMs.

**Two AI models work together:**
- **Claude Opus 4.6** (via Amazon Bedrock) — the "Reasoner": plans workflows, interprets policy, classifies exceptions, writes audit-grade narrative.
- **Amazon Nova Act** — the "Executor": drives browser and desktop UIs on legacy systems. Production-grade UI agent.

**Runtime substrate:** AWS Bedrock AgentCore (Runtime, Gateway, Memory, Registry, Browser, Observability).

---

## Target Users

| User | Role |
|------|------|
| Ops Analyst | Reviews exceptions, approves/rejects edge cases |
| BSA / Compliance Officer | Reviews audit logs, updates policy bundles |
| CFO / COO | Reads cost meter, savings dashboard |
| BackOfficePilot Operator | Configures workflows, deploys agents, onboards customers |

---

## Core Requirements (REQ-PRD-001 through REQ-PRD-016)

### REQ-PRD-001 — Multi-tenant Isolation
No data, credential, or runtime artifact belonging to one customer is ever accessible to another customer, under any failure mode. This is a precondition for sale in regulated BFSI.

**Key constraints:**
- DDB partition keys prefixed with tenant ID
- IAM policies templated per-customer via CDK
- Secrets Manager paths prefixed by customer ID
- JWT `customer_id` claim enforced on every API call
- Cross-tenant access denied at IAM layer with CloudTrail evidence

### REQ-PRD-002 — Per-agent and Per-workflow Cost Meter
Near-real-time cost estimate per agent, per workflow, and per run — surfaced in the dashboard and via API.

**Key constraints:**
- Within 60 seconds of run start, cost estimate within $0.05 of final
- Cost decomposed by component (Opus, Nova Act, AgentCore Browser, Runtime, infra) and by step
- 30-day cost total + 30-day forecast per agent
- Weekly reconciliation against AWS billing; variance < 5% per agent

### REQ-PRD-003 — Audit Log Immutability
Every state transition of every run recorded as an `AuditEvent` in an append-only, hash-chained, tamper-evident store. No record can be deleted by any party for the retention window.

**Key constraints:**
- SHA-256 hash chain linking events
- S3 Object Lock in compliance mode (not governance)
- Signed URL + SHA-256 manifest for independent verification
- Aborted runs still capture partial state + cost snapshot

### REQ-PRD-004 — Workflow Definition Versioning
Validate every workflow YAML against JSON Schema before acceptance. Record every accepted definition as an immutable version. At most one active version per (customer, workflow).

**Key constraints:**
- Schema validation returns all errors, not just the first
- In-flight runs continue against old version (snapshot semantics)
- Rollback supported; logged as audit event
- Semver bump computed from diff

### REQ-PRD-005 — Agent Deployment via AgentCore Registry
Deploy every agent via AgentCore Registry with canary and full rollouts. Rollback to any previous version within 60 seconds.

**Key constraints:**
- Canary at configurable traffic percentage
- Existing runs unaffected by rollout
- Previous version retained as rollback candidate for 30 days
- Rollback is data-only (version pin), no rebuild

### REQ-PRD-006 — Workflow Run Scheduling
Trigger runs on cron-style schedules. Tolerate transient failures with exponential backoff (1m, 5m, 30m). Surface missed schedules.

**Key constraints:**
- EventBridge Scheduler for cron
- Paused agents skip scheduled runs (emit `RunSkipped` audit event)
- UTC by default; timezone configurable

### REQ-PRD-007 — Manual Run Trigger and Abort
Ad-hoc run trigger with optional input overrides. Abort honored within 30 seconds.

**Key constraints:**
- Abort signal via DDB attribute; orchestrator polls between steps
- Current step allowed to complete; no new steps dispatched
- Aborted run includes abort actor, timestamp, partial cost snapshot

### REQ-PRD-008 — Exception Detection and Routing
Detect when the Reasoner cannot resolve deterministically (confidence below threshold). Classify into registered exception categories. Route to escalation channel per workflow policy.

**Key constraints:**
- Confidence threshold default 0.85, configurable per workflow
- Unknown categories still escalated with full narrative
- Repeated exception on retry → run aborted with `reason=repeated_exception`

### REQ-PRD-009 — Escalation SLA Tracking
SLA timer on every open escalation. Breach event on expiry. Resolution actor and decision recorded in audit log.

**Key constraints:**
- Configurable `reviewer_sla_minutes` per workflow
- Breach → email fallback notification
- Resolution decisions: approve / reject / edit
- Edit provides modified payload; run resumes with it

### REQ-PRD-010 — Dashboard Run History View
Paginated, filterable run list. Detail view with timeline, step list, cost meter, audit log, and immutable run bundle download.

**Key constraints:**
- Filters: workflow, status, date range (no full page reload)
- Live updates via SSE for in-flight runs
- Download bundle via time-limited signed URL

### REQ-PRD-011 — Dashboard Exception Queue
Queue of open escalations grouped by severity. Inline review surface for approve/reject/edit without leaving the page.

**Key constraints:**
- Ordered by severity (high > medium > low), oldest first within severity
- Reasoner narrative, step context, screenshots visible
- SLA-breached items highlighted with red badge

### REQ-PRD-012 — Dashboard Cost View
Per-agent and per-workflow cost with daily/weekly/monthly aggregations. Comparison against labor-cost baselines. Forecast for next billing period.

**Key constraints:**
- Stacked bar chart of cost by workflow (30 days)
- Outlier annotations (>2σ runs)
- "Savings vs labor" panel with NPV
- Export CSV

### REQ-PRD-013 — Policy Bundle Versioning
Versioned history of every customer's policy bundle. Each run associated with the bundle version active at run-start. Upload without disrupting in-flight runs.

**Key constraints:**
- Semver-bumped on upload; previous version retained
- In-flight runs continue against their snapshot version
- Rollback supported; audit-logged

### REQ-PRD-014 — Audit Log Retention
Minimum 7 years retention. Customer-configurable up to 10 years. Export on demand before expiry.

**Key constraints:**
- S3 Object Lock compliance mode
- Transition to Glacier Deep Archive after retention period
- Export produces tarred bundle + SHA-256 manifest

### REQ-PRD-015 — Customer Onboarding
Operator-driven onboarding provisions all per-tenant resources (DDB, S3, KMS, IAM, AgentCore Memory, Cognito) in under 30 minutes.

**Key constraints:**
- Idempotent provisioning (retry-safe)
- Deprovision retains archive + audit per REQ-PRD-014
- All onboarding actions audit-logged

### REQ-PRD-016 — API Authentication and Authorization
JWT bearer tokens via Amazon Cognito. Scope-and-tenant policy enforcement. Cross-tenant access denied with 403.

**Key constraints:**
- Scopes: `customer:read`, `customer:write`, `operator`
- Operator role logged on every action
- Expired tokens → 401; client refreshes via Cognito

---

## Non-Functional Requirements (Summary)

| Category | ID | Title |
|----------|-----|-------|
| Security | NFR-SEC-001 | VPC isolation |
| Security | NFR-SEC-002 | Least-privilege IAM |
| Security | NFR-SEC-003 | Encryption at rest |
| Security | NFR-SEC-004 | Encryption in transit |
| Security | NFR-SEC-005 | Secrets rotation |
| Security | NFR-SEC-006 | PII redaction in logs |
| Performance | NFR-PERF-001 | API latency (p95 < target) |
| Performance | NFR-PERF-002 | Run-start latency (p95 < 5s) |
| Cost | NFR-COST-001 | Per-run cost ceiling ($1.00 for recon) |
| Cost | NFR-COST-002 | Cost meter accuracy (< 5% variance) |
| Observability | NFR-OBS-001 | OpenTelemetry tracing |
| Observability | NFR-OBS-002 | Agent-level cost visibility |
| Compliance | NFR-COMP-001 | Audit log immutability |
| Compliance | NFR-COMP-002 | SOC 2 alignment |
| Availability | NFR-AVAIL-001 | 99.5% control-plane availability |

---

## First Use Case: Invoice Reconciliation

The MVP demonstrates a daily 3-way match reconciliation workflow:
1. Download bank transaction file from banking portal (via Nova Act)
2. Export open invoices from NetSuite (via Nova Act)
3. Match transactions against invoices (Reasoner applies policy rules)
4. Post journal entries for matched items (via Nova Act)
5. Escalate exceptions (amount mismatch, missing invoice, duplicate payment) to human reviewer via Slack

**Budget:** < $1.00 per run, < 15 minutes duration.
