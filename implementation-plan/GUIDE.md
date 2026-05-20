# BackOfficePilot — Developer Implementation Guide

> **Purpose:** This is the single document a developer needs to understand, set up, and build the entire BackOfficePilot platform end-to-end.

---

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [AWS Architecture Diagrams](#2-aws-architecture-diagrams)
3. [Technology Stack](#3-technology-stack)
4. [Prerequisites & Environment Setup](#4-prerequisites--environment-setup)
5. [Repository Structure](#5-repository-structure)
6. [Step-by-Step Build Guide](#6-step-by-step-build-guide)
7. [Development Methodology](#7-development-methodology)
8. [API Contract](#8-api-contract)
9. [Data Model](#9-data-model)
10. [Key Design Patterns](#10-key-design-patterns)
11. [Deployment & CI/CD](#11-deployment--cicd)
12. [Testing Strategy](#12-testing-strategy)
13. [Quick Reference](#13-quick-reference)

---

## 1. Project Overview

### What is BackOfficePilot?

A **multi-model agentic automation platform** for BFSI (Banking, Financial Services, Insurance) back-office. It automates repetitive, rule-heavy tasks that today require humans to navigate legacy ERPs, banking portals, and lending CRMs.

### The Two-Model Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR (PERC Loop)                          │
│              Plan → Execute → Reconcile → Commit                    │
├─────────────────────────────┬───────────────────────────────────────┤
│     REASONER                │         EXECUTOR                      │
│     Claude Opus 4.6         │         Amazon Nova Act (primary)     │
│     via Amazon Bedrock      │         AgentCore Browser (fallback)  │
│                             │                                       │
│  • Plans workflows          │  • Drives browser UIs                 │
│  • Interprets policy        │  • Navigates legacy ERPs              │
│  • Classifies exceptions    │  • Downloads/uploads files            │
│  • Writes audit narrative   │  • Fills forms, clicks buttons        │
└─────────────────────────────┴───────────────────────────────────────┘
```

### First Use Case: Invoice Reconciliation

Daily 3-way match: download bank transactions → export open invoices from NetSuite → match using policy rules → post journal entries → escalate exceptions to humans via Slack.

**Budget:** < $1.00 per run, < 15 minutes duration.

---

## 2. AWS Architecture Diagrams

### 2.1 High-Level System Context

```
                    ┌──────────────────────────────────────┐
                    │         BackOfficePilot               │
                    │   Multi-Model Agent Platform          │
                    └──────────┬───────────────────────────┘
                               │
        ┌──────────────────────┼──────────────────────────┐
        │                      │                          │
        ▼                      ▼                          ▼
┌───────────────┐    ┌─────────────────┐    ┌─────────────────────┐
│ Amazon Bedrock│    │ Amazon Nova Act  │    │ AWS AgentCore       │
│ Claude Opus   │    │ (UI Automation) │    │ • Runtime           │
│ (Reasoning)   │    │                 │    │ • Gateway           │
└───────────────┘    └─────────────────┘    │ • Memory            │
                                            │ • Registry          │
        ┌───────────────────────────────┐   │ • Browser           │
        │ Customer Systems              │   │ • Observability     │
        │ • NetSuite / Sage (ERP)       │   └─────────────────────┘
        │ • Banking Portals             │
        │ • Lending CRMs                │           │
        │ • Slack / Teams / Email       │           ▼
        └───────────────────────────────┘   ┌─────────────────────┐
                                            │ CloudWatch GenAI    │
                                            │ Observability       │
                                            └─────────────────────┘
```

### 2.2 AWS Account & VPC Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        AWS Organization (Single Payer)                    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  PRODUCTION ACCOUNT (backofficepilot-prod)                       │    │
│  │                                                                   │    │
│  │  VPC: 10.10.0.0/16 (us-east-1)                                  │    │
│  │  ┌─────────────────────────────────────────────────────────┐     │    │
│  │  │ Public Subnets                                           │     │    │
│  │  │   • API Gateway / ALB                                    │     │    │
│  │  │   • NAT Gateways                                         │     │    │
│  │  ├─────────────────────────────────────────────────────────┤     │    │
│  │  │ Private Subnets                                          │     │    │
│  │  │   • Lambda (Control Plane API)                           │     │    │
│  │  │   • Fargate (Admin Jobs)                                 │     │    │
│  │  ├─────────────────────────────────────────────────────────┤     │    │
│  │  │ Isolated Subnets (NO internet egress)                    │     │    │
│  │  │   • AgentCore Runtime Fleet                              │     │    │
│  │  │   • AgentCore Gateway Endpoints                          │     │    │
│  │  │   • VPC Endpoints for AWS Services                       │     │    │
│  │  └─────────────────────────────────────────────────────────┘     │    │
│  │                                                                   │    │
│  │  Data Stores (account-local):                                    │    │
│  │    • DynamoDB (single-table, on-demand)                          │    │
│  │    • S3 + Object Lock (audit, run bundles)                       │    │
│  │    • S3 Versioned (policy bundles, workflows)                    │    │
│  │    • OpenSearch Serverless (run history search)                   │    │
│  │    • Secrets Manager (customer credentials)                      │    │
│  │    • KMS CMK (per-environment + BYOK for enterprise)             │    │
│  │    • AgentCore Memory (session + long-term context)              │    │
│  │    • AgentCore Registry (agent versions)                         │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  UAT ACCOUNT (backofficepilot-uat)                               │    │
│  │  VPC: 10.20.0.0/16 (us-east-1) — Mirror of Prod                 │    │
│  │  Same stack structure, different account, NO shared data          │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │  CI ACCOUNT (backofficepilot-ci)                                 │    │
│  │  • CDK Deploy Role (cross-account assume into Prod/UAT)          │    │
│  │  • S3 Build Artifacts                                            │    │
│  │  • GitHub Actions Runner                                         │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  ⚠️  NO VPC Peering, NO Transit Gateway between Prod and UAT            │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.3 Container Architecture (What Runs Where)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         RUNTIME ARCHITECTURE                              │
│                                                                          │
│  ┌──────────────┐     HTTPS/JWT      ┌──────────────────────────┐       │
│  │  Next.js 14  │ ──────────────────► │  API Gateway + Lambda    │       │
│  │  Dashboard   │                     │  (FastAPI Control Plane)  │       │
│  │  (Vercel /   │ ◄── SSE ────────── │                          │       │
│  │   Amplify)   │                     │  Endpoints:              │       │
│  └──────────────┘                     │  • /customers/*/runs     │       │
│        ▲                              │  • /runs/*/steps         │       │
│        │ Cognito JWT                  │  • /escalations          │       │
│        │                              │  • /policy-bundles       │       │
│  ┌─────┴────────┐                    └───────────┬──────────────┘       │
│  │ Amazon       │                                │                       │
│  │ Cognito      │                                │ Read/Write             │
│  │ (User Pool)  │                                ▼                       │
│  └──────────────┘                    ┌──────────────────────────┐       │
│                                      │  DynamoDB + S3            │       │
│  ┌──────────────────────────────┐    │  (Run state, Audit,      │       │
│  │  EventBridge Scheduler       │    │   Cost, Escalations)     │       │
│  │  (Cron triggers)             │    └──────────────────────────┘       │
│  └──────────────┬───────────────┘                ▲                       │
│                 │ Invoke                          │                       │
│                 ▼                                 │                       │
│  ┌──────────────────────────────────────────────────────────────┐       │
│  │              AGENTCORE RUNTIME (Orchestrator Agent)            │       │
│  │                                                                │       │
│  │  ┌────────────────────────────────────────────────────────┐   │       │
│  │  │  PERC Loop (Plan → Execute → Reconcile → Commit)       │   │       │
│  │  │                                                         │   │       │
│  │  │  ┌─────────────┐  ┌──────────────┐  ┌──────────────┐  │   │       │
│  │  │  │ Bedrock     │  │ Nova Act     │  │ AgentCore    │  │   │       │
│  │  │  │ Opus 4.6    │  │ Executor     │  │ Browser      │  │   │       │
│  │  │  │ (Reasoner)  │  │ (Primary)    │  │ (Fallback)   │  │   │       │
│  │  │  └─────────────┘  └──────────────┘  └──────────────┘  │   │       │
│  │  └────────────────────────────────────────────────────────┘   │       │
│  │                                                                │       │
│  │  Emits: Domain Events → EventBridge → Subscribers             │       │
│  │    • AuditWriter (S3 hash chain)                              │       │
│  │    • CostMeterSubscriber (pricing.yaml)                       │       │
│  │    • EscalationOpener (Slack/Teams/Email)                     │       │
│  └──────────────────────────────────────────────────────────────┘       │
│                                                                          │
│  ┌──────────────────────────────────────────────────────────────┐       │
│  │  AgentCore Observability → CloudWatch GenAI Observability     │       │
│  │  (Traces, Metrics, Per-agent cost visibility)                 │       │
│  └──────────────────────────────────────────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────────┘
```

### 2.4 Data Flow — Single Reconciliation Run

```
┌─────────┐    Cron/Manual     ┌──────────────┐
│EventBridge│ ────────────────► │ Orchestrator │
│Scheduler │                    │ (PERC Loop)  │
└─────────┘                    └──────┬───────┘
                                      │
                    ┌─────────────────┼─────────────────┐
                    │                 │                  │
                    ▼                 ▼                  ▼
            ┌──────────────┐  ┌────────────┐  ┌──────────────┐
            │ 1. PLAN      │  │ 2. EXECUTE │  │ 3. RECONCILE │
            │ Claude Opus  │  │ Nova Act   │  │ Claude Opus  │
            │ reads policy │  │ drives UI  │  │ checks match │
            │ produces plan│  │ on ERP     │  │ vs policy    │
            └──────────────┘  └────────────┘  └──────────────┘
                                                      │
                              ┌────────────────────────┤
                              │                        │
                    ┌─────────▼──────┐      ┌─────────▼──────┐
                    │ Match OK       │      │ Exception      │
                    │ → Post journal │      │ → Escalate     │
                    │ → Commit       │      │ → Slack notify │
                    └────────────────┘      └────────────────┘
                              │                        │
                              ▼                        ▼
                    ┌────────────────────────────────────────┐
                    │ 4. COMMIT                               │
                    │ • Audit event (hash-chained → S3 WORM) │
                    │ • Cost meter update (DDB)               │
                    │ • Run bundle (S3)                       │
                    │ • Domain events (EventBridge)           │
                    └────────────────────────────────────────┘
```

### 2.5 Tenant Isolation Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    TENANT ISOLATION MODEL                         │
│                                                                   │
│  Customer A (JWT: customer_id=acme)                              │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ DDB:     PK = CUSTOMER#acme, RUN#acme_*                  │    │
│  │ S3:      s3://bop-prod-runs/cust=acme/*                  │    │
│  │ Secrets: cust/acme/netsuite/creds                        │    │
│  │ KMS:     alias/bop-prod-cust-acme                        │    │
│  │ Memory:  bop/prod/cust/acme                              │    │
│  │ Cognito: App Client for acme                             │    │
│  │ IAM:     PerCustomerIamRole(customer_id="acme")          │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  Customer B (JWT: customer_id=globex)                            │
│  ┌─────────────────────────────────────────────────────────┐    │
│  │ DDB:     PK = CUSTOMER#globex, RUN#globex_*              │    │
│  │ S3:      s3://bop-prod-runs/cust=globex/*                │    │
│  │ Secrets: cust/globex/bank_portal/creds                   │    │
│  │ KMS:     alias/bop-prod-cust-globex                      │    │
│  │ Memory:  bop/prod/cust/globex                            │    │
│  │ Cognito: App Client for globex                           │    │
│  │ IAM:     PerCustomerIamRole(customer_id="globex")        │    │
│  └─────────────────────────────────────────────────────────┘    │
│                                                                   │
│  ⚠️  IAM denies cross-prefix access + CloudTrail evidence        │
│  ⚠️  JWT customer_id claim enforced on every API call            │
└─────────────────────────────────────────────────────────────────┘
```


### 2.6 Orchestrator Internal Architecture (Hexagonal)

```
┌─────────────────────────────────────────────────────────────────────────┐
│                    ORCHESTRATOR (AgentCore Runtime)                       │
│                                                                          │
│  ┌─── src/api/ ─────────────────────────────────────────────────────┐   │
│  │  WorkflowRunController (thin FastAPI router, returns 202)         │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│  ┌─── src/application/ ─────────────────────────────────────────────┐   │
│  │  RunWorkflowUseCase → PERCLoop → ExecutorRouter                   │   │
│  │                                                                    │   │
│  │  Between each step:                                               │   │
│  │    ✓ Check abort signal (DDB poll)                                │   │
│  │    ✓ Check cost ceiling (CostCeilingEnforcer)                     │   │
│  │    ✓ Emit domain events                                           │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│  ┌─── src/domain/ (PURE — zero AWS imports) ────────────────────────┐   │
│  │  Models: Workflow, Plan, Step, Run, Exception, Escalation         │   │
│  │  Logic:  ExceptionClassifier, PolicyApplier, CostCeilingEnforcer  │   │
│  │  Events: PlanCreated, StepExecuted, ExceptionRaised, RunCompleted │   │
│  │  Ports:  ReasonerPort, ExecutorPort, MemoryPort, EventBusPort,    │   │
│  │          AuditLogPort, SecretPort                                  │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                              │                                           │
│  ┌─── src/adapters/ ────────────────────────────────────────────────┐   │
│  │  bedrock/opus_reasoner.py      → implements ReasonerPort          │   │
│  │  novaact/executor.py           → implements ExecutorPort          │   │
│  │  agentcore/browser_executor.py → implements ExecutorPort          │   │
│  │  agentcore/memory_adapter.py   → implements MemoryPort            │   │
│  │  agentcore/event_bridge_bus.py → implements EventBusPort          │   │
│  │  audit/s3_hash_chain.py        → implements AuditLogPort          │   │
│  │  secrets/secrets_manager.py    → implements SecretPort             │   │
│  │  slack/escalation_adapter.py   → channel adapter                  │   │
│  └───────────────────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## 3. Technology Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| **Frontend** | Next.js 14 App Router, TypeScript, Tailwind CSS, shadcn/ui | SSR + RSC for fast first paint, type safety, accessible components |
| **Backend** | Python 3.12, FastAPI, Pydantic v2 | Async, OpenAPI-native, strong validation |
| **Package Mgmt** | uv (Python), pnpm (frontend) | Fast, deterministic |
| **BDD** | pytest-bdd | Integrates with pytest ecosystem |
| **Testing** | pytest, hypothesis (property-based), Playwright (E2E) | Comprehensive coverage |
| **Infrastructure** | AWS CDK (Python) | Same language as backend, type-safe IaC |
| **Observability** | OpenTelemetry SDK → AgentCore Observability → CloudWatch | End-to-end tracing |
| **AI Models** | Claude Opus 4.6 (Bedrock), Amazon Nova Act | Reasoning + UI execution |
| **Runtime** | AWS Bedrock AgentCore (6 services) | Purpose-built agent hosting |

> ⚠️ **Locked stack.** Changing any of the above requires writing an ADR in `specs/30-architecture/32-adrs/`.

---

## 4. Prerequisites & Environment Setup

### 4.1 Required Tools

```bash
# Python 3.12+
python3 --version  # must be 3.12+

# uv (Python package manager)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Node.js 20+ and pnpm
node --version     # must be 20+
npm install -g pnpm

# AWS CLI v2
aws --version

# AWS CDK
npm install -g aws-cdk

# Git
git --version
```

### 4.2 AWS Account Setup

You need three AWS accounts under an Organization:

| Account | Purpose | Engineer Access |
|---------|---------|----------------|
| `backofficepilot-ci` | CI/CD, deploy role | Full |
| `backofficepilot-uat` | Development & testing | Read/Write |
| `backofficepilot-prod` | Customer production | Just-in-time only |

**For MVP development, start with UAT account only.**

### 4.3 AWS Services to Enable

In your UAT account, ensure these services are available:
- Amazon Bedrock (enable Claude Opus 4.6 model access)
- Amazon Nova Act (request access)
- AWS Bedrock AgentCore (Runtime, Gateway, Memory, Registry, Browser, Observability)
- DynamoDB, S3, Lambda, API Gateway, EventBridge
- Cognito, Secrets Manager, KMS
- OpenSearch Serverless
- CloudWatch

### 4.4 Local Development Setup

```bash
# Clone the repo
git clone <repo-url> backofficepilot
cd backofficepilot

# Python environment
uv venv
source .venv/bin/activate
uv pip install -e ".[dev]"

# Frontend
cd web
pnpm install
cd ..

# AWS credentials (configure for UAT account)
aws configure --profile bop-uat
export AWS_PROFILE=bop-uat

# Verify
make test    # should pass (or show "no tests" initially)
make lint    # should pass
```

---

## 5. Repository Structure

```
backofficepilot/
├── specs/                          ← SOURCE OF TRUTH (read these first!)
│   ├── 00-meta/                    ← Glossary, naming conventions, traceability
│   ├── 10-product-bop/
│   │   ├── 11-requirements/        ← REQ-PRD-001 through REQ-PRD-016
│   │   ├── 12-design/              ← DESIGN-A2-001 through DESIGN-A2-008
│   │   ├── 13-stories/             ← STORY-A3-* (MVP) and STORY-A4-* (Phase 2)
│   │   └── 14-landing/             ← (future)
│   ├── 20-usecase-recon/           ← Reconciliation use-case specifics
│   ├── 30-architecture/
│   │   ├── 31-c4-diagrams/         ← System context, container, component views
│   │   ├── 32-adrs/                ← ADR-001 through ADR-012
│   │   ├── 33-data-model/          ← Core entities, storage strategy
│   │   ├── 34-api-contract/        ← openapi.yaml + schemas
│   │   └── 35-event-catalog/       ← Domain event definitions
│   └── 40-non-functional/          ← NFR-SEC-*, NFR-PERF-*, NFR-COST-*, etc.
│
├── features/                       ← BDD .feature files (one per story)
│   └── BDD-A3-010.feature          ← Example: PERC loop scenarios
│
├── tests/                          ← pytest + pytest-bdd (mirrors src/)
│   ├── conftest.py
│   ├── domain/                     ← Pure domain model tests
│   ├── application/                ← Use case + orchestrator tests
│   └── adapters/                   ← Adapter integration tests
│
├── src/                            ← Python backend
│   ├── domain/                     ← PURE models + logic (NO AWS imports)
│   │   ├── workflow.py
│   │   ├── plan.py
│   │   ├── step.py
│   │   ├── run.py
│   │   ├── exception_classifier.py
│   │   ├── escalation.py
│   │   ├── audit_event.py
│   │   ├── cost_meter.py
│   │   ├── cost_ceiling.py
│   │   ├── policy_applier.py
│   │   ├── events.py               ← Domain event definitions
│   │   └── ports.py                ← Port ABCs (interfaces)
│   ├── application/                ← Orchestration layer
│   │   ├── run_workflow_use_case.py
│   │   ├── perc_loop.py
│   │   └── executor_router.py
│   ├── adapters/                   ← External integrations
│   │   ├── bedrock/opus_reasoner.py
│   │   ├── novaact/executor.py
│   │   ├── agentcore/browser_executor.py
│   │   ├── agentcore/memory_adapter.py
│   │   ├── agentcore/event_bridge_bus.py
│   │   ├── audit/s3_hash_chain.py
│   │   ├── secrets/secrets_manager_adapter.py
│   │   ├── slack/escalation_adapter.py
│   │   └── cost/cost_meter_subscriber.py
│   └── api/                        ← FastAPI routers (thin)
│       ├── main.py
│       ├── middleware/auth.py
│       ├── middleware/tenant_context.py
│       ├── routers/runs.py
│       ├── routers/workflows.py
│       └── routers/escalations.py
│
├── web/                            ← Next.js 14 frontend
│   ├── app/
│   │   ├── layout.tsx
│   │   ├── (auth)/login/page.tsx
│   │   ├── (customer)/
│   │   │   ├── layout.tsx
│   │   │   ├── page.tsx            ← Run list (dashboard home)
│   │   │   ├── runs/[runId]/page.tsx
│   │   │   ├── escalations/page.tsx
│   │   │   ├── cost/page.tsx
│   │   │   └── workflows/page.tsx
│   │   └── (operator)/
│   │       └── customers/page.tsx
│   ├── components/
│   │   ├── ui/                     ← shadcn/ui primitives
│   │   ├── runs/                   ← Run-specific components
│   │   ├── escalations/            ← Escalation components
│   │   └── charts/                 ← Recharts wrappers
│   ├── lib/
│   │   ├── _generated/             ← OpenAPI TypeScript client (DO NOT EDIT)
│   │   ├── auth.ts
│   │   └── sse.ts
│   ├── middleware.ts
│   ├── tailwind.config.ts
│   └── package.json
│
├── workflows/                      ← Workflow YAML definitions
│   └── daily_reconciliation.yaml
│
├── policy/                         ← Customer policy markdown bundles
│   └── acme/
│       ├── index.yaml
│       ├── three_way_match.md
│       └── journal_posting_rules.md
│
├── config/
│   └── pricing.yaml                ← Cost meter pricing source-of-truth
│
├── infra/                          ← AWS CDK (Python)
│   ├── app.py
│   ├── stacks/
│   │   ├── network_stack.py
│   │   ├── data_stack.py
│   │   ├── security_stack.py
│   │   ├── agentcore_stack.py
│   │   └── auth_stack.py
│   └── constructs/
│       └── per_customer_iam_role.py
│
├── scripts/
│   ├── build_traceability.py       ← Generates traceability matrix
│   ├── generate_client.sh          ← OpenAPI → TypeScript client
│   ├── verify_audit.py             ← Audit chain verification
│   └── onboard_customer.py         ← Customer provisioning
│
├── Makefile                        ← Common commands
├── pyproject.toml                  ← Python project config
└── .github/workflows/ci.yml        ← CI pipeline
```


---

## 6. Step-by-Step Build Guide

### Phase 1: MVP (3 Weeks — 16 Stories)

Follow this exact order. Each step depends on the ones before it.

---

### WEEK 1 — Foundation & Infrastructure

#### Step 1: Bootstrap Repo (STORY-A3-001) — Day 1

```bash
# 1. Initialize Python project
mkdir -p src/{domain,application,adapters,api} tests/{domain,application,adapters}
uv init
# Edit pyproject.toml:
#   - python = ">=3.12"
#   - dependencies: fastapi, pydantic>=2.0, uvicorn, mangum, boto3
#   - dev-dependencies: pytest, pytest-bdd, hypothesis, ruff, mypy, playwright

# 2. Initialize frontend
cd web && pnpm create next-app@14 . --typescript --tailwind --app --src-dir=no
pnpm add @tanstack/react-query recharts react-hook-form zod
cd ..

# 3. Create Makefile
cat > Makefile << 'EOF'
.PHONY: test lint codegen traceability

test:
	uv run pytest tests/ -v

lint:
	uv run ruff check src/ tests/
	uv run mypy src/
	cd web && pnpm lint

codegen:
	scripts/generate_client.sh

traceability:
	uv run python scripts/build_traceability.py
EOF

# 4. Create CI pipeline
mkdir -p .github/workflows
# Create ci.yml with: lint → test → cdk synth → codegen verify

# 5. Create pre-commit config
# .pre-commit-config.yaml with ruff, mypy, prettier hooks
```

**Verify:** `make test` runs, `make lint` passes on empty project.

---

#### Step 2: CDK Infrastructure (STORY-A3-002) — Days 2-3

```bash
mkdir -p infra/{stacks,constructs}
cd infra && uv init && uv add aws-cdk-lib constructs
```

Create these CDK stacks:

| Stack | Resources |
|-------|-----------|
| `network_stack.py` | VPC (10.20.0.0/16), 3 AZs, public/private/isolated subnets, NAT GW |
| `data_stack.py` | DynamoDB (single-table, on-demand), S3 buckets (runs, audit+ObjectLock, policy), OpenSearch Serverless |
| `security_stack.py` | KMS CMK, `PerCustomerIamRole` construct, Secrets Manager paths |
| `agentcore_stack.py` | AgentCore Runtime, Gateway, Memory provisioning, EventBridge rules |
| `auth_stack.py` | Cognito User Pool, app client, domain |

```bash
# Deploy to UAT
cd infra
cdk synth
cdk deploy --all --profile bop-uat
```

**Verify:** `cdk synth` produces valid CloudFormation. DDB table, S3 buckets, Cognito pool exist in UAT.

---

#### Step 3: FastAPI Control Plane (STORY-A3-003) — Days 3-4

```bash
# Create the OpenAPI spec first (spec-first approach)
# File: specs/30-architecture/34-api-contract/openapi.yaml

# Then implement FastAPI app
touch src/api/main.py
touch src/api/middleware/{__init__,auth,tenant_context}.py
touch src/api/routers/{__init__,runs,workflows,escalations}.py
```

Key endpoints to implement:

```python
# src/api/main.py
app = FastAPI(title="BackOfficePilot", version="0.1.0")

# Middleware chain:
# 1. JWT validation (Cognito issuer, audience, expiry)
# 2. Tenant context extraction (customer_id from JWT claim)
# 3. Request logging

# Routes:
# GET  /customers/{id}/runs          → listRuns (paginated, filterable)
# GET  /runs/{id}                    → getRun
# GET  /runs/{id}/steps              → listSteps
# GET  /runs/{id}/audit              → getRunAudit
# GET  /runs/{id}/cost               → getRunCost
# POST /customers/{id}/workflows/{wf}/runs → startRun (returns 202)
# POST /runs/{id}/abort              → abortRun
# PUT  /customers/{id}/workflows/{wf}     → upsertWorkflow
# GET  /customers/{id}/escalations   → listEscalations
# POST /escalations/{id}/resolve     → resolveEscalation
```

Generate TypeScript client:
```bash
# scripts/generate_client.sh
npx openapi-typescript specs/30-architecture/34-api-contract/openapi.yaml \
  -o web/lib/_generated/api.d.ts
```

**Verify:** OpenAPI spec validates. TypeScript client generates. JWT middleware rejects invalid tokens with 401.

---

#### Step 4: Next.js Dashboard Scaffold (STORY-A3-004) — Days 4-5

```bash
cd web
# Install shadcn/ui
pnpm dlx shadcn-ui@latest init
# Add components: button, card, table, badge, dialog, tabs, skeleton

# Configure brand palette in tailwind.config.ts:
# navy: "#0B1F4D", ice: "#CADCFC", gold: "#F9C846", teal: "#0D9488", coral: "#F96167"
```

Create the App Router structure:
```
web/app/
├── layout.tsx                    ← Shell + TanStack Query provider
├── (auth)/login/page.tsx         ← Cognito hosted UI redirect
├── (customer)/
│   ├── layout.tsx                ← Customer nav + auth guard
│   ├── page.tsx                  ← Dashboard home (run list)
│   ├── runs/[runId]/page.tsx     ← Run detail
│   ├── escalations/page.tsx      ← Exception queue
│   ├── cost/page.tsx             ← Cost charts
│   └── workflows/page.tsx        ← Workflow management
└── (operator)/
    └── customers/page.tsx        ← Operator view
```

**Verify:** Login flow works with Cognito. Auth routes redirect unauthenticated users. Brand palette applied.

---

#### Step 5: Domain Models (STORY-A3-005) — Day 5

```bash
touch src/domain/{workflow,plan,step,run,exception_classifier,escalation,audit_event,cost_meter,cost_ceiling,policy_applier,events,ports}.py
touch tests/domain/{test_models,test_status_transitions}.py
```

Key rules:
- **ZERO AWS imports** in `src/domain/`
- All models are Pydantic v2 `BaseModel`
- Status transitions enforced (e.g., `Run` cannot go from `succeeded` → `running`)
- Port interfaces are ABCs with no implementation
- Property-based tests with hypothesis

```python
# src/domain/ports.py — Example
from abc import ABC, abstractmethod

class ReasonerPort(ABC):
    @abstractmethod
    async def plan(self, workflow, policy_bundle) -> Plan: ...
    @abstractmethod
    async def reconcile(self, step_result, policy) -> ReconcileDecision: ...

class ExecutorPort(ABC):
    @abstractmethod
    async def execute(self, step, credentials) -> StepResult: ...
```

**Verify:** `pytest tests/domain/ -v` passes. Hypothesis finds no invariant violations.

---

### WEEK 2 — Core Orchestration

#### Step 6: Workflow Validator (STORY-A3-006) — Day 6

Implement the validation pipeline:
1. YAML parse (PyYAML safe loader)
2. JSON Schema validate
3. Pydantic model construction
4. Cross-field validation (system refs, step refs, policy doc existence)

**Verify:** Invalid YAML returns structured error list. Valid YAML produces `WorkflowDefinition`.

---

#### Step 7: Bedrock Opus Reasoner (STORY-A3-007) — Days 7-8

```python
# src/adapters/bedrock/opus_reasoner.py
class BedrockOpusReasoner(ReasonerPort):
    async def plan(self, workflow, policy_bundle) -> Plan:
        # Load policy bundle as cached prompt prefix
        # Call Claude Opus 4.6 via Bedrock with structured output
        # Return Plan model with steps + reasoning trace
        # Track token usage for cost meter

    async def reconcile(self, step_result, policy) -> ReconcileDecision:
        # Evaluate step result against policy rules
        # Return confidence score + decision (match/exception/retry)
```

**Verify:** Plan generation produces valid `Plan`. Reconcile returns confidence + decision. Token counts reported.

---

#### Step 8: Nova Act Executor (STORY-A3-008) — Days 8-9

```python
# src/adapters/novaact/executor.py
class NovaActExecutor(ExecutorPort):
    async def execute(self, step, credentials) -> StepResult:
        # Navigate to system URL
        # Authenticate with stored credentials
        # Execute intent-based actions
        # Capture screenshots
        # Return observations + actions taken
```

**Verify:** Executor navigates URLs, performs actions, captures screenshots, handles timeouts.

---

#### Step 9: AgentCore Browser Fallback (STORY-A3-009) — Day 9

Same `ExecutorPort` interface, uses AgentCore Browser (sandboxed Playwright). Implement `ExecutorRouter` that tries Nova Act first, falls back to Browser on failure.

**Verify:** Fallback triggers after Nova Act failure. Same interface, same result format.

---

#### Step 10: PERC Loop (STORY-A3-010) — Days 9-11 ⭐ CRITICAL PATH

```python
# src/application/perc_loop.py
class PERCLoop:
    async def run(self, workflow, plan) -> RunResult:
        for step in plan.steps:
            # CHECK: abort signal (DDB poll)
            if await self._is_aborted(run_id): break

            # CHECK: cost ceiling
            if self.cost_ceiling.is_breached(): break

            # EXECUTE
            result = await self.executor_router.execute(step)

            # RECONCILE
            decision = await self.reasoner.reconcile(result, policy)

            if decision.is_exception:
                exception = self.classifier.classify(decision, step)
                await self.event_bus.emit(ExceptionRaised(exception))
                if step.retry_count >= max_retries:
                    await self._escalate(exception)
                    break

            # EMIT events
            await self.event_bus.emit(StepExecuted(step, result))

        # COMMIT
        await self.event_bus.emit(RunCompleted(run))
```

**Verify:** 5-step workflow completes. Retry logic works. Cost ceiling aborts. Abort signal honored. Hypothesis finds no invariant violations.

---

### WEEK 3 — Subscribers & Dashboard

#### Step 11: Audit Writer (STORY-A3-011) — Day 12

```python
# src/adapters/audit/s3_hash_chain.py
class S3HashChainAuditLog(AuditLogPort):
    def write(self, event, prev_hash) -> AuditEvent:
        payload = canonicalize(event)  # sorted keys, UTF-8, ISO-8601
        this_hash = sha256(f"{prev_hash}\n{payload}".encode()).hexdigest()
        # Write to DDB (PK=RUN#{run_id}, SK=AUDIT#{seq:08d})
        # Write to S3 with Object Lock retention
        return AuditEvent(prev_hash=prev_hash, this_hash=this_hash, ...)
```

Also create `scripts/verify_audit.py` for independent chain verification.

**Verify:** Hash chain verifiable. S3 objects have Object Lock. Verification script works.

---

#### Step 12: Cost Meter (STORY-A3-012) — Day 12

Subscribe to domain events, compute cost from `config/pricing.yaml`, maintain per-run tally, emit `CostMetered` events, enforce ceiling.

**Verify:** Cost computed correctly from token counts. Pricing version recorded. Daily rollups work.

---

#### Step 13: Slack Escalation (STORY-A3-013) — Day 13

Post escalation messages to Slack with: narrative, step context, screenshots, Approve/Reject buttons.

**Verify:** Messages posted. Buttons resolve escalations via API.

---

#### Step 14: Dashboard — Run Pages (STORY-A3-014) — Days 13-14

Build run list (paginated, filterable) and run detail (timeline, audit, cost, bundle download, SSE for live runs).

**Verify:** Server-rendered first paint. Filters work without reload. SSE streams live updates.

---

#### Step 15: Dashboard — Exceptions & Cost (STORY-A3-015) — Days 14-15

Build exception queue (severity-grouped, inline approve/reject/edit) and cost view (Recharts stacked bar, savings panel, CSV export).

**Verify:** Severity ordering correct. Inline actions work. Charts render with brand palette.

---

#### Step 16: Traceability Generator (STORY-A3-016) — Day 15

`scripts/build_traceability.py` — walks specs, features, tests; cross-references; generates matrix; exits non-zero on orphans.

**Verify:** `make traceability` produces matrix. Orphan detection works.

---

## 7. Development Methodology

### The Iron Rule: SPEC → BDD → TDD → CODE

```
1. READ the spec     → Find story in specs/10-product-bop/13-stories/
2. WRITE BDD         → Create features/BDD-{story-id}.feature from ACs
3. WRITE tests       → Create tests/{layer}/test_{name}.py (must be RED)
4. IMPLEMENT         → Minimum code to turn tests GREEN
5. REFACTOR          → Clean up without breaking tests
6. UPDATE spec       → Set status: done, bump version, set last-updated
7. UPDATE matrix     → Run `make traceability`
```

### What You Must NEVER Do

- ❌ Write code before the failing test exists
- ❌ Put AWS imports in `src/domain/`
- ❌ Edit generated OpenAPI client code by hand
- ❌ Mix domain logic and adapter logic in the same module
- ❌ Skip the ADR when making a non-trivial architectural choice
- ❌ Invent a spec ID that doesn't exist in `/specs`

---

## 8. API Contract

### Resource Model

```
/customers/{customerId}/workflows                    GET, PUT
/customers/{customerId}/workflows/{workflowId}/runs  POST (start run)
/runs/{runId}                                        GET
/runs/{runId}/steps                                  GET
/runs/{runId}/audit                                  GET
/runs/{runId}/cost                                   GET
/runs/{runId}/abort                                  POST
/runs/{runId}/stream                                 SSE (live updates)
/customers/{customerId}/escalations                  GET
/customers/{customerId}/escalations/stream           SSE (live queue)
/escalations/{escalationId}/resolve                  POST
/policy-bundles/{bundleId}                           GET, PUT
```

### Conventions

- **Auth:** JWT via Cognito. Scopes: `customer:read`, `customer:write`, `operator`
- **Pagination:** Cursor-based (`?cursor=...`)
- **Errors:** RFC 9457 Problem Details
- **Streaming:** SSE (server → client only)
- **Idempotency:** `Idempotency-Key` header on writes
- **Long-running:** `202 Accepted` + `Location` header for async operations

### Code Generation

```bash
make codegen
# Produces:
#   web/lib/_generated/api.d.ts      (TypeScript types)
#   web/lib/_generated/client.ts     (TypeScript client)
```

---

## 9. Data Model

### Core Entities

| Entity | Store | Description |
|--------|-------|-------------|
| Customer | DDB | BFSI institution (tier: pilot/platform/enterprise) |
| Workflow | DDB | Versioned YAML automation definition |
| Agent | DDB | Deployed instance on AgentCore Runtime |
| Run | DDB + S3 | Single workflow execution |
| Plan | DDB + S3 | Reasoner's step-by-step plan |
| Step | DDB + S3 | Unit of executor work |
| Exception | DDB | Condition agent cannot resolve |
| Escalation | DDB | Routing to human reviewer + SLA |
| AuditEvent | S3 (WORM) | Hash-chained state transition |
| CostMeter | DDB + Athena | Per-run cost breakdown |
| PolicyBundle | S3 + DDB | Versioned markdown rules |

### DynamoDB Single-Table Design

| Access Pattern | PK | SK |
|---------------|-----|-----|
| Get customer | `CUSTOMER#{id}` | `META` |
| List workflows | `CUSTOMER#{id}` | `WORKFLOW#{wf_id}` |
| Get run | `RUN#{run_id}` | `META` |
| List steps | `RUN#{run_id}` | `STEP#{seq}` |
| Audit events | `RUN#{run_id}` | `AUDIT#{seq:08d}` |
| Open escalations (GSI) | `ESCQ#{customer_id}` | `OPEN#{opened_at}` |

---

## 10. Key Design Patterns

### Hexagonal Architecture

```
Domain (pure) ← Ports (interfaces) ← Adapters (AWS implementations)
```

- Domain has NO external dependencies
- Ports are abstract base classes
- Adapters implement ports with real AWS services
- Swap adapters for testing (mock adapters)

### Event-Driven Internal Bus

Every state transition emits a domain event:
```
PlanCreated → StepExecuted → ExceptionRaised → EscalationOpened
RunCompleted → CostMetered → SlaBreached → RunAborted
```

Subscribers (audit writer, cost meter, escalation opener) are independent.

### Snapshot Semantics

When a run starts, workflow definition + policy bundle hashes are recorded. The run executes against that snapshot forever, even if config changes mid-run.

### Cost Ceiling Enforcement

```
Soft ceiling: workflow.budget.max_cost_per_run_usd (default $1.00)
Hard ceiling: 2× soft (kill switch)
Check: between every step in the PERC loop
```

---

## 11. Deployment & CI/CD

### Promotion Flow

```
local dev → push → CI (lint + test + cdk synth)
                        ↓
                   cdk deploy --env uat
                        ↓
                   manual approval gate
                        ↓
                   cdk deploy --env prod
```

### Agent Rollout (via AgentCore Registry)

```
CI → pushVersion(image, git_sha) → Registry
Operator → startCanary(10% traffic) → monitor 1 hour
         → promoteToFull(100%) OR rollback(<60 seconds)
```

### Key Commands

```bash
# Local development
make test                    # Run all tests
make lint                    # Ruff + mypy + tsc
make codegen                 # Regenerate TypeScript client
make traceability            # Rebuild traceability matrix

# Infrastructure
cd infra && cdk synth        # Validate CloudFormation
cd infra && cdk deploy --all # Deploy to current profile's account

# Frontend
cd web && pnpm dev           # Local dev server
cd web && pnpm build         # Production build
```

---

## 12. Testing Strategy

| Layer | Tool | What to Test |
|-------|------|-------------|
| Domain models | pytest + hypothesis | Status transitions, invariants, property-based |
| Application (PERC loop) | pytest + hypothesis | Orchestration logic, retry, abort, ceiling |
| Adapters | pytest + moto/localstack | AWS integration, error handling |
| API | pytest + httpx | Endpoint behavior, auth, tenant isolation |
| BDD | pytest-bdd | Acceptance criteria from specs |
| Frontend | Playwright | E2E flows, accessibility (axe-core) |

### Running Tests

```bash
# All tests
make test

# Specific layer
uv run pytest tests/domain/ -v
uv run pytest tests/application/ -v
uv run pytest tests/adapters/ -v

# BDD only
uv run pytest tests/ -m bdd

# Frontend E2E
cd web && pnpm test:e2e
```

---

## 13. Quick Reference

### Reading Order for New Developers

1. **This file** (GUIDE.md) — overview and build steps
2. `specs/CLAUDE.md` — engineering discipline and rules
3. `specs/00-meta/00-glossary.md` — terminology
4. `specs/30-architecture/31-c4-diagrams/DESIGN-C4-001-context.md` — system context
5. `specs/30-architecture/31-c4-diagrams/DESIGN-C4-003-component-orchestrator.md` — core component
6. `specs/10-product-bop/13-stories/EPIC-A-001-mvp-platform-foundation.md` — MVP scope
7. Pick your first story from `specs/10-product-bop/13-stories/STORY-A3-*`

### Brand Palette (for frontend)

| Name | Hex | Usage |
|------|-----|-------|
| Navy | `#0B1F4D` | Primary text, headers |
| Ice | `#CADCFC` | Backgrounds, cards |
| Gold | `#F9C846` | Accents, highlights |
| Teal | `#0D9488` | Success states, CTAs |
| Coral | `#F96167` | Errors, alerts, SLA breach |

### Environment Variables

```bash
# Required for backend
AWS_REGION=us-east-1
AWS_PROFILE=bop-uat
COGNITO_USER_POOL_ID=<from CDK output>
COGNITO_CLIENT_ID=<from CDK output>
DDB_TABLE_NAME=bop-uat-main
AUDIT_BUCKET=bop-uat-audit
RUNS_BUCKET=bop-uat-runs
EVENTBRIDGE_BUS_NAME=bop-uat-events

# Required for frontend
NEXT_PUBLIC_API_URL=https://<api-gw-id>.execute-api.us-east-1.amazonaws.com
NEXT_PUBLIC_COGNITO_DOMAIN=<cognito-domain>.auth.us-east-1.amazoncognito.com
NEXT_PUBLIC_COGNITO_CLIENT_ID=<client-id>
```

### MVP Definition of Done

- [ ] All 16 stories `status: done`
- [ ] Reconciliation workflow runs unattended in UAT for 5 consecutive business days
- [ ] p95 run-start latency under 5 seconds
- [ ] Per-run cost under $1.00
- [ ] Dashboard renders: run list, run detail, exception queue, cost view
- [ ] CI green; traceability matrix shows ≥ 90% REQ coverage
- [ ] Demo video recorded (90 seconds, one full run)

---

## Appendix: AWS Services Used

| Service | Purpose | Cost Driver |
|---------|---------|-------------|
| Amazon Bedrock (Claude Opus 4.6) | Reasoning, planning, classification | Token usage (~$0.55/run) |
| Amazon Nova Act | UI automation on customer systems | Agent-hour billing |
| AgentCore Runtime | Hosts orchestrator agents | vCPU + memory hours |
| AgentCore Gateway | Tool surface for agents | Request count |
| AgentCore Memory | Session + long-term context | Storage + requests |
| AgentCore Registry | Agent version management | Free (management plane) |
| AgentCore Browser | Fallback UI executor | Session hours |
| AgentCore Observability | Traces + metrics | Ingestion volume |
| DynamoDB | Hot state (runs, escalations, cost) | Read/write capacity |
| S3 + Object Lock | Immutable audit, run bundles | Storage + requests |
| OpenSearch Serverless | Run history search | OCU hours |
| Lambda | Control plane API | Invocations + duration |
| API Gateway | HTTPS endpoint | Requests |
| EventBridge | Event routing + scheduling | Events published |
| Cognito | Authentication | MAU |
| Secrets Manager | Customer credentials | Secrets stored + API calls |
| KMS | Encryption keys | Key usage |
| CloudWatch | Logs + metrics | Ingestion + storage |

**Estimated monthly cost per customer (pilot scale):** ~$143 infrastructure + ~$0.55 × runs/day inference.
