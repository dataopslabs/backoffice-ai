---
id: DESIGN-C4-003
title: C4 Level 3 — Orchestrator components
type: design
status: approved
owner: founder
depends-on: [DESIGN-C4-002, ADR-002, ADR-004, ADR-006]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# C4 Level 3 — Orchestrator components

The orchestrator is the most important container in BackOfficePilot. This view shows its
internal components and how they collaborate. Hexagonal architecture (`ADR-002`) is visible
here as a ring: domain in the center, application around it, adapters at the edge.

```mermaid
flowchart TB
    subgraph Orchestrator[Orchestrator Agent — AgentCore Runtime]
      direction TB

      subgraph API[api/ — FastAPI surface]
        WFController[WorkflowRunController]:::api
      end

      subgraph App[application/ — use cases]
        RunUseCase[RunWorkflowUseCase]:::app
        PERCLoop[PERCLoop<br/>Plan · Execute · Reconcile · Commit]:::app
        ExecutorRouter[ExecutorRouter]:::app
      end

      subgraph Domain[domain/ — pure logic]
        Workflow[Workflow model]:::dom
        Plan[Plan model]:::dom
        Step[Step model]:::dom
        ExceptionCls[ExceptionClassifier]:::dom
        Policy[PolicyApplier]:::dom
        CostCeiling[CostCeilingEnforcer]:::dom
      end

      subgraph Ports[ports]
        RP[/ReasonerPort/]:::port
        EP[/ExecutorPort/]:::port
        MP[/MemoryPort/]:::port
        BP[/EventBusPort/]:::port
        AP[/AuditLogPort/]:::port
        SecP[/SecretPort/]:::port
      end

      subgraph Adapters[adapters/]
        BedrockA[BedrockOpusReasoner]:::adp
        NovaA[NovaActExecutor]:::adp
        BrowserA[AgentCoreBrowserExecutor]:::adp
        MemoryA[AgentCoreMemoryAdapter]:::adp
        EventA[EventBridgeBus + InProcessBus]:::adp
        AuditA[S3HashChainAuditLog]:::adp
        SecretsA[SecretsManagerAdapter]:::adp
      end
    end

    WFController --> RunUseCase
    RunUseCase --> PERCLoop
    PERCLoop --> ExecutorRouter
    PERCLoop --> ExceptionCls
    PERCLoop --> Policy
    PERCLoop --> CostCeiling

    PERCLoop -.uses.-> RP
    PERCLoop -.uses.-> MP
    PERCLoop -.uses.-> BP
    PERCLoop -.uses.-> AP
    ExecutorRouter -.uses.-> EP
    ExecutorRouter -.uses.-> SecP

    RP -.implements.-> BedrockA
    EP -.implements.-> NovaA
    EP -.implements.-> BrowserA
    MP -.implements.-> MemoryA
    BP -.implements.-> EventA
    AP -.implements.-> AuditA
    SecP -.implements.-> SecretsA

    classDef api fill:#1E2761,stroke:#0B1F4D,color:#fff
    classDef app fill:#F9C846,stroke:#0B1F4D,color:#0B1F4D
    classDef dom fill:#CADCFC,stroke:#0B1F4D,color:#0B1F4D
    classDef port fill:#fff,stroke:#0B1F4D,stroke-dasharray:4 2,color:#0B1F4D
    classDef adp fill:#0D9488,stroke:#0B1F4D,color:#fff
```

## Components

### API layer

| Component | Responsibility |
|---|---|
| **WorkflowRunController** | Thin FastAPI router. Receives a "start run" command from EventBridge Scheduler. Delegates to the use case. Returns 202 immediately; the run is async. |

### Application layer

| Component | Responsibility |
|---|---|
| **RunWorkflowUseCase** | Orchestrates a single workflow execution end-to-end. Loads the workflow definition, picks the executor, invokes the PERC loop, ensures commit. |
| **PERCLoop** | The Plan → Execute → Reconcile → Commit state machine. Calls `ReasonerPort` for plan and reconcile; calls `ExecutorPort` for execute; emits domain events at every transition; enforces cost ceiling per iteration. |
| **ExecutorRouter** | Decides which executor to call (`NovaActExecutor` primary, `AgentCoreBrowserExecutor` fallback per `ADR-011`). |

### Domain layer

| Component | Responsibility |
|---|---|
| **Workflow** | Pure model representing the YAML-defined workflow. Validates structure. |
| **Plan** | Pure model representing a Reasoner output: list of steps, success criteria, reasoning trace. |
| **Step** | Pure model for a unit of executor work. |
| **ExceptionClassifier** | Pure logic to bucket exceptions by type and severity. Takes Reasoner output + step result; produces an `Exception` value. |
| **PolicyApplier** | Resolves a policy bundle against a domain decision (e.g. "is this match within tolerance?"). |
| **CostCeilingEnforcer** | Tracks accumulated cost in the current run; signals abort when the per-run cap is exceeded. |

### Ports

| Port | Implementations |
|---|---|
| `ReasonerPort` | `BedrockOpusReasoner` |
| `ExecutorPort` | `NovaActExecutor`, `AgentCoreBrowserExecutor` |
| `MemoryPort` | `AgentCoreMemoryAdapter` |
| `EventBusPort` | `InProcessEventBus` (default), `EventBridgeBus` (fanout) |
| `AuditLogPort` | `S3HashChainAuditLog` |
| `SecretPort` | `SecretsManagerAdapter` |

## Code-level layout

```
src/
├── api/
│   └── workflow_run_controller.py
├── application/
│   ├── run_workflow_use_case.py
│   ├── perc_loop.py
│   └── executor_router.py
├── domain/
│   ├── workflow.py
│   ├── plan.py
│   ├── step.py
│   ├── exception_classifier.py
│   ├── policy_applier.py
│   ├── cost_ceiling.py
│   └── ports.py            # all Port ABCs
└── adapters/
    ├── bedrock/opus_reasoner.py
    ├── novaact/executor.py
    ├── agentcore/{browser_executor,memory_adapter,event_bridge_bus}.py
    ├── audit/s3_hash_chain.py
    └── secrets/secrets_manager_adapter.py
```

## Why this matters

Every behavior in the orchestrator can be traced to one of these components. When a story
implements a new exception category, it touches the `ExceptionClassifier` and one or two
adapters — never the API layer, never the executor adapters. When we add the AgentCore
Identity adapter later, no domain code changes. The architecture is what keeps the spec-driven
discipline tractable.
