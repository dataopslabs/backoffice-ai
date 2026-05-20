---
id: DESIGN-C4-002
title: C4 Level 2 — Containers
type: design
status: approved
owner: founder
depends-on: [DESIGN-C4-001, ADR-005]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# C4 Level 2 — Containers

The container view shows the deployable units inside BackOfficePilot. Each box is a separate
process or service.

```mermaid
flowchart TB
    %% Users
    User([Customer Users]):::person
    Operator([BackOfficePilot Operator]):::person

    %% Frontend
    subgraph FE[Frontend]
      WebApp[Next.js 14 App Router<br/>Dashboard · Cost Meter · Exception Queue]:::container
    end

    %% Control plane
    subgraph CP[Control Plane]
      APIGw[API Gateway + Lambda<br/>FastAPI control plane]:::container
      Scheduler[EventBridge Scheduler<br/>Workflow cron triggers]:::container
    end

    %% Agent runtime
    subgraph AR[AgentCore Runtime]
      Orchestrator[Orchestrator Agent<br/>Plan / Execute / Reconcile / Commit]:::container
    end

    %% Tools and stores
    subgraph TS[Tools and Stores]
      Gateway[AgentCore Gateway<br/>Customer system tool surface]:::container
      Memory[AgentCore Memory<br/>Session · project context]:::container
      Registry[AgentCore Registry<br/>Agent versions]:::container
      DDB[(DynamoDB<br/>Run state · cost rollups)]:::store
      S3[(S3 + Object Lock<br/>Run bundles · audit logs)]:::store
      OS[(OpenSearch<br/>Run history index)]:::store
      Secrets[(Secrets Manager<br/>Customer creds)]:::store
      KMS[(KMS<br/>Customer-managed keys)]:::store
    end

    %% External
    Bedrock[(Bedrock<br/>Claude Opus 4.6)]:::external
    Nova[(Nova Act)]:::external
    Browser[(AgentCore Browser)]:::external
    Obs[(AgentCore Observability<br/>→ CloudWatch GenAI)]:::external
    Customer[(Customer Systems)]:::external

    %% Relationships
    User --> WebApp
    Operator --> WebApp
    WebApp -->|HTTPS<br/>OpenAPI client| APIGw
    APIGw -->|read run state| DDB
    APIGw -->|sign read URL for| S3
    Scheduler -->|invoke| Orchestrator

    Orchestrator -->|invokes for planning,<br/>reconcile, narration| Bedrock
    Orchestrator -->|reads context| Memory
    Orchestrator -->|writes context| Memory
    Orchestrator -->|loads version| Registry
    Orchestrator -->|dispatch tool calls| Gateway
    Gateway -->|primary executor| Nova
    Gateway -->|fallback executor| Browser
    Nova -->|drives| Customer
    Browser -->|drives| Customer

    Orchestrator -->|persists run state| DDB
    Orchestrator -->|writes run bundle| S3
    Orchestrator -->|emits traces| Obs
    Orchestrator -->|reads secrets| Secrets

    DDB -.encrypted by.-> KMS
    S3 -.encrypted by.-> KMS

    classDef person fill:#1E2761,stroke:#0B1F4D,color:#fff,stroke-width:2px
    classDef container fill:#F9C846,stroke:#0B1F4D,color:#0B1F4D,stroke-width:1px
    classDef external fill:#CADCFC,stroke:#0B1F4D,color:#0B1F4D,stroke-width:1px
    classDef store fill:#FFFFFF,stroke:#0B1F4D,color:#0B1F4D,stroke-dasharray:4 2,stroke-width:1px
```

## Containers

| Container | Tech | Responsibility |
|---|---|---|
| **Next.js 14 Dashboard** | Next.js App Router + TS + Tailwind + shadcn/ui | Customer-facing UI: cost meter, exception queue, run history, audit replay. |
| **API Gateway + Lambda (control plane)** | FastAPI on Lambda behind API Gateway | Reads run state, signs S3 URLs, manages workflow definitions, exposes the OpenAPI surface to the dashboard. |
| **EventBridge Scheduler** | EventBridge cron rules | Triggers workflow runs on schedule (e.g. daily reconciliation at 07:00 CT). |
| **Orchestrator Agent** | Python (FastAPI / pure orchestration loop) on AgentCore Runtime | Runs the PERC loop: plan, execute, reconcile, commit. One instance per customer per workflow. |
| **AgentCore Gateway** | AWS managed | Translates customer APIs / MCP servers / Nova Act / AgentCore Browser into agent-callable tools. |
| **AgentCore Memory** | AWS managed | Per-session and long-term memory store the orchestrator reads/writes between steps. |
| **AgentCore Registry** | AWS managed | Source of truth for orchestrator version; supports canary and rollback. |
| **DynamoDB** | AWS managed (single-table design) | Run state, cost rollups, escalation timers. Hot-path reads. |
| **S3 + Object Lock** | AWS managed | Immutable run bundles, audit logs, screenshots. |
| **OpenSearch** | AWS managed | Search across run history (customer-facing search). |
| **Secrets Manager** | AWS managed | Customer-system credentials. Rotated. |
| **KMS** | AWS managed | Customer-managed keys for at-rest encryption. |
| **External: Bedrock** | AWS managed | Hosts Claude Opus 4.6. |
| **External: Nova Act** | AWS managed | Primary Executor. |
| **External: AgentCore Browser** | AWS managed | Fallback Executor. |
| **External: Observability → CloudWatch** | AWS managed | Trace and metric pipeline. |

## Deployment boundaries

- Every container above lives inside a **single AWS account per environment** (`ADR-009`).
- Prod and UAT are mirror deployments, distinct accounts, distinct VPCs.
- The Next.js dashboard runs on Vercel or AWS Amplify; treat it as the only resource outside the
  VPC. It reaches the control plane through API Gateway with Cognito/JWT auth.

## Hot paths

- **Workflow run**: Scheduler → Orchestrator → (Bedrock + Gateway + Memory + Customer Systems) → S3 + DDB + Observability.
- **Dashboard load**: User → Next.js → API Gateway → Lambda → DDB → S3 (signed URLs for artifacts).
- **Exception escalation**: Orchestrator → Slack/Teams (via Gateway/Executor) → human → API Gateway → DDB update → Orchestrator resumes.

## Why the control plane is separate from the agent runtime

The control plane is a stateless CRUD-style API. Lambda + API Gateway is ideal for it. The
agent runtime is a long-running orchestration loop with side-effects. AgentCore Runtime is ideal
for it. Mixing them in one container would force one to compromise. The boundary is intentional.
