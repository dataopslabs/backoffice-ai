---
id: NFR-COST-001
title: Per-run cost ceiling enforcement
type: nfr
status: approved
owner: founder
depends-on: [ADR-012]
covers-req: [REQ-PRD-002]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-COST-001 — Per-run cost ceiling

## Statement

Every workflow run **shall** abort cleanly when the synthetic cost meter exceeds the workflow's
`max_cost_per_run_usd` configuration. The default ceiling is $1.00; customers and operators may
override.

## Verification

- Property-based tests (hypothesis) generate event streams and verify ceiling enforcement at
  the orchestrator boundary.
- A regression test plays back a known oversized run and asserts the abort fires.

## Enforcement points

1. The cost meter subscriber publishes `CostMetered` events as the run progresses.
2. The cost-ceiling enforcer (in-process subscriber) compares `total_so_far_usd` against the
   configured ceiling.
3. On breach, the enforcer raises a domain signal that the PERC loop honors at the next state
   transition.
4. The orchestrator emits `RunAborted` with `reason=cost_ceiling_exceeded` and writes the
   partial-state run bundle.

## Safeguards

- The ceiling is hard, not soft. No "let it finish, we'll talk."
- Operator can override per-run via an explicit flag, audit-logged.
- A second ceiling at 2× the per-run cap acts as a kill switch in case the meter under-counts.
