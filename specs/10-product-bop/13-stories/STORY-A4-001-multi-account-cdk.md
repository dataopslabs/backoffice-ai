---
id: STORY-A4-001
title: Multi-account CDK and Prod environment bring-up
type: story
status: draft
owner: founder
depends-on: [STORY-A3-002, ADR-009, ADR-010]
covers-req: [REQ-PRD-001]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-001 — Prod environment bring-up

## Story

As the founder, I need a Prod AWS account stood up with its own VPC, data stores, and AgentCore
fleet, deployable by `cdk deploy --env prod --all`, so that the first paying customer can run
in Prod.

## Acceptance criteria

AC-1: Given the Prod AWS account is provisioned in the Organization,
      when `cdk deploy --env prod` runs,
      then the VPC, data, agentcore, and api stacks are created in `us-east-1`.

AC-2: Given the Prod stacks,
      when synth-time tests run,
      then they pass the same security assertions as UAT (no peering, all encrypted, Object
      Lock compliance).

AC-3: Given a deploy attempt against the wrong account,
      when CDK synth runs,
      then it fails before any CloudFormation change is applied.

AC-4: Given a manual approval gate in GitHub Actions,
      when an operator approves,
      then Prod deploy proceeds; otherwise the workflow blocks.

## Definition of done

- [ ] Prod is stood up end-to-end.
- [ ] First customer (paid pilot) is onboardable.
- [ ] Runbook for first deploy documented.
