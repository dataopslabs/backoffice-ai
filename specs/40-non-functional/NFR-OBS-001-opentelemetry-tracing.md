---
id: NFR-OBS-001
title: OpenTelemetry tracing on every action
type: nfr
status: approved
owner: founder
depends-on: [ADR-005]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-OBS-001 — OpenTelemetry tracing

## Statement

Every domain event and every external call **shall** carry a W3C trace context propagated
end-to-end (API → Lambda → orchestrator → Bedrock → Nova Act → AgentCore Memory) so a single
`trace_id` correlates the entire run.

## Implementation

- OpenTelemetry Python SDK auto-instruments FastAPI, boto3, and the Anthropic SDK.
- Trace exporter sends spans to AgentCore Observability.
- Trace IDs are included in every event envelope (`SPEC-EVT-001`).
- Customer-facing run detail page links from a step to its trace view in CloudWatch GenAI
  Observability (operator-only link).

## Spans of interest

| Span | Captured attributes |
|---|---|
| `run.start` | customer_id, workflow_id, workflow_version, trigger |
| `plan.call` | model_id, input_tokens, output_tokens, cached_fraction, latency |
| `step.dispatch` | step_id, executor, system, intent_summary |
| `step.execute` | duration, observations_summary, screenshots_uri |
| `reconcile.call` | decision, confidence, latency |
| `escalation.open` | category, severity, channel |
| `run.commit` | total_cost_usd, bundle_uri, status |

## Verification

- Trace sampling 100% in UAT, 100% in Prod for v1 (revisit after load profile).
- Synthetic Canary asserts `trace_id` end-to-end correlation per run.
