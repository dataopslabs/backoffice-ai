---
id: DESIGN-C4-004
title: C4 Level 3 — Reasoner (Bedrock Opus) adapter
type: design
status: approved
owner: founder
depends-on: [DESIGN-C4-003, ADR-004]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# C4 Level 3 — Reasoner adapter

The `BedrockOpusReasoner` is the only implementation of `ReasonerPort` in v1. This view shows
how it composes the prompt, manages cache, and validates output.

```mermaid
flowchart LR
    PERC[PERCLoop] -->|plan(req)<br/>or reconcile(req)| Adapter

    subgraph Adapter[BedrockOpusReasoner]
      direction TB
      PromptBuilder[PromptBuilder]:::comp
      CacheCtl[PromptCacheController<br/>Bedrock prompt cache]:::comp
      Invoker[BedrockInvoker<br/>boto3 / anthropic SDK]:::comp
      SchemaValidator[JSONSchemaValidator]:::comp
      RetryPolicy[RetryPolicy<br/>schema-aware retry, max 1]:::comp
      TokenAcct[TokenAccountant]:::comp

      PromptBuilder --> CacheCtl
      CacheCtl --> Invoker
      Invoker --> SchemaValidator
      SchemaValidator --> RetryPolicy
      RetryPolicy --> Invoker
      Invoker --> TokenAcct
    end

    Adapter -->|plan JSON<br/>or reconcile JSON| PERC
    Adapter -->|emit BedrockInvocationCompleted| Bus[EventBusPort]
    Adapter -->|emit ReasonerTokensUsed| Bus

    classDef comp fill:#0D9488,stroke:#0B1F4D,color:#fff
```

## Components inside the adapter

| Component | Responsibility |
|---|---|
| **PromptBuilder** | Assembles the prompt from the workflow definition, the customer policy bundle, the current state, and the request mode (plan vs reconcile). Uses string templates from `prompts/`. |
| **PromptCacheController** | Marks the long-lived prefix (workflow definition + policy bundle) as a cache breakpoint so Bedrock serves it at 10% input cost. Refreshes on workflow definition version bump. |
| **BedrockInvoker** | Calls `bedrock-runtime:InvokeModel` (or the Anthropic SDK with Bedrock backend). Handles streaming where applicable. |
| **JSONSchemaValidator** | Validates output against the schema in `34-api-contract/schemas/plan.schema.json` or `reconcile.schema.json`. Failure triggers retry. |
| **RetryPolicy** | One retry on schema-validation failure (with an explicit "your previous output was invalid because X" prefix). No retry on transport errors — let them propagate. |
| **TokenAccountant** | Records input/output tokens (and which fraction was cached) and emits events for the cost meter (`ADR-012`). |

## Calling conventions

Two methods on `ReasonerPort`:

```python
class ReasonerPort(ABC):
    @abstractmethod
    async def plan(self, request: PlanRequest) -> Plan: ...

    @abstractmethod
    async def reconcile(self, request: ReconcileRequest) -> ReconcileDecision: ...
```

`PlanRequest` and `ReconcileRequest` are Pydantic models in `src/domain/ports.py`. Schemas
exposed in `34-api-contract/schemas/`.

## Prompt strategy

- **System prompt**: a short, stable string describing the Reasoner role. Version-pinned per
  workflow.
- **Cached prefix**: customer policy bundle + workflow definition. ~5–20K tokens. Marked
  cacheable in the Bedrock request.
- **Per-call suffix**: current state, step result, instructions. ~500–2,000 tokens.
- **Output format**: strictly JSON, schema-enforced. We never parse free-form English.

## Failure modes

| Failure | Detection | Response |
|---|---|---|
| Bedrock 5xx | boto3 exception | Propagate; orchestrator pauses run; retried on next scheduler trigger or operator action. |
| Bedrock 4xx (auth, quota) | boto3 exception | Page on-call. Quota issues are billing problems. |
| Output not JSON | `JSONSchemaValidator` parse error | Retry once with "previous output not JSON" hint. Second failure: escalate run. |
| Output JSON but schema-invalid | `JSONSchemaValidator` schema fail | Retry once with diff hint. Second failure: escalate run. |
| Output low confidence | `confidence` field in plan/decision | Domain-layer rule: low confidence → escalate, not retry. |

## Cost characteristics (modeling assumptions)

- Plan call: ~8K input (mostly cached), ~2K output. ~$0.05.
- Reconcile call: ~6K input (mostly cached), ~1K output. ~$0.03.
- Typical run: 1 plan + ~5 reconciles = ~$0.20 in Reasoner cost.

Recorded per call via `TokenAccountant`; published to cost meter on `BedrockInvocationCompleted`
event. Pricing source: `config/pricing.yaml`.

## Test surface

- **Unit**: PromptBuilder, SchemaValidator pure-function tests.
- **Integration (recorded)**: `vcr.py` cassettes capture real Bedrock responses for deterministic
  reruns.
- **Live (gated)**: a separate suite that hits Bedrock for real. Runs nightly, not in PR CI.
