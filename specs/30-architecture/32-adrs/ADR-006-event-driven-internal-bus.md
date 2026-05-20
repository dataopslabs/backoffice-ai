---
id: ADR-006
title: Internal event bus for domain events
type: adr
status: approved
owner: founder
depends-on: [ADR-002]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-006: Internal event bus for domain events

## Context

Several capabilities need to react to the same orchestrator activity:

- **Audit log**: persist every state transition with a hash chain.
- **Cost meter**: tally per-agent and per-workflow spend in near-real time.
- **Customer dashboard**: stream live status to the Next.js UI via Server-Sent Events.
- **Compliance hooks**: trigger reviews on specific exception categories.
- **Future analytics**: feed ML models for exception prediction and policy improvement.

If each capability calls into the orchestrator (or vice versa) the coupling becomes unmanageable.

## Decision

The orchestrator emits **domain events** to an internal event bus. Subscribers are independent
modules that respond to events. The event catalog lives in `30-architecture/35-event-catalog/`.

**Transport**: in-process publish/subscribe via a Python `EventBusPort`. Production adapter
publishes the same events to EventBridge for cross-process consumers (cost meter and dashboard
fan-out).

Event types (initial):

- `WorkflowRunStarted`
- `PlanCreated`
- `StepDispatched`
- `StepCompleted`
- `StepFailed`
- `ReconcileDecided`
- `ExceptionRaised`
- `EscalationOpened`
- `EscalationResolved`
- `RunCompleted`
- `RunAborted`
- `CostMetered`

Schemas in `35-event-catalog/`.

## Alternatives considered

- **Synchronous calls**: keeps things simple; couples everything to the orchestrator.
- **Kafka / MSK**: overkill; cost not justified at our scale.
- **SQS only**: lacks fanout to multiple subscribers without per-consumer queues.
- **EventBridge directly**: chosen for cross-process fanout, but adds network latency in the hot
  path. We use it as a secondary publisher, not the primary.

## Consequences

**Positive**

- Adding a new consumer (e.g. compliance webhook) does not touch the orchestrator.
- Each subscriber tested in isolation; the orchestrator tested without subscribers.
- The event log is a natural audit trail.

**Negative**

- Two-step debugging for cross-subscriber issues.
- Event schema is a contract; breaking changes require versioning.
- "Eventual consistency" between the dashboard and the database. Acceptable for our use case.

## Versioning

Events use semver in their schema. Breaking changes require a new event type
(e.g. `RunCompletedV2`) and a migration plan. Schemas live in `35-event-catalog/`.
