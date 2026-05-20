---
id: ADR-005
title: AgentCore Runtime is the host for all production agents
type: adr
status: approved
owner: founder
depends-on: [ADR-004]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-005: AgentCore Runtime is the host for all production agents

## Context

Three plausible hosting models exist for the BackOfficePilot orchestrator and its long-running
agent processes:

1. AWS Lambda + Step Functions (the obvious "build it yourself" answer in 2024).
2. ECS Fargate tasks coordinated by EventBridge.
3. AWS Bedrock **AgentCore Runtime**, which is purpose-built for hosting agents.

AgentCore Runtime provides: identity-aware execution, integration with AgentCore Gateway for
tools, integration with AgentCore Memory for session state, native trace emission to AgentCore
Observability, and Registry-driven version rollout.

## Decision

Deploy every agent (one per customer per workflow) as an AgentCore Runtime resource. The
orchestrator logic runs inside the agent runtime; the FastAPI control-plane surface runs on
Lambda (low-volume) or Fargate (admin operations) outside the agent runtime, in the same VPC.

## Alternatives considered

- **Lambda + Step Functions**: portable across clouds, but every AgentCore benefit (Gateway
  integration, Memory, Registry, Observability) becomes a build-it-yourself ticket.
- **Fargate everywhere**: gives more control over runtime characteristics but no Registry, no
  built-in identity, and we end up rebuilding what AgentCore already provides.
- **Mixed**: tempting but doubles the operations surface.

## Consequences

**Positive**

- Tool exposure (Gateway), session state (Memory), version rollout (Registry), and explainability
  (Observability) are first-class instead of bolted-on.
- Per-agent cost telemetry comes from Observability out of the box (`ADR-012`).
- Roll-forward/rollback semantics are managed at the Registry level — safer than blue/green on
  Fargate.

**Negative**

- AWS lock-in is structural. Multi-cloud is no longer a one-PR change.
- AgentCore is newer than Lambda; service-level behavior may evolve and surprise us.
- Some workflows (small admin tasks) are over-served by the runtime; we run those on Lambda
  outside the runtime instead.

## Boundary

- Agent code: AgentCore Runtime.
- Control plane (API for the dashboard, admin endpoints, scheduled triggers): Lambda + Fargate
  inside the same VPC, talking to the same DynamoDB / S3 stores.
- Frontend dashboard: Vercel or AWS Amplify; hits the control plane through API Gateway.

Both Prod and UAT have their own AgentCore Runtime fleet (`ADR-009`).
