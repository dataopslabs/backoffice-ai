---
id: REQ-PRD-002
title: Per-agent and per-workflow cost meter
type: req
status: approved
owner: founder
depends-on: [ADR-012]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-002 — Per-agent and per-workflow cost meter

## Statement

The system **shall** surface, in the customer dashboard and via API, a near-real-time estimate of
the cost incurred by each agent, each workflow, and each individual run.

## Rationale

Outcome-based pricing requires the customer to see cost as it accrues. Cost ceilings cannot be
enforced without a meter. The cost story is also a key differentiator in the pitch.

## Acceptance criteria

AC-1: Given a run in progress,
      when 60 seconds have elapsed since the run started,
      then the cost meter displays an estimate within $0.05 of the eventual final cost.

AC-2: Given a completed run,
      when the customer dashboard loads the run detail,
      then the cost is decomposed by component (Opus, Nova Act, AgentCore Browser, Runtime,
      infrastructure) and by step.

AC-3: Given the agent list view,
      when the customer opens it,
      then each agent shows a 30-day cost total and a forecast for the next 30 days.

AC-4: Given a workflow with multiple agents (multiple customers),
      when an operator opens the workflow analytics view,
      then a per-workflow cost rollup is shown, summed across all agents using that workflow.

AC-5: Given the weekly reconciliation job runs,
      when comparing synthetic cost to actual AWS billing,
      then variance shall be < 5% per agent; greater drift triggers an alert (see `NFR-COST-002`).

## Implementation notes

- Cost values are derived from event bus subscriptions (`ADR-012`).
- Pricing source-of-truth: `config/pricing.yaml`, versioned.
- Customer-facing display labels the figure "BackOfficePilot estimate" with a methodology link.

## Linked NFRs

- `NFR-COST-001` Per-run cost ceiling enforcement.
- `NFR-COST-002` Cost-meter accuracy.
- `NFR-OBS-002` Agent-level cost visibility.
- `NFR-OBS-003` Workflow-level cost rollup.
