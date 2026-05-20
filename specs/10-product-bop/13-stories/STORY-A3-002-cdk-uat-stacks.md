---
id: STORY-A3-002
title: CDK network, data, and AgentCore stacks for UAT
type: story
status: approved
owner: founder
depends-on: [STORY-A3-001, ADR-009, ADR-010]
covers-req: [REQ-PRD-001]
bdd: []
tests: [tests/infra/test_synth.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-002 — CDK UAT stacks

## Story

As the founder, I need a single `cdk deploy --env uat` command to provision the UAT VPC, data
stores (DDB, S3 with Object Lock, KMS), and AgentCore resources (Runtime fleet, Gateway,
Memory namespace, Registry, Observability), so that subsequent stories deploy against real
infrastructure.

## Acceptance criteria

AC-1: Given a clean UAT account,
      when I run `cdk deploy --env uat --all`,
      then a VPC `10.20.0.0/16` is created with three AZs and three subnet tiers.

AC-2: Given the deploy completes,
      when I list resources,
      then I see: 1 DDB table (single-table design), 3 S3 buckets (runs, audit with Object Lock,
      policy-bundles versioned), 1 KMS CMK, 1 AgentCore Runtime resource group, 1 AgentCore
      Memory namespace, 1 AgentCore Registry namespace.

AC-3: Given any synthesized stack,
      when `pytest tests/infra/test_synth.py` runs,
      then assertions confirm: no peering, all data stores encrypted with the CMK, audit bucket
      has compliance-mode Object Lock.

AC-4: Given a destroy command,
      when the operator runs `cdk destroy --env uat --all`,
      then non-protected resources are removed; audit bucket and CMK survive deletion (per
      retention policies).

## Technical notes

- Stack layout per `ADR-010`.
- Object Lock retention default 7 years (`REQ-PRD-014`).
- AgentCore resource names prefixed `bop-uat-`.
- A `cdk.context.json` lookup binds account ID to env; deploy to wrong account fails at synth.

## Definition of done

- [ ] UAT deploy succeeds end-to-end from a clean slate.
- [ ] `test_synth.py` covers VPC, IAM, KMS, S3 Object Lock, encryption settings.
- [ ] Documentation: `infra/README.md` covers prerequisites and how to deploy.
