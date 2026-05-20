---
id: STORY-A3-012
title: Cost meter subscriber + pricing.yaml
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-A2-006, NFR-COST-001, NFR-COST-002]
covers-req: [REQ-PRD-002]
bdd: [BDD-A3-012.feature]
tests: [tests/adapters/test_cost_meter.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-012 — Cost meter subscriber

## Story

As the founder, I need the cost meter subscriber implemented, consuming Bedrock, Nova Act,
AgentCore Browser, and Runtime events; computing per-run cost from `config/pricing.yaml`;
emitting `CostMetered` at intervals; and writing rollups to DDB.

## Acceptance criteria

AC-1: Given a stream of `BedrockInvocationCompleted` events for a run,
      when the meter processes them,
      then the running tally equals `Σ (tokens × pricing) + Σ (agent-seconds × pricing)`
      within rounding tolerance.

AC-2: Given the run's accumulated cost crosses the ceiling,
      when the next `CostMetered` event is emitted,
      then `ceiling_breached=true` is set; the orchestrator aborts on its next state check.

AC-3: Given pricing.yaml version is bumped during a run,
      when events arrive after the bump,
      then the new pricing is applied; events before the bump used the older pricing; the run
      record stores both pricing versions encountered.

AC-4: Given a completed run,
      when the cost meter is finalized,
      then the bundle contains a per-step cost breakdown matching `CostMeter.by_step`.

AC-5: Given the weekly reconciliation job runs,
      when synthetic costs vs AWS bill diverge by > 5%,
      then an alert fires (`NFR-COST-002`).

## Technical notes

- Subscriber runs in-process within the orchestrator (lowest-latency).
- Secondary EventBridge publisher fans events to a Lambda-based meter for cross-process
  reads (dashboard).
- Pricing.yaml schema validated on load.

## Definition of done

- [ ] Subscriber covered by ACs.
- [ ] Pricing.yaml committed with v1 rates.
- [ ] Reconciliation job scaffold present (full run scheduled in `STORY-A4-015`).
