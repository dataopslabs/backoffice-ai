---
id: REQ-PRD-007
title: Manual run trigger and abort
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-006]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-007 — Manual run trigger and abort

## Statement

The system **shall** allow an authorized user to trigger an ad-hoc workflow run with optional
input overrides, **shall** allow an authorized user to abort a running execution, and **shall**
guarantee that abort is honored within 30 seconds.

## Acceptance criteria

AC-1: Given an active workflow,
      when an authorized user calls `POST /customers/{id}/workflows/{wf}/runs`,
      then a new run is created with `trigger=manual` and any provided overrides applied.

AC-2: Given a running run,
      when the user calls `POST /runs/{id}/abort`,
      then the orchestrator receives the abort signal within 5 seconds and stops dispatching
      new steps within 30 seconds.

AC-3: Given an in-flight executor call when abort is requested,
      when the abort signal is processed,
      then the current step is allowed to complete (or be cancelled if the executor supports it);
      no new steps are dispatched.

AC-4: Given an aborted run,
      when the run bundle is finalized,
      then it includes the abort actor, abort timestamp, and partial-state cost meter snapshot.

## Implementation notes

- Abort signal carried by a DDB attribute on the run record; orchestrator polls between steps.
- Idempotency: repeated abort calls return 202 with the original abort timestamp.
- Authorization: requires `customer:write` for that customer or `operator` scope.
