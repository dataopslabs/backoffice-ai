---
id: SPEC-EVT-001
title: Domain event catalog
type: spec
status: approved
owner: founder
depends-on: [ADR-006, DESIGN-DATA-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# Domain event catalog

This is the canonical list of domain events emitted on the internal event bus
(`EventBusPort`). All events share a common envelope; specific payload schemas are listed per
event. Schemas in JSON form live in `35-event-catalog/schemas/`.

## Envelope

Every event published to the bus carries:

```json
{
  "event_id": "evt_01H...",
  "event_type": "RunCompleted",
  "event_version": "1.0",
  "occurred_at": "2026-05-20T13:04:21.193Z",
  "actor": "agent_recon_acme",
  "customer_id": "cust_acme",
  "run_id": "run_20260520_001",
  "trace_id": "<otel-trace-id>",
  "payload": { /* event-specific */ }
}
```

| Field | Meaning |
|---|---|
| `event_id` | ULID. Unique. |
| `event_type` | PascalCase past-tense verb. Stable. |
| `event_version` | semver. Bumped on payload change. |
| `occurred_at` | ISO-8601 UTC. |
| `actor` | The entity that emitted the event (usually an agent ID). |
| `customer_id` | Always present for runtime events; null for system events. |
| `run_id` | Present when the event belongs to a workflow run. |
| `trace_id` | OpenTelemetry trace ID for cross-system correlation. |
| `payload` | Event-specific body. |

## Catalog

### Run lifecycle

#### `WorkflowRunStarted` (v1.0)

Emitted when the orchestrator begins a run.

```json
{
  "workflow_id": "wf_recon_daily",
  "workflow_version": "1.4.2",
  "trigger": "scheduled",
  "input_summary": { "transaction_date": "2026-05-19", "expected_rows": 47 }
}
```

Consumers: Audit log, cost meter (initializes counters), dashboard.

#### `PlanCreated` (v1.0)

Emitted after the Reasoner produces a plan.

```json
{
  "plan_id": "plan_...",
  "step_count": 5,
  "reasoning_trace_uri": "s3://...",
  "opus_input_tokens": 8203,
  "opus_output_tokens": 1976,
  "cached_token_fraction": 0.87,
  "model_id": "anthropic.claude-opus-4-6"
}
```

Consumers: Audit log, cost meter, dashboard.

#### `StepDispatched` (v1.0)

Emitted before the Executor begins a step.

```json
{
  "step_id": "s2",
  "sequence": 2,
  "intent": "Export open invoices from NetSuite",
  "executor": "novaact",
  "system": "netsuite",
  "expected_duration_seconds": 30
}
```

Consumers: Audit log, dashboard (live progress).

#### `StepCompleted` (v1.0)

Emitted on successful step completion.

```json
{
  "step_id": "s2",
  "status": "success",
  "duration_seconds": 28,
  "observations_summary": { "rows_exported": 89 },
  "executor_cost_usd": 0.084
}
```

Consumers: Audit log, cost meter, dashboard.

#### `StepFailed` (v1.0)

Emitted when a step terminates without success.

```json
{
  "step_id": "s2",
  "failure_class": "element_not_found",
  "duration_seconds": 14,
  "will_retry": true,
  "retry_count": 1
}
```

Consumers: Audit log, dashboard, alerting.

#### `ReconcileDecided` (v1.0)

Emitted after the Reasoner judges a step result.

```json
{
  "step_id": "s2",
  "decision": "advance",
  "confidence": 0.97,
  "reasoning_uri": "s3://..."
}
```

`decision` is one of `advance | retry | escalate | abort`.

Consumers: Audit log, dashboard.

#### `ExceptionRaised` (v1.0)

Emitted when an exception is classified.

```json
{
  "exception_id": "exc_...",
  "step_id": "s4",
  "category": "amount_mismatch",
  "severity": "medium",
  "reasoner_summary": "Payment $9,847.32 vs invoice $9,847.23; outside $0.05 tolerance.",
  "confidence": 0.91
}
```

Consumers: Audit log, dashboard, escalation router.

#### `EscalationOpened` (v1.0)

Emitted when an exception is routed to a human.

```json
{
  "escalation_id": "esc_...",
  "exception_id": "exc_...",
  "channel": "slack",
  "thread_ref": "C012/p1700000",
  "assignee": "ops-on-call"
}
```

Consumers: Audit log, SLA tracker, dashboard.

#### `EscalationResolved` (v1.0)

Emitted when a human resolves an escalation.

```json
{
  "escalation_id": "esc_...",
  "decision": "approve",
  "reviewer": "alice@acme.bank",
  "minutes_open": 24,
  "sla_breached": false
}
```

Consumers: Audit log, dashboard, run resume hook.

#### `RunCompleted` (v1.0)

Emitted when a run ends successfully.

```json
{
  "status": "succeeded",
  "step_count": 5,
  "exception_count": 1,
  "duration_seconds": 345,
  "total_cost_usd": 0.73,
  "bundle_uri": "s3://..."
}
```

Consumers: Audit log, cost meter (finalize), dashboard, customer-summary email job.

#### `RunAborted` (v1.0)

Emitted when a run is aborted by cost ceiling, operator action, or unrecoverable failure.

```json
{
  "reason": "cost_ceiling_exceeded",
  "step_count_completed": 3,
  "partial_cost_usd": 1.04
}
```

Consumers: Audit log, alerting, dashboard.

### Telemetry

#### `BedrockInvocationCompleted` (v1.0)

```json
{
  "model_id": "anthropic.claude-opus-4-6",
  "purpose": "plan",
  "input_tokens": 8203,
  "output_tokens": 1976,
  "cached_input_tokens": 7137,
  "estimated_cost_usd": 0.047
}
```

Consumers: cost meter.

#### `NovaActSessionTicked` (v1.0)

```json
{
  "session_id": "nova_...",
  "step_id": "s2",
  "agent_seconds": 28,
  "estimated_cost_usd": 0.037
}
```

Consumers: cost meter.

#### `AgentCoreBrowserUsed` (v1.0)

```json
{
  "session_id": "acb_...",
  "step_id": "s2",
  "duration_seconds": 30,
  "estimated_cost_usd": 0.020
}
```

Consumers: cost meter.

#### `AgentRuntimeTicked` (v1.0)

```json
{
  "duration_seconds": 350,
  "estimated_cost_usd": 0.011
}
```

Consumers: cost meter.

#### `CostMetered` (v1.0)

Emitted when a cost meter rollup crosses a threshold or every minute during an active run.

```json
{
  "scope": "run",
  "scope_id": "run_20260520_001",
  "total_so_far_usd": 0.42,
  "ceiling_usd": 1.00,
  "ceiling_breached": false
}
```

Consumers: dashboard, alerting, cost-ceiling enforcer.

## Versioning rules

- **Additive payload change** (new optional field) → patch bump in `event_version`. Subscribers
  ignore unknown fields.
- **Required field added or field removed** → major bump. New event type required
  (`RunCompletedV2`) and old type kept for a deprecation window.

## Consumers (high-level)

| Consumer | Subscribes to |
|---|---|
| **Audit log writer** | All events. Writes hash-chained `AuditEvent` rows. |
| **Cost meter** | All `*Ticked`, `*Used`, `*InvocationCompleted`, `RunCompleted`, `RunAborted`. |
| **Dashboard fanout** | `WorkflowRunStarted`, `StepDispatched`, `StepCompleted`, `StepFailed`, `ReconcileDecided`, `ExceptionRaised`, `EscalationOpened`, `EscalationResolved`, `RunCompleted`, `RunAborted`, `CostMetered`. |
| **Cost-ceiling enforcer** | `CostMetered`. Triggers `RunAborted` when ceiling breached. |
| **Customer-summary email job** | `RunCompleted`. Sends a daily digest. |
| **SLA tracker** | `EscalationOpened`, `EscalationResolved`. |

## Bus implementation

- In-process: `InProcessEventBus` (Python). Default for the orchestrator process.
- Cross-process: `EventBridgeBus` (AWS EventBridge custom bus, one per environment).
- Both implement `EventBusPort`. The orchestrator publishes to both; subscribers choose.

Schemas live as JSON Schema in `35-event-catalog/schemas/<EventType>.schema.json`. Each is
referenced by `event_type` for runtime validation.
