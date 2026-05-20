---
id: ADR-004
title: Multi-model orchestration — Claude Opus 4.6 (Reasoner) + Amazon Nova Act (Executor)
type: adr
status: approved
owner: founder
depends-on: [ADR-002]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-004: Multi-model orchestration

## Context

A back-office automation workflow includes ~5–15 genuine reasoning decisions (interpret policy,
classify exception) and 50–500 UI actions (click, type, download). A single foundation model
asked to do both will be either too expensive (reasoner doing keystrokes at $25/M output tokens)
or too unreliable (UI agent making policy decisions without depth).

The right partition is along the seam where reasoning and execution naturally cleave.

## Decision

Use **two models** behind two ports:

- **`ReasonerPort`** implemented by **`BedrockOpusReasoner`** — Claude Opus 4.6 via AWS Bedrock.
  Plans workflows, classifies exceptions, writes audit narration.
- **`ExecutorPort`** implemented by **`NovaActExecutor`** (primary) and
  **`AgentCoreBrowserExecutor`** (fallback). Performs UI actions.

A deterministic **orchestrator** mediates between them via a Plan → Execute → Reconcile → Commit
loop. Schemas in `34-api-contract/` define the handoff payloads.

## Alternatives considered

- **Single Opus loop calling tool functions directly**: works for demo; cost and latency fail in
  production.
- **Single Nova Act with embedded reasoning**: Nova Act's planning model is tuned for UI
  decomposition, not policy interpretation; the demo lies about the depth of reasoning.
- **Three or more models** (e.g. Sonnet for routine classification, Opus for hard cases): a
  desirable v2, blocked by complexity in v1. Left as a future optimization with the routing layer
  pluggable behind `ReasonerPort`.

## Consequences

**Positive**

- Cost: ~$0.55 per reconciliation run end-to-end (vs ~$3 for a single-model approach).
- Latency: parallel execution while reasoning is happening, no model bottleneck on UI steps.
- Reliability: deterministic UI execution where it matters, probabilistic reasoning where it
  helps.
- Extensible: each port has one or more adapters; future models or providers slot in without
  domain changes.

**Negative**

- The orchestrator becomes the most important piece of code. Tested heavily.
- Two SDK dependencies (Anthropic-via-Bedrock and Nova Act) to track for breaking changes.
- Schema validation overhead on every handoff (intentional, not regrettable).

## Mitigations

- Hexagonal layout (`ADR-002`) ensures both adapters can be swapped or supplemented.
- The orchestrator's PERC loop is implemented as a state machine with explicit transitions and
  cost ceilings (`NFR-COST-001`).
- Property-based tests (hypothesis) exercise the orchestrator over generated plan/result
  sequences.
