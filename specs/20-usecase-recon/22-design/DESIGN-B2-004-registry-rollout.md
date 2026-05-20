---
id: DESIGN-B2-004
title: AgentCore Registry rollout for the recon use case
type: design
status: approved
owner: founder
depends-on: [ADR-005, DESIGN-A2-003]
covers-req: [REQ-PRD-005]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-B2-004 — Registry rollout for recon

## Goal

Apply the general agent rollout flow (`DESIGN-A2-003`) to the Acme reconciliation use case
specifically: which versions to register, what the canary signal looks like, and how to roll
back during a paid pilot.

## Initial version registration

- Version `0.1.0` corresponds to MVP feature-complete (end of Session 3 sprint).
- Subsequent versions follow semver per the orchestrator-image semver, not per workflow YAML.

## Canary signal

The canary is deemed healthy if, over the last hour of canary traffic:

| Metric | Threshold |
|---|---|
| Run success rate | ≥ 95% |
| p95 run-start latency | ≤ 5s |
| Per-run cost | ≤ $1.00 (95th percentile) |
| Exception miscategorization rate | ≤ 2% (sampled by operator) |
| `SlaBreached` events on canary runs | 0 |

If any threshold is breached for 30+ minutes during a canary, the auto-rollback rule fires.

## Rollback drill

We rehearse rollback monthly in UAT:

1. Deploy a deliberately-broken version (e.g. a no-op orchestrator).
2. Trigger one run; observe failure.
3. Trigger rollback to previous version.
4. Confirm new runs use the previous version within 60 seconds.
5. Verify no in-flight runs were corrupted.

A successful rollback drill is the gate before promoting to Prod.

## Prod cutover plan

- T-7d: Final UAT sign-off (5-day unattended success).
- T-3d: Prod stacks deployed (`STORY-A4-001`). Customer account configured.
- T-1d: Customer dry-run with synthetic data in Prod.
- T-0:  First real run with live data. Operator on standby.

## Future canary improvements

- Per-customer canary slice (e.g. 10% of one customer's runs, not 10% across customers).
- Multi-day canary with seasonality-aware comparison to baseline.
