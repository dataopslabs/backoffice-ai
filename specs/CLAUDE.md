# CLAUDE.md — BackOfficePilot Spec Repository

This file is loaded automatically by Claude Code at the start of every session in this repository.
Read it first; it tells you how this project is engineered and what discipline to follow.

## What is BackOfficePilot

A multi-model agentic automation platform for the BFSI (banking, financial services, insurance) back-office.
Two foundation models, by design:

- **Claude Opus 4.6** (via Amazon Bedrock) — the **Reasoner**. Plans workflows, interprets policy,
  classifies exceptions, writes audit-grade narrative.
- **Amazon Nova Act** — the **Executor**. Drives browser and desktop UIs on legacy ERPs, banking
  portals, and lending CRMs. Production-grade UI agent.

The runtime substrate is **AWS Bedrock AgentCore**:

| Service | Role in BackOfficePilot |
|---|---|
| AgentCore Runtime | Hosts every agent. Serverless. |
| AgentCore Gateway | Exposes customer APIs, internal tools, and MCP servers to agents. |
| AgentCore Memory | Session state, per-project context, long-term customer knowledge. |
| AgentCore Registry | Agent versioning, rollout, rollback. |
| AgentCore Browser | Sandboxed browser host. Used in dev/test and as a fallback executor when Nova Act cannot reach a system. |
| AgentCore Observability | Spans, traces, agent-level metrics. Surfaced in CloudWatch GenAI Observability. |

Cost telemetry (per-agent and per-workflow) flows from AgentCore Observability into the BackOfficePilot
product UI. Prod and UAT are deployed in **separate VPCs** with no shared data path.

## The engineering discipline — Spec-Driven Development

Every implementation in this repository follows this chain, in this order:

```
SPEC  →  BDD  →  TDD  →  CODE
```

1. **SPEC** — a markdown file in `/specs` defines the requirement, story, design, or NFR.
   Every spec file has a stable identifier (e.g. `STORY-B3-027`) and frontmatter that records its
   dependencies, the BDD files that cover it, and the tests that prove it.
2. **BDD** — Gherkin `.feature` files in `/features` translate spec acceptance criteria into
   executable behavior contracts. One feature file per story, named by ID
   (e.g. `BDD-B3-027.feature`).
3. **TDD** — pytest-bdd test files in `/tests` wire the feature scenarios to step definitions and
   assertions. Tests are written **before** implementation. Red → green → refactor.
4. **CODE** — Python (FastAPI + Pydantic v2) and TypeScript (Next.js 14 App Router) implementation
   that makes the failing tests pass.

**You do not skip a step.** If a spec is missing or wrong, fix the spec first.
If a test is missing or wrong, fix the test first. The code is the last thing to change.

## When a user asks you to implement something

1. Look up the relevant spec(s) by ID. If the ID does not exist in `/specs`, stop and tell the user
   the spec is missing — do not invent one.
2. If the BDD `.feature` file does not exist, generate it from the spec's acceptance criteria.
   ACs in this repo are written in Given/When/Then form so the translation is mechanical.
3. If pytest-bdd step definitions and tests do not exist, generate them. Tests must be red.
4. Implement the minimum code to turn the tests green. Refactor without breaking tests.
5. Update the spec frontmatter: `status: done`, bump `version`, set `last-updated`.
6. Update `00-meta/03-traceability-matrix.md` (or rerun the script that generates it).

## Architecture patterns to respect

These are not suggestions; deviating from them requires an ADR.

- **Hexagonal architecture** (ports & adapters) on the Python backend. The domain layer has no
  AWS, AgentCore, or vendor knowledge. Adapters live in `adapters/` and implement domain ports.
- **Domain-Driven Design (light)**: one bounded context per workflow family
  (`reconciliation`, `kyc`, `loan_ops`, `regulatory_filing`). Each context has its own
  models, services, and adapter folder.
- **Event-driven internally**: agents emit domain events (`PlanCreated`, `StepExecuted`,
  `ExceptionRaised`, `RunCompleted`) on an internal event bus. Observability, audit log,
  cost meter, and customer dashboard are independent subscribers.
