---
id: META-000
title: Glossary
type: meta
status: approved
owner: founder
depends-on: []
version: 0.1.0
last-updated: 2026-05-19
---

# Glossary

Definitions of terms used throughout the BackOfficePilot spec set. When a term appears in a spec,
use the exact spelling and capitalization defined here.

## Product and company

| Term | Definition |
|---|---|
| **BackOfficePilot** | The product. One word, camel case as shown. Never "Back Office Pilot" or "back-office-pilot" in prose. |
| **Founder** | The owner role on every spec until additional owners exist. |
| **Customer** | A BFSI institution that licenses BackOfficePilot. Typically mid-market: $1B–$50B AUM (banks), 50–500 employees in ops. |
| **End user** | A person at the customer who interacts with BackOfficePilot directly (ops analyst, BSA officer, CFO). |

## Core architecture

| Term | Definition |
|---|---|
| **Reasoner** | The role played by Claude Opus 4.6. Plans workflows, interprets policy, classifies exceptions, narrates audit evidence. |
| **Executor** | The role played by Amazon Nova Act (primary) and AgentCore Browser (fallback). Performs UI actions on customer systems. |
| **Orchestrator** | The deterministic control layer that mediates between Reasoner and Executor. Implemented as a FastAPI application hosted on AgentCore Runtime. |
| **Agent** | A long-lived configuration of orchestrator + workflow definition + customer-specific policy. One agent per customer per workflow type. |
| **Workflow** | A bounded automation procedure (e.g. "daily invoice reconciliation"). Defined as YAML in `/workflows/`. |
| **Run** | One execution of a workflow on a specific input set (e.g. yesterday's bank transactions). Has a unique `run_id`, a status, a cost, and a complete audit trail. |
| **Step** | One unit of work within a run, dispatched by the orchestrator to the Executor. |
| **Plan** | The structured JSON output of the Reasoner that lists the steps to execute. |
| **Exception** | A situation the agent cannot resolve deterministically. Routed to a human reviewer via Slack or Teams. |
| **Escalation** | The act of routing an exception to a human, plus the SLA timer that runs until they respond. |

## AWS / AgentCore services

Reference the AWS marketing capitalization. AgentCore service names are always two words,
each capitalized, with no hyphen.

| Term | Definition |
|---|---|
| **Amazon Bedrock** | The AWS managed-foundation-model service. We invoke Claude Opus 4.6 through Bedrock. |
| **AgentCore Runtime** | Serverless host for agents. Each agent is deployed as a runtime resource. |
| **AgentCore Gateway** | Tool and API exposure layer. Translates customer APIs and MCP servers into agent-callable tools. |
| **AgentCore Memory** | Per-session state, per-project context, and long-term customer knowledge store. |
| **AgentCore Registry** | Source of truth for agent versions. Rollout, rollback, canary all flow through Registry. |
| **AgentCore Browser** | Sandboxed browser-host service. Used for dev/test and as a fallback Executor when Nova Act cannot reach a system. |
| **AgentCore Observability** | Trace and metric collection for agent activity. Exports to CloudWatch GenAI Observability. |
| **CloudWatch GenAI Observability** | The AWS dashboarding surface for GenAI workloads. Reads from AgentCore Observability. |
| **Bedrock prompt cache** | The caching feature on Bedrock that lets us pay 10% of input token cost for stable prefix content (customer policy bundles, workflow definitions). |
| **Bedrock Batch API** | Asynchronous Bedrock invocation with 50% discount. Used for non-real-time runs. |
| **Amazon Nova Act** | UI-execution agent service from AWS. $4.75 / agent-hour. Reached GA December 2025. |

## Multi-model orchestration

| Term | Definition |
|---|---|
| **Plan / Execute / Reconcile / Commit (PERC) loop** | The deterministic agent loop. Plan = Reasoner produces JSON plan. Execute = Executor performs step. Reconcile = Reasoner judges result. Commit = Orchestrator writes run bundle. |
| **Handoff protocol** | The schema-validated contract between Reasoner and Executor. JSON Schemas live in `30-architecture/34-api-contract/`. |
| **Cost ceiling** | A per-run hard cap (default $1.00) enforced by the orchestrator. Breach aborts the run. |
| **Run bundle** | The full set of artifacts produced by a run: plan, step results, audit log, screenshots, cost meter, exception list. Persisted to S3 (WORM) and indexed in DynamoDB. |
| **Audit hash chain** | SHA-256 chain across run events. Each event includes the previous event's hash. Tamper evidence. |

## Compliance and security

| Term | Definition |
|---|---|
| **WORM** | Write Once Read Many. S3 Object Lock in compliance mode. Used for run bundles. |
| **Least privilege** | IAM policy posture: agents have access only to the systems and credentials their workflow declares. |
| **Customer-managed key** | KMS key whose lifecycle is controlled by the customer. We never see the key material. |
| **PII** | Personally Identifiable Information. Subject to redaction policies depending on jurisdiction. |
| **PHI** | Protected Health Information. Subject to HIPAA controls. Out of scope for v1. |
| **HIPAA / PCI-DSS / GLBA / SOX** | Regulatory frameworks customers may be subject to. Spec annotations call out which controls apply where. |
| **SOC 2** | Auditable controls framework. BackOfficePilot targets Type II readiness by Q3 2027. |
| **VPC isolation** | Network-level separation. Prod and UAT run in distinct VPCs with no shared subnets, transit gateway, or peering. |

## Engineering process

| Term | Definition |
|---|---|
| **Spec** | A markdown file in `/specs` with the standard frontmatter that defines a requirement, story, design, ADR, or NFR. |
| **BDD** | Behavior-Driven Development. In this repo: Gherkin feature files under `/features` exercised by pytest-bdd. |
| **TDD** | Test-Driven Development. In this repo: pytest tests written before the implementation. |
| **ADR** | Architecture Decision Record. A short document capturing a non-obvious choice and its rationale. |
| **Traceability matrix** | The auto-generated mapping from each REQ/NFR to the stories that implement it, the feature files that exercise it, and the test functions that assert it. |
| **Bounded context** | A DDD term for a self-contained domain area. Each workflow family (reconciliation, KYC, loan ops, regulatory) is one. |
| **Port** | A domain-layer interface (Python ABC) describing what the domain needs from an adapter. |
| **Adapter** | An infrastructure-layer class that implements a port using a specific vendor or library (Bedrock, Nova Act, AgentCore, Slack, ...). |
| **Hexagonal architecture** | The pattern that separates domain logic from infrastructure via ports and adapters. Also called "ports and adapters." |
| **Event bus** | The internal in-process or SNS/EventBridge mechanism by which domain events propagate to subscribers. |

## File-system terms

| Term | Definition |
|---|---|
| `/specs` | This folder. Spec source of truth. |
| `/features` | Gherkin feature files (one per story). |
| `/tests` | pytest test modules mirroring the source tree. |
| `/src` | Python backend source. Hexagonal layout (`domain/`, `application/`, `adapters/`, `api/`). |
| `/web` | Next.js 14 frontend source. |
| `/workflows` | YAML workflow definitions. |
| `/policy` | Customer-specific policy markdown bundles. |
| `/infra` | AWS CDK (Python) stacks. Prod and UAT stacks deploy to separate accounts. |

## Word use to standardize

- **agent** (lowercase) when used as a noun, e.g. "the reconciliation agent."
- **Agent** (capitalized) only at the start of a sentence or when referring to the role abstractly.
- **executor**, **reasoner**, **orchestrator** lowercase in prose; PascalCase when referring to a code class.
- **back-office** hyphenated when used as an adjective. "Back-office workflows."
- **mid-market** hyphenated. "Mid-market BFSI."
- **runtime**, **registry**, **gateway**, **memory** lowercase when generic; PascalCase as part of AgentCore service names.

## Terms to avoid

- "RPA" — used only when contrasting BackOfficePilot to incumbents. We are not RPA.
- "Bot" — implies stateless scripting. We are agents.
- "AI copilot" — implies a human-in-the-driver-seat tool. We are unattended.
- "Black box" — undermines our audit story. Prefer "explainable" or "audit-grade."
