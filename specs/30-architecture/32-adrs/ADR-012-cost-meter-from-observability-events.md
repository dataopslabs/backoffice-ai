---
id: ADR-012
title: Cost meter is derived from AgentCore Observability events, not from billing
type: adr
status: approved
owner: founder
depends-on: [ADR-005, ADR-006]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-012: Cost meter derived from Observability events

## Context

The product promises per-agent and per-workflow cost visibility in the customer dashboard. AWS
billing data has multi-hour latency, is account-scoped (not agent-scoped), and is not designed
to support a near-real-time customer-facing display.

AgentCore Observability emits per-invocation events that include token counts (for Bedrock
invocations) and agent-hour fractions (for AgentCore Runtime and Nova Act). These can be
multiplied by published rates to produce a synthetic cost.

## Decision

Build a **cost meter** subscriber on the internal event bus (`ADR-006`) that:

1. Consumes `BedrockInvocationCompleted`, `NovaActSessionTicked`, `AgentCoreBrowserUsed`,
   `AgentRuntimeTicked` (event names normalized in `35-event-catalog/`).
2. Multiplies counters by current pricing (held in `config/pricing.yaml`, versioned).
3. Writes per-run, per-workflow, per-agent, per-customer rollups to DynamoDB.
4. Emits `CostMetered` events on threshold breaches (per `NFR-COST-001` ceilings).
5. Reconciles weekly against actual AWS Cost & Usage Reports to detect drift, with alarms on
   >5% variance.

Customer dashboard reads the rollups directly.

## Alternatives considered

- **Bill-based**: customer sees real billing data. Latency unacceptable; not agent-scoped.
- **Tag-and-bill with CloudWatch metrics**: too coarse; doesn't isolate per-workflow within an
  agent.
- **Manual estimates**: trivially wrong over time.

## Consequences

**Positive**

- Near-real-time cost feedback to customers and to the cost-ceiling enforcer.
- Per-agent / per-workflow attribution is first-class.
- Pricing changes (e.g. Bedrock model price drop) are a `config/pricing.yaml` edit; no code
  change needed.

**Negative**

- Synthetic cost diverges from actual AWS bill by some margin. We track it.
- Pricing source-of-truth lives in two places (AWS pricing pages and `pricing.yaml`). Update
  hygiene matters.
- When AWS introduces a new billing dimension we haven't modeled, our number is low.

## Mitigations

- Weekly reconciliation script flags drift > 5%.
- `pricing.yaml` versioned; cost data tagged with the pricing version used.
- Customer-facing display shows the cost as "BackOfficePilot estimate" with a fine-print link to
  the methodology page.

## NFR linkage

This ADR is the technical backbone for:

- `NFR-COST-001` — per-run cost ceiling.
- `NFR-OBS-002` — agent-level cost visibility.
- `NFR-OBS-003` — workflow-level cost rollup in dashboard.
