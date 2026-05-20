---
id: REQ-PRD-006
title: Workflow run scheduling
type: req
status: approved
owner: founder
depends-on: []
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-006 — Workflow run scheduling

## Statement

The system **shall** trigger workflow runs on cron-style schedules declared in the workflow
definition, **shall** tolerate transient downstream failures by retrying with exponential
backoff up to a configured limit, and **shall** surface missed schedules to the customer.

## Acceptance criteria

AC-1: Given a workflow with a cron `0 13 * * MON-FRI`,
      when the system clock reaches 13:00 UTC on a weekday,
      then a new run is created with `trigger=scheduled`.

AC-2: Given the orchestrator is unreachable at the scheduled time,
      when the next minute boundary occurs,
      then EventBridge Scheduler retries; up to 3 retries with backoff (1m, 5m, 30m).

AC-3: Given all retries exhausted,
      when the orchestrator remains unreachable,
      then a `RunMissed` audit event is emitted and an alert is sent to the operator.

AC-4: Given a paused agent,
      when the schedule fires,
      then the run is not created; a `RunSkipped` audit event is emitted.

## Implementation notes

- Schedules defined under `triggers.cron` in workflow YAML.
- EventBridge Scheduler resources provisioned by `infra/stacks/agentcore_stack.py`.
- Time zones: schedules are UTC by default; the YAML may declare a `timezone` field.
