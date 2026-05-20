---
id: ADR-010
title: Use AWS CDK (Python) for infrastructure
type: adr
status: approved
owner: founder
depends-on: [ADR-001, ADR-009]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-010: AWS CDK (Python) for infrastructure-as-code

## Context

Infrastructure is significant: VPCs, AgentCore Runtime/Gateway/Memory/Registry/Browser/
Observability, Lambda, Fargate, DynamoDB, S3 (with Object Lock), KMS, Secrets Manager,
EventBridge, API Gateway, CloudWatch dashboards. Two accounts (Prod + UAT). All must be
reproducible.

Two mainstream IaC choices: **Terraform** (HCL) and **AWS CDK** (TypeScript, Python, others).

## Decision

Use **AWS CDK** with **Python** as the language. One CDK app deploys both Prod and UAT
environments via parameterized stacks.

## Alternatives considered

- **Terraform**: cross-cloud, universally understood, mature module ecosystem (terraform-aws-modules).
  Wins on portability; loses on AgentCore coverage in early days (newer services are first-class
  in CDK constructs, second-class in TF provider releases).
- **AWS SAM**: limited to Lambda + a few service types. Not enough for our footprint.
- **CDK TypeScript**: equally good. We choose Python to keep the backend language single.
- **Pulumi**: less common; smaller ecosystem; not worth the deviation.

## Consequences

**Positive**

- One language across backend, IaC, and scripts.
- AgentCore constructs are first-party in CDK (or arrive there first when AWS releases L2
  constructs).
- Multi-account deploys handled by `cdk.context.json` and stack environments.
- Snapshot testing of synthesized CloudFormation via `aws-cdk.assertions`.

**Negative**

- AWS lock-in (acceptable given `ADR-005`).
- CDK upgrades occasionally break stacks; pin versions and bump deliberately.
- Some older AWS services have weaker L2 constructs; we drop to L1 where needed.

## Layout

```
infra/
├── app.py                          # CDK app entrypoint
├── cdk.context.json                # account / region pinning
├── stacks/
│   ├── network_stack.py            # VPC + subnets per env
│   ├── data_stack.py               # DynamoDB, S3, KMS
│   ├── agentcore_stack.py          # AgentCore Runtime, Gateway, Memory, Registry config
│   ├── api_stack.py                # API Gateway + Lambda for control plane
│   ├── observability_stack.py      # CloudWatch dashboards, alarms
│   └── frontend_stack.py           # Amplify or Vercel-adjacent resources
├── shared/                         # constants, helpers
└── tests/
    └── test_synth.py               # snapshot tests
```
