# BackOfficePilot — Implementation Plan

## Development Methodology

**Spec-Driven Development:** Every implementation follows this chain strictly:
```
SPEC → BDD → TDD → CODE
```
1. Spec exists in `/specs` (requirement, story, or design)
2. BDD `.feature` file translates acceptance criteria into Gherkin scenarios
3. pytest-bdd test files wire scenarios to assertions (tests written BEFORE code)
4. Code makes the failing tests pass

**Never skip a step.** If a spec is missing, fix the spec first. If a test is missing, fix the test first.

---

## Phase 1: MVP Platform Foundation (EPIC-A-001)

**Timeline:** 3 weeks (solo founder)
**Goal:** End-to-end BFSI invoice reconciliation workflow running in UAT with cost telemetry, audit log, and working customer dashboard.

### Sprint 1 (Week 1) — Foundation & Infrastructure

| Story | Title | Deliverable |
|-------|-------|-------------|
| STORY-A3-001 | Bootstrap repo, dev environment, CI skeleton | Monorepo structure, GitHub Actions CI, pre-commit hooks, uv + pnpm setup |
| STORY-A3-002 | CDK network + data + AgentCore stacks for UAT | VPC, DDB table, S3 buckets (Object Lock), Secrets Manager, KMS, EventBridge, AgentCore Runtime/Gateway/Memory provisioned |
| STORY-A3-003 | FastAPI control-plane scaffold + OpenAPI codegen | FastAPI app on Lambda, OpenAPI 3.1 spec, generated TypeScript client |
| STORY-A3-004 | Next.js dashboard scaffold + Cognito auth | Next.js 14 App Router, Cognito integration, auth middleware, shadcn/ui setup |
| STORY-A3-005 | Domain models with property-based tests | Pydantic v2 models for Run, Plan, Step, Exception, Escalation, CostMeter + hypothesis tests |

### Sprint 2 (Week 2) — Core Orchestration

| Story | Title | Deliverable |
|-------|-------|-------------|
| STORY-A3-006 | Workflow YAML validator | JSON Schema + Pydantic validator, cross-field validation, structured error responses |
| STORY-A3-007 | BedrockOpusReasoner adapter | ReasonerPort implementation: plan generation, reconciliation, exception classification |
| STORY-A3-008 | NovaActExecutor adapter | ExecutorPort implementation: primary UI automation via Nova Act |
| STORY-A3-009 | AgentCoreBrowserExecutor adapter | ExecutorPort fallback implementation via AgentCore Browser |
| STORY-A3-010 | PERC loop orchestrator + cost ceiling enforcer | Plan→Execute→Reconcile→Commit state machine, abort handling, cost ceiling check |

### Sprint 3 (Week 3) — Subscribers & Dashboard

| Story | Title | Deliverable |
|-------|-------|-------------|
| STORY-A3-011 | S3 hash-chain audit writer | AuditLogPort implementation: hash chain, S3 Object Lock, DDB write, verification script |
| STORY-A3-012 | Cost meter subscriber + pricing.yaml | CostMeterSubscriber, pricing config, per-run tally, rollup aggregator |
| STORY-A3-013 | Slack escalation adapter | Slack channel integration, escalation message with context + buttons |
| STORY-A3-014 | Dashboard — run list + run detail pages | Run history page, run detail with timeline/steps/audit/cost, SSE streaming |
| STORY-A3-015 | Dashboard — exception queue + cost view | Escalation queue with inline review, cost charts with Recharts |
| STORY-A3-016 | Traceability matrix generator script | `scripts/build_traceability.py` — walks specs, features, tests; generates matrix |

### MVP Definition of Done

- [ ] All 16 stories `status: done`
- [ ] Reconciliation workflow runs unattended in UAT for 5 consecutive business days
- [ ] p95 run-start latency under 5 seconds
- [ ] Per-run cost under $1.00
- [ ] Dashboard renders run list, run detail, exception queue, cost view
- [ ] CI green; traceability matrix shows ≥ 90% REQ coverage
- [ ] Demo video recorded (90 seconds, narrating one full run)

---

## Phase 2: Production Hardening (EPIC-A-002)

**Timeline:** 12 months post-MVP
**Goal:** Multi-customer, SOC 2-ready, paid-pilot-supporting production product.

### Theme 1: Production Deployment
| Story | Title |
|-------|-------|
| STORY-A4-001 | Multi-account CDK + Prod environment bring-up |
| STORY-A4-002 | Cognito multi-tenant auth with scopes |

