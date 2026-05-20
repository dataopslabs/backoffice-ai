---
id: DESIGN-C4-005
title: C4 Level 3 — Executor (Nova Act + AgentCore Browser) adapters
type: design
status: approved
owner: founder
depends-on: [DESIGN-C4-003, ADR-011]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# C4 Level 3 — Executor adapters

The `ExecutorPort` has two implementations: `NovaActExecutor` (primary) and
`AgentCoreBrowserExecutor` (fallback). They expose the same interface; the `ExecutorRouter`
picks one per call.

```mermaid
flowchart TB
    PERC[PERCLoop] -->|execute(step)| Router[ExecutorRouter]
    Router -->|route| NovaA
    Router -->|route| BrowserA

    subgraph NovaA[NovaActExecutor]
      direction TB
      NA_Translate[StepTranslator<br/>step → Nova Act intent]:::comp
      NA_Session[SessionManager<br/>warm contexts, 2FA]:::comp
      NA_Invoke[NovaActInvoker<br/>SDK call]:::comp
      NA_Capture[ResultCapture<br/>screenshots, observed values]:::comp
      NA_Timer[AgentHourTimer]:::comp

      NA_Translate --> NA_Session
      NA_Session --> NA_Invoke
      NA_Invoke --> NA_Capture
      NA_Invoke --> NA_Timer
    end

    subgraph BrowserA[AgentCoreBrowserExecutor]
      direction TB
      B_Translate[StepTranslator<br/>step → playbook]:::comp
      B_Sandbox[BrowserSandbox<br/>AgentCore Browser session]:::comp
      B_Driver[BrowserDriver<br/>actions]:::comp
      B_Capture[ResultCapture]:::comp

      B_Translate --> B_Sandbox
      B_Sandbox --> B_Driver
      B_Driver --> B_Capture
    end

    NovaA -->|StepResult| PERC
    BrowserA -->|StepResult| PERC

    NovaA -->|emit NovaActSessionTicked| Bus[EventBusPort]
    BrowserA -->|emit AgentCoreBrowserUsed| Bus

    classDef comp fill:#0D9488,stroke:#0B1F4D,color:#fff
```

## Common port

```python
class ExecutorPort(ABC):
    @abstractmethod
    async def execute(self, step: Step, ctx: ExecutionContext) -> StepResult: ...

    @abstractmethod
    def capabilities(self) -> set[Capability]: ...
```

`capabilities()` is what `ExecutorRouter` consults when a workflow YAML declares
`executor_requirements`.

## Common contract

Both executors must:

- Honor the **timeout** in `step.timeout_seconds`.
- Return a `StepResult` with one of `success | failure | partial` and a structured
  `observations` dict that the Reasoner can parse on reconcile.
- Capture screenshots when the step is an escalation candidate (configured per workflow).
- Emit per-call cost events (`NovaActSessionTicked` or `AgentCoreBrowserUsed`).
- Treat the customer system as a black box; never persist customer data outside the run bundle.

## NovaActExecutor specifics

| Component | Responsibility |
|---|---|
| **StepTranslator** | Converts a `Step` into a Nova Act natural-language intent plus parameters. |
| **SessionManager** | Pools warm Nova Act sessions per customer to amortize cold start. Handles 2FA via TOTP from Secrets Manager. |
| **NovaActInvoker** | Calls the Nova Act SDK. |
| **ResultCapture** | Parses the SDK return, normalizes observed values, attaches screenshots. |
| **AgentHourTimer** | Tracks elapsed agent-hour fractions per step for cost telemetry. |

## AgentCoreBrowserExecutor specifics

| Component | Responsibility |
|---|---|
| **StepTranslator** | Converts a `Step` into a deterministic browser playbook (action list). |
| **BrowserSandbox** | Provisions an AgentCore Browser session, isolated per run. |
| **BrowserDriver** | Executes the playbook (click, type, navigate, download) via AgentCore Browser API. |
| **ResultCapture** | Same role as in Nova Act adapter. |

## Routing inputs

The router considers:

1. **Environment**: UAT defaults to `AgentCoreBrowserExecutor` for reproducibility.
2. **Nova Act health**: if a service-health probe flags degraded state, routing flips.
3. **Workflow requirements**: `executor_requirements: [...]` in YAML; pick the executor that
   advertises all required capabilities.
4. **Operator override**: forced routing via a run flag, used for debugging.

Capability examples:

- `pdf-download` — both support.
- `file-upload` — Nova Act primary; AgentCore Browser supported.
- `desktop-citrix` — Nova Act only.
- `dom-replay` — AgentCore Browser only (deterministic re-run by trace ID).

## StepResult schema (excerpt)

```json
{
  "step_id": "s2",
  "status": "success",
  "observations": {
    "exported_file_s3_uri": "s3://...",
    "row_count": 47
  },
  "actions_taken": [
    {"type": "login", "system": "netsuite"},
    {"type": "navigate", "path": "Reports/AR Aging"},
    {"type": "export_csv"}
  ],
  "duration_seconds": 14,
  "screenshots": ["s3://..."],
  "executor": "novaact",
  "cost_usd_estimate": 0.012
}
```

Full schema in `34-api-contract/schemas/step_result.schema.json`.

## Why two executors

Pure resilience and dev velocity. AgentCore Browser is deterministic enough that PR-CI can
exercise full workflows headlessly without Nova Act credit consumption. Nova Act is the
production-grade story we sell to customers. Both behind one port (`ADR-011`).
