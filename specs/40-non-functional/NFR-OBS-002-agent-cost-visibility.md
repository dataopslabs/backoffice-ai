---
id: NFR-OBS-002
title: Agent-level and workflow-level cost visibility
type: nfr
status: approved
owner: founder
depends-on: [ADR-012]
covers-req: [REQ-PRD-002, REQ-PRD-012]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-OBS-002 — Agent-level cost visibility

## Statement

The customer dashboard **shall** display, with no more than 60 seconds of lag from the underlying
event, both per-agent and per-workflow cost rollups across configurable time windows (24 hours,
7 days, 30 days, custom).

## Dimensions

- **Per agent**: cost for a single customer × workflow over time.
- **Per workflow**: cost across all agents (across all customers, operator only).
- **Per run**: cost breakdown by component (Opus, Nova Act, AgentCore Browser, Runtime,
  infra).

## Rollups

- Materialized in DDB as the `CostMeter` records are updated by the meter subscriber.
- Background job aggregates per-customer daily totals into a `CostDaily` table.
- Dashboard reads from `CostMeter` (live runs) and `CostDaily` (historical).

## Verification

- Synthetic check posts events that should sum to a known value; the rollup matches within 1¢.
- Lag is observed via CloudWatch metric `CostMeterLagSeconds`; alarm on p95 > 60s.

## Operator-only views

- Cross-customer aggregations.
- Cost-by-model-id rollups.
- Cost reconciliation variance report (`NFR-COST-002`).
