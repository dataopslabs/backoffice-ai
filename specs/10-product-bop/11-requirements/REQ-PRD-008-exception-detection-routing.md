---
id: REQ-PRD-008
title: Exception detection and routing
type: req
status: approved
owner: founder
depends-on: [ADR-004, REQ-PRD-009]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-008 — Exception detection and routing

## Statement

The system **shall** detect when a step produces a result the Reasoner cannot resolve
deterministically, **shall** classify the result into a registered exception category, and
**shall** route the exception to an escalation channel per the workflow's escalation policy.

## Acceptance criteria

AC-1: Given a step result the Reasoner judges as ambiguous (confidence below threshold),
      when reconcile completes,
      then an `Exception` record is created and an `ExceptionRaised` event emitted.

AC-2: Given an exception is raised,
      when the exception's category matches a registered category,
      then it is escalated via the workflow's configured channel (Slack, Teams, or email).

AC-3: Given an exception is raised whose category is `unknown`,
      when the category is unrecognized,
      then it is still escalated, with the category marked `unknown` and the Reasoner's full
      narrative attached.

AC-4: Given the same step is retried after an exception,
      when the second attempt also raises an exception,
      then the run is aborted with `reason=repeated_exception`; per-run cost meter is captured.

## Implementation notes

- Exception categories registered in `workflows/<name>.yaml` under `exception_categories`.
- Routing logic lives in `domain/exception_classifier.py` (pure) + `application/exception_router.py`
  (chooses channel adapter).
- Confidence threshold default 0.85; configurable per workflow.

## Linked

- `REQ-PRD-009` Escalation SLA tracking.
- `DESIGN-A2-004` Exception routing design.
