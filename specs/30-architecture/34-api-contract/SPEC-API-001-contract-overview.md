---
id: SPEC-API-001
title: API contract overview
type: spec
status: approved
owner: founder
depends-on: [ADR-008]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# API contract overview

The authoritative API contract lives next to this file in `openapi.yaml`. This document
explains its shape, conventions, and how it is consumed.

## Surface

The control-plane API serves two readers:

1. **The Next.js dashboard** — customer-facing UI.
2. **Operator tooling** — internal scripts for customer onboarding, workflow deployment, support.

The same OpenAPI document drives both. Authorization scopes distinguish customer-readable from
operator-only endpoints.

## Resource model

```
/customers
/customers/{customerId}
/customers/{customerId}/workflows
/customers/{customerId}/workflows/{workflowId}
/customers/{customerId}/workflows/{workflowId}/agents
/customers/{customerId}/workflows/{workflowId}/runs
/runs/{runId}
/runs/{runId}/steps
/runs/{runId}/audit
/runs/{runId}/cost
/runs/{runId}/bundle           ← signed S3 URL
/escalations
/escalations/{escalationId}
/escalations/{escalationId}/resolve
/policy-bundles
/policy-bundles/{bundleId}
```

## Conventions

- **Versioning**: URL path is unversioned; OpenAPI document has an `info.version`. Backward-
  compatible additions bump the patch; breaking changes coordinate via a new top-level path
  segment (`/v2/`) and an ADR.
- **Auth**: JWT via Amazon Cognito; scopes `customer:read`, `customer:write`, `operator`.
- **Pagination**: cursor-based. Response includes `nextCursor`; clients pass it back as
  `?cursor=...`.
- **Filtering**: simple query params. Complex filters use a documented `filter` query param with
  a small grammar.
- **Errors**: RFC 9457 Problem Details. Every error has a stable `type` URI under
  `https://backofficepilot.ai/problems/`.
- **Idempotency**: write operations accept an `Idempotency-Key` header.
- **Long-running**: "start run" returns `202 Accepted` with a `Location` header pointing to the
  run resource. Clients poll or subscribe via SSE on `/runs/{runId}/stream`.

## Streaming

The dashboard subscribes to two SSE endpoints:

- `/runs/{runId}/stream` — live run updates.
- `/customers/{customerId}/escalations/stream` — live escalation queue.

SSE chosen over WebSocket for simplicity; one-way server→client suffices.

## OperationId convention

Each operation has a stable `operationId` in `camelCase`:

- `listCustomers`, `getCustomer`, `createCustomer`
- `listWorkflows`, `getWorkflow`, `upsertWorkflow`
- `listRuns`, `getRun`, `startRun`, `abortRun`
- `listSteps`, `getStep`
- `getRunAudit`, `getRunCost`, `getRunBundle`
- `listEscalations`, `resolveEscalation`
- `listPolicyBundles`, `getPolicyBundle`, `upsertPolicyBundle`

Used as method names in the generated TypeScript client (e.g. `client.listRuns(...)`).

## Schemas

All schemas live in `openapi.yaml#/components/schemas` (or referenced from
`schemas/*.json`). Key schemas:

- `Customer`, `Workflow`, `Agent`, `Run`, `Step`, `Exception`, `Escalation`, `AuditEvent`,
  `CostMeter`, `PolicyBundle` — match the entities in `DESIGN-DATA-001`.
- `Plan`, `StepResult` — the Reasoner/Executor handoff shapes (also referenced by the agent
  runtime internally).
- `Problem` — RFC 9457 error envelope.
- `CursorPage<T>` — generic pagination wrapper.

## Code generation

```bash
make codegen
# Runs:
#   datamodel-codegen --input openapi.yaml --output src/api/_generated/models.py
#   openapi-typescript openapi.yaml -o web/lib/_generated/api.d.ts
#   openapi-typescript-client openapi.yaml -o web/lib/_generated/client.ts
```

Generated paths under `_generated/` are gitignored from manual edits but committed for review.
`make verify-codegen` fails if running `make codegen` produces a diff.

## What is not in the API

- **Agent runtime internals**: the orchestrator's Reasoner ↔ Executor handoffs are not part of
  the public API. They have JSON Schemas (`34-api-contract/schemas/`) but are invoked internally,
  not exposed.
- **Direct DynamoDB or S3 access**: the dashboard never reads DDB directly; everything flows
  through this API.
- **Tool definitions**: AgentCore Gateway exposes tools to the orchestrator. Those tools have
  their own schemas, separate from this customer-facing API.

## OpenAPI file expectations

The file `openapi.yaml` is hand-authored. It MUST:

- Pass `spectral lint` (config in `.spectral.yaml`) with zero errors.
- Validate against the OpenAPI 3.1 schema.
- Round-trip cleanly through codegen (no warnings).
- Carry an `info.version` that is bumped per change.
- Include examples for every schema.