- **API-first with OpenAPI 3.1**: the file `specs/30-architecture/34-api-contract/openapi.yaml`
  is the contract. FastAPI types and the Next.js TypeScript client are generated from it.
  Never modify generated code by hand.
- **Configuration-as-code for workflows**: new workflows live as YAML files in `/workflows`.
  No code change is required to add a workflow; the runtime reads them.
- **ADRs for non-obvious choices**: anything that future-you will second-guess goes in
  `specs/30-architecture/32-adrs/` with a number and a rationale.

## Repository layout (high level)

```
backofficepilot/
├── specs/                  ← spec-driven source of truth (this folder)
├── features/               ← BDD feature files, one per story
├── tests/                  ← pytest + pytest-bdd, mirrors source tree
├── src/                    ← Python backend
│   ├── domain/             ← pure domain models + services (no AWS)
│   ├── application/        ← orchestration, use cases
│   ├── adapters/           ← AgentCore, Nova Act, Bedrock, Slack, ...
│   └── api/                ← FastAPI routers (thin, delegate to application)
├── web/                    ← Next.js 14 App Router frontend
│   ├── app/
│   ├── components/         ← shadcn/ui-based
│   └── lib/                ← generated OpenAPI client
├── workflows/              ← workflow YAML definitions
├── policy/                 ← customer-specific policy markdown
├── infra/                  ← AWS CDK (Python)
└── scripts/                ← traceability matrix generator, codegen, ...
```

## File naming convention (in /specs)

`<TYPE>-<DOMAIN>-<NUMBER>-<kebab-slug>.md`

Examples:

- `REQ-PRD-014-audit-log-retention.md`
- `STORY-B3-027-wire-agentcore-gateway-tool.md`
- `ADR-007-pytest-bdd-over-behave.md`
- `NFR-PERF-002-cost-per-run-budget.md`
- `DESIGN-A2-003-agentcore-runtime-integration.md`

Full naming grammar lives in `00-meta/02-naming-conventions.md`. Read it before creating a new file.

## Frontmatter schema (every spec file)

```yaml
---
id: STORY-B3-027            # immutable once published
title: Short human-readable title
type: story                 # req | nfr | story | epic | design | adr | spec
status: draft               # draft | approved | in-progress | done | superseded
owner: founder              # github handle or role
depends-on: [DESIGN-A2-003, REQ-PRD-018]
covers-req: [REQ-PRD-018, REQ-PRD-019]      # only on stories/designs
bdd: [BDD-B3-027.feature]                   # only on stories
tests: [test_gateway_netsuite.py::test_export_invoices]
version: 0.1.0
last-updated: 2026-05-19
---
```

## Stack lock-in (do not override without an ADR)

| Layer | Choice |
|---|---|
| Frontend | Next.js 14 App Router, TypeScript, Tailwind, shadcn/ui |
| Backend | Python 3.12, FastAPI, Pydantic v2, uv (package manager) |
| BDD | pytest-bdd |
| TDD | pytest, hypothesis (property-based for orchestrator), Playwright (E2E) |
| Infra | AWS CDK (Python), per-VPC stacks for Prod and UAT |
| Frontend tooling | pnpm |
| Observability | OpenTelemetry SDK → AgentCore Observability → CloudWatch GenAI Observability |

## When in doubt

- Read the relevant spec(s) first. Specs in `/specs` are authoritative.
- For "why is it like this," check `30-architecture/32-adrs/`.
- For "what does this term mean," check `00-meta/00-glossary.md`.
- If still ambiguous, ask the user before guessing.

## What you should never do

- Invent a spec ID that doesn't exist in `/specs`.
- Write production code before the failing test exists.
- Modify generated OpenAPI client code by hand (regenerate it instead).
- Mix domain logic and adapter logic in the same module.
- Skip the ADR when making a non-trivial architectural choice.
- Reference an agent service by lowercase or kebab-case (it is "AgentCore Runtime", not "agentcore-runtime").
