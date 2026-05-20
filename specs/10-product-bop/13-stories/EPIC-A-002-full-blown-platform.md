---
id: EPIC-A-002
title: Full-blown — 12-month productization
type: epic
status: approved
owner: founder
depends-on: [EPIC-A-001]
covers-req: [REQ-PRD-001, REQ-PRD-005, REQ-PRD-009, REQ-PRD-011, REQ-PRD-013, REQ-PRD-014, REQ-PRD-015, REQ-PRD-016]
version: 0.1.0
last-updated: 2026-05-19
---

# EPIC-A-002 — Full-blown product (12-month)

## Goal

Take the MVP from a demo running in UAT to a multi-customer, SOC 2-ready, paid-pilot-supporting
production product by Q4 2027.

## Themes

1. **Production deployment** — Prod account, Prod VPC, two paying customers running daily.
2. **Hardening** — SOC 2 readiness, BYOK, compliance evidence pipelines.
3. **Customer self-service** — onboarding wizard, policy bundle UI, run replay.
4. **Operational scale** — multi-region, multi-tenant scale-out, BYOK, customer-facing API
   documentation.
5. **Workflow library expansion** — KYC and loan-ops stubs.

## Stories (full-blown)

- `STORY-A4-001` Multi-account CDK + Prod environment bring-up.
- `STORY-A4-002` Cognito multi-tenant auth with scopes.
- `STORY-A4-003` AgentCore Registry canary + rollback flow.
- `STORY-A4-004` Policy bundle versioning UI.
- `STORY-A4-005` Run replay (timeline scrubber with action playback).
- `STORY-A4-006` SLA tracker + breach notifications.
- `STORY-A4-007` Teams + email escalation channels.
- `STORY-A4-008` Run history search via OpenSearch.
- `STORY-A4-009` Audit log export.
- `STORY-A4-010` BYOK KMS support (enterprise tier).
- `STORY-A4-011` SOC 2 control mapping and evidence pipeline.
- `STORY-A4-012` Customer onboarding wizard.
- `STORY-A4-013` CFO savings dashboard.
- `STORY-A4-014` Compliance officer audit-export surface.
- `STORY-A4-015` Weekly cost reconciliation job.
- `STORY-A4-016` AgentCore Identity adapter.
- `STORY-A4-017` KYC workflow stub (for library expansion).
- `STORY-A4-018` Loan-ops workflow stub.
- `STORY-A4-019` Public customer-facing API docs site.
- `STORY-A4-020` Data residency selector (region pinning).

## Definition of done (epic-level)

- 2+ paying customers running production workflows.
- SOC 2 Type I gap assessment complete; Type II observation period started.
- 99.5% control-plane availability and 99.0% workflow success rate over a rolling 90 days.
- ARR ≥ $750K.
- Workflow library has at least 2 production workflows beyond reconciliation.
