---
id: DESIGN-B2-005
title: Observability dashboards and per-workflow cost meters
type: design
status: approved
owner: founder
depends-on: [ADR-005, ADR-012, NFR-OBS-001, NFR-OBS-002]
covers-req: [REQ-PRD-002, REQ-PRD-012]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-B2-005 — Observability dashboards

## Goal

Use CloudWatch GenAI Observability (fed by AgentCore Observability) to give both operators and
customers visibility into the recon agent's behavior and cost.

## Operator dashboard — "Recon Agent Health"

Sections:

1. **Run volume + success** — runs per day, success rate, abort reasons.
2. **Latency** — run-start p50/p95/p99, step-execution p95 by step type.
3. **Cost** — total spend over 30 days, breakdown by component (Bedrock, Nova Act, Browser,
   Runtime), variance from forecast.
4. **Errors** — top error classes (Bedrock 5xx, NetSuite element-not-found, etc.) with links to
   exemplar traces.
5. **SLA** — open escalations, SLA-breached count, median time to resolve.
6. **Executor routing** — % runs on Nova Act vs AgentCore Browser; fallback triggers.

Built as a CloudWatch Dashboard JSON; deployed by CDK alongside the agent.

## Customer dashboard surfaces (per `REQ-PRD-012`)

- **Cost over time** — daily totals, color-coded by workflow.
- **Savings vs labor baseline** — cumulative $ saved.
- **Run history** — list with cost annotations.
- **Exception trend** — count by category over 30 days.

Customer dashboard reads from `CostMeter` / `CostDaily` DDB tables (DESIGN-A2-006); CloudWatch
dashboards stay operator-only.

## Metrics emitted by the recon agent specifically

| Metric | Unit | Use |
|---|---|---|
| `RunsStartedCount` | count | Volume baseline |
| `RunsSucceededCount` | count | Success-rate calc |
| `RunStartLatencyMs` | ms | p95 SLO |
| `StepLatencyMs` (by step.intent) | ms | Identify slow steps |
| `ExceptionsRaisedCount` (by category) | count | Trend |
| `EscalationsResolvedSeconds` | seconds | Reviewer responsiveness |
| `BedrockCachedTokenFraction` | fraction | Cache health |
| `CostPerRunUsd` | dollars | Cost drift |

## Alarms

| Alarm | Condition | Action |
|---|---|---|
| `RunStartLatencyHigh` | p95 > 5s for 15 min | Page operator |
| `CostPerRunHigh` | p95 > $1.20 for 1 hour | Page operator |
| `ExceptionsSpike` | category count > 3σ of 7-day baseline | Notify customer + operator |
| `AuditChainBreak` | any chain integrity failure | Page operator immediately |

## Linkage to traces

Every alarm includes a link to the most-recent matching trace in CloudWatch GenAI Observability
so triage starts with a concrete example.
