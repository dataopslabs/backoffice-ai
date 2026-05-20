---
id: STORY-A3-010
title: PERC loop orchestrator + cost ceiling enforcer
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, STORY-A3-007, STORY-A3-008, DESIGN-C4-003, NFR-COST-001]
covers-req: [REQ-PRD-007, REQ-PRD-008]
bdd: [BDD-A3-010.feature]
tests: [tests/application/test_perc_loop.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-010 — PERC loop + cost ceiling

## Story

As the founder, I need the orchestrator's Plan → Execute → Reconcile → Commit loop implemented
in `application/perc_loop.py` with property-based tests covering happy path, retries,
escalations, aborts, and cost-ceiling breaches.

## Acceptance criteria

AC-1: Given a workflow definition with 5 sequential steps,
      when the loop runs,
      then all 5 steps execute in order, and a `RunCompleted` event is emitted.

AC-2: Given step 3 fails with `element_not_found`,
      when reconciliation chooses `retry`,
      then the step is retried up to `max_retries` (default 2), with backoff; subsequent
      failure triggers escalation.

AC-3: Given the cost-meter publishes a `CostMetered` event with `ceiling_breached=true`,
      when the loop checks before dispatching the next step,
      then the run is aborted with `reason=cost_ceiling_exceeded`.

AC-4: Given an abort signal arrives mid-step,
      when the executor returns,
      then no further steps are dispatched and the run commits cleanly with status=aborted.

AC-5: Given hypothesis generates arbitrary step + result sequences,
      when applied to the loop,
      then no invariant violations occur (e.g. no `RunCompleted` after `RunAborted`).

## Technical notes

- State machine implemented as a plain Python class; not a third-party FSM library.
- Cost-ceiling check between steps, not within steps.
- All transitions emit events; subscribers handle persistence (`STORY-A3-011`, `STORY-A3-012`).

## Definition of done

- [ ] PERC loop covered by hypothesis suite.
- [ ] Integration test runs a 5-step workflow end-to-end against mock adapters.
- [ ] BDD `BDD-A3-010.feature` covers each AC.