### Theme 2: Agent Lifecycle
| Story | Title |
|-------|-------|
| STORY-A4-003 | AgentCore Registry canary + rollback flow |

### Theme 3: Customer Self-Service
| Story | Title |
|-------|-------|
| STORY-A4-004 | Policy bundle versioning UI |
| STORY-A4-005 | Run replay (timeline scrubber with action playback) |
| STORY-A4-012 | Customer onboarding wizard |
| STORY-A4-013 | CFO savings dashboard |
| STORY-A4-019 | Public customer-facing API docs site |

### Theme 4: Operational Maturity
| Story | Title |
|-------|-------|
| STORY-A4-006 | SLA tracker + breach notifications |
| STORY-A4-007 | Teams + email escalation channels |
| STORY-A4-008 | Run history search via OpenSearch |
| STORY-A4-015 | Weekly cost reconciliation job |
| STORY-A4-016 | AgentCore Identity adapter |

### Theme 5: Compliance & Security
| Story | Title |
|-------|-------|
| STORY-A4-009 | Audit log export |
| STORY-A4-010 | BYOK KMS support (enterprise tier) |
| STORY-A4-011 | SOC 2 control mapping and evidence pipeline |
| STORY-A4-014 | Compliance officer audit-export surface |
| STORY-A4-020 | Data residency selector (region pinning) |

### Theme 6: Workflow Library Expansion
| Story | Title |
|-------|-------|
| STORY-A4-017 | KYC workflow stub |
| STORY-A4-018 | Loan-ops workflow stub |

### Phase 2 Definition of Done

- [ ] 2+ paying customers running production workflows
- [ ] SOC 2 Type I gap assessment complete; Type II observation period started
- [ ] 99.5% control-plane availability over rolling 90 days
- [ ] 99.0% workflow success rate over rolling 90 days
- [ ] ARR ≥ $750K
- [ ] Workflow library has at least 2 production workflows beyond reconciliation

---

## Repository Structure

```
backofficepilot/
├── specs/                  ← Spec-driven source of truth
│   ├── 00-meta/            ← Glossary, naming conventions, traceability
│   ├── 10-product-bop/     ← Requirements, designs, stories
│   ├── 20-usecase-recon/   ← Reconciliation use-case specifics
│   ├── 30-architecture/    ← C4 diagrams, ADRs, data model, API contract
│   └── 40-non-functional/  ← NFRs (security, perf, cost, compliance)
├── features/               ← BDD feature files (one per story)
├── tests/                  ← pytest + pytest-bdd (mirrors source tree)
├── src/                    ← Python backend
│   ├── domain/             ← Pure domain models + services (NO AWS)
│   ├── application/        ← Orchestration, use cases
│   ├── adapters/           ← AgentCore, Nova Act, Bedrock, Slack, ...
│   └── api/                ← FastAPI routers (thin, delegate to application)
├── web/                    ← Next.js 14 App Router frontend
│   ├── app/                ← App Router pages
│   ├── components/         ← shadcn/ui-based
│   └── lib/                ← Generated OpenAPI client
├── workflows/              ← Workflow YAML definitions
├── policy/                 ← Customer-specific policy markdown
├── config/                 ← pricing.yaml, feature flags
├── infra/                  ← AWS CDK (Python)
├── scripts/                ← Traceability generator, codegen, onboarding
└── .github/workflows/      ← CI/CD pipelines
```

---

## Development Workflow (Per Story)

1. **Read the spec** — Find the story in `/specs/10-product-bop/13-stories/`
2. **Write BDD** — Create `features/BDD-{story-id}.feature` from acceptance criteria
3. **Write tests** — Create `tests/{layer}/test_{name}.py` with pytest-bdd step definitions (tests must be RED)
4. **Implement** — Write minimum code to turn tests GREEN
5. **Refactor** — Clean up without breaking tests
6. **Update spec** — Set `status: done`, bump `version`, set `last-updated`
7. **Update traceability** — Run `make traceability`

---

## Key Technical Decisions for Developers

1. **Domain layer has NO AWS imports.** If you need AWS, it goes in an adapter.
2. **Never modify generated OpenAPI client code.** Regenerate from the spec.
3. **Workflow YAML is configuration-as-code.** No code change needed to add a workflow.
4. **Events are the integration mechanism.** Audit, cost, and dashboard are event subscribers.
5. **Tests before code.** CI will fail if a story has no tests.
6. **Property-based testing** (hypothesis) for the orchestrator and domain models.
7. **Snapshot semantics** — in-flight runs are never affected by config changes.
