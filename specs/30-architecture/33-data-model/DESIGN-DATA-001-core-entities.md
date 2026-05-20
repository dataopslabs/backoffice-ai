---
id: DESIGN-DATA-001
title: Core entities and relationships
type: design
status: approved
owner: founder
depends-on: [DESIGN-C4-003, ADR-002, ADR-006]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# Core entities and relationships

This document defines the canonical entities in the BackOfficePilot domain. Every entity has a
representation in the domain layer (`src/domain/`) and is persisted via one or more adapters.

```mermaid
erDiagram
    Customer ||--o{ Workflow : "configures"
    Workflow ||--o{ Agent : "instantiates"
    Agent ||--o{ Run : "produces"
    Run ||--|{ Step : "contains"
    Run ||--|| Plan : "starts with"
    Plan ||--|{ Step : "schedules"
    Step ||--o| Exception : "may raise"
    Exception ||--o| Escalation : "routed to"
    Run ||--|{ AuditEvent : "emits"
    Run ||--|| CostMeter : "tallies"
    Customer ||--|{ PolicyBundle : "owns"
    Workflow }o--|| PolicyBundle : "references"

    Customer {
        string customer_id PK
        string name
        string vpc_environment "prod | uat"
        string tier "pilot | platform | enterprise"
        datetime created_at
    }

    Workflow {
        string workflow_id PK
        string customer_id FK
        string name
        string version "semver"
        json definition "YAML parsed"
        json executor_requirements
        decimal max_cost_per_run_usd
        int max_duration_minutes
    }

    Agent {
        string agent_id PK
        string workflow_id FK
        string registry_version "AgentCore Registry version"
        string status "active | paused | rolling_out | retired"
        datetime deployed_at
    }

    Run {
        string run_id PK
        string agent_id FK
        string status "pending | running | succeeded | failed | aborted"
        datetime started_at
        datetime finished_at
        decimal total_cost_usd
        string trigger "scheduled | manual | retry"
        string parent_run_id "if retry"
    }

    Plan {
        string plan_id PK
        string run_id FK
        json steps
        string reasoning_trace
        decimal opus_input_tokens
        decimal opus_output_tokens
        datetime created_at
    }

    Step {
        string step_id PK
        string run_id FK
        int sequence
        string intent
        string system
        string status "pending | success | failure | partial"
        json observations
        json actions_taken
        int duration_seconds
        string executor "novaact | agentcore_browser"
        decimal cost_usd_estimate
        datetime started_at
        datetime finished_at
    }

    Exception {
        string exception_id PK
        string step_id FK
        string category "amount_mismatch | missing_invoice | duplicate_payment | currency | other"
        string severity "low | medium | high"
        string reasoner_decision
        string reasoner_confidence
        json context
    }

    Escalation {
        string escalation_id PK
        string exception_id FK
        string channel "slack | teams | email"
        string thread_ref
        string assigned_to
        string status "open | acknowledged | resolved"
        datetime opened_at
        datetime resolved_at
        int sla_minutes_breached
    }

    AuditEvent {
        string event_id PK
        string run_id FK
        int sequence
        string event_type
        string prev_hash
        string this_hash
        json payload
        datetime timestamp
    }

    CostMeter {
        string meter_id PK
        string run_id FK
        decimal opus_usd
        decimal novaact_usd
        decimal agentcore_browser_usd
        decimal runtime_usd
        decimal infra_usd
        decimal total_usd
        json by_step
    }

    PolicyBundle {
        string bundle_id PK
        string customer_id FK
        string name
        string version "semver"
        string content_uri "S3 path"
        datetime created_at
    }
```

## Entity descriptions

### Customer
A BFSI institution that licenses BackOfficePilot. Tier governs feature access.

### Workflow
A versioned YAML definition of an automation procedure. One per (customer, workflow type).
Multiple workflow types per customer (reconciliation, KYC, ...).

### Agent
A deployed instance of a workflow running on AgentCore Runtime. Multiple versions exist over
time; Registry holds the version history. At any moment there is one active version per
workflow.

### Run
A single execution. Has cost, audit, and exception artifacts attached.

### Plan
The Reasoner's output at the start of a run. Immutable once created; replans become new Plans
linked to the parent run.

### Step
A unit of work dispatched to the Executor. Sequenced within a run.

### Exception
A condition the agent flagged as unable to resolve deterministically. Always tied to a step.

### Escalation
The routing of an exception to a human reviewer. Carries SLA timer.

### AuditEvent
Append-only record of every state change in a run. Hash-chained.

### CostMeter
Per-run cost rollup. Updated as the run progresses.

### PolicyBundle
A versioned bundle of markdown files representing the customer's rules. Loaded into the
Reasoner's cached prefix.

## Persistence map

| Entity | Primary store | Reason |
|---|---|---|
| Customer, Workflow, Agent | DynamoDB | Hot path; structured access. |
| Run, Plan, Step | DynamoDB (state) + S3 (run bundle, immutable copy) | DDB for live status, S3 for archive/audit. |
| Exception, Escalation | DynamoDB | Hot path; queue semantics. |
| AuditEvent | S3 + Object Lock (WORM) | Tamper evidence; never updated. |
| CostMeter | DynamoDB + Athena view on Observability events | Live + historical reconciliation. |
| PolicyBundle | S3 + DynamoDB pointer | Bundle in S3, metadata in DDB. |

## Single-table DDB design

DynamoDB uses single-table design with composite keys:

| Access pattern | PK | SK |
|---|---|---|
| Get customer | `CUSTOMER#{id}` | `META` |
| List workflows for customer | `CUSTOMER#{id}` | `WORKFLOW#{wf_id}` |
| Get workflow | `WORKFLOW#{wf_id}` | `META` |
| List agents for workflow | `WORKFLOW#{wf_id}` | `AGENT#{agent_id}` |
| Get run | `RUN#{run_id}` | `META` |
| List steps for run | `RUN#{run_id}` | `STEP#{seq}` |
| List exceptions for run | `RUN#{run_id}` | `EXC#{seq}` |
| Open escalations for customer | `ESCQ#{customer_id}` (GSI) | `OPEN#{opened_at}` |

GSI for cross-cutting queries (open escalations, runs by customer over time window).

## Mutability rules

- Customer, Workflow, Agent: mutable through versioned updates. History via Registry (for Agent)
  and version field (for Workflow).
- Run, Plan, Step: status field is mutable in DDB. Run bundle in S3 is immutable.
- Exception, Escalation: status mutable in DDB; resolution captured as a new AuditEvent.
- AuditEvent: immutable. Never updated, never deleted.
- CostMeter: live tally is mutable in DDB; final value at run end is immutable in S3 run bundle.
