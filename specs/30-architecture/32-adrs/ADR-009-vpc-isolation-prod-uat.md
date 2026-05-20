---
id: ADR-009
title: Prod and UAT in separate VPCs with no shared data path
type: adr
status: approved
owner: founder
depends-on: [ADR-005]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-009: Prod and UAT in separate VPCs with no shared data path

## Context

Customer pilots must run in a Production environment that is materially indistinguishable from
the long-term home of their data. Engineers must be able to test against AgentCore Runtime,
Gateway, Memory, Registry, and Observability without risking customer data exposure or
side-effects.

The standard cloud approach offers a spectrum:

- Single account, single VPC, subnet/tag separation.
- Single account, two VPCs.
- Two AWS accounts (one Prod, one UAT) with two VPCs.

For BFSI customers and the future SOC 2 audit, the bar is high.

## Decision

- **Two AWS accounts**, one for Prod, one for UAT. Linked via AWS Organizations under a single
  payer.
- **One VPC per account** in the primary region (`us-east-1` initially).
- **No peering, no transit gateway, no shared subnets** between Prod and UAT.
- **No shared state stores**: Prod DynamoDB tables, S3 buckets, KMS keys, Secrets Manager
  secrets are distinct from UAT and live only in the Prod account.
- **No shared IAM roles** across accounts other than the deploy-from-CI role, which can write to
  either account but cannot move data between them.
- Both VPCs are deployed by the same CDK app (`ADR-010`) with environment-parameterized stacks.

## Alternatives considered

- **Single account, single VPC, namespacing by tag/prefix**: cheapest, weakest isolation, fails
  audit conversations.
- **Single account, two VPCs**: stronger network isolation but blast radius is one IAM mistake.
- **Three accounts (Prod, UAT, Dev)**: adds a Dev account. We collapse Dev into UAT for now;
  may split later (left as a candidate ADR).

## Consequences

**Positive**

- Hard isolation. A misconfigured UAT IAM policy cannot reach Prod data.
- Independent billing visibility per account.
- AgentCore Runtime fleets are independent: a runtime bug in UAT cannot affect Prod agents.
- Customer audit conversation has a clean answer.

**Negative**

- Two accounts to operate. Slightly higher per-month AWS overhead.
- Cross-account IAM role discipline required for the deploy pipeline.
- Some AWS services have per-account quotas that we must monitor twice.

## Implementation

- CDK app exposes a `--env prod|uat` flag and deploys to the appropriate account.
- Each stack has a name suffix (`-prod`, `-uat`) and cannot be deployed to the wrong account
  (a `cdk.context.json` lookup validates account ID at synth time).
- Network: each VPC has three AZs, public/private/isolated subnets. AgentCore Runtime resources
  in isolated subnets.
- All cross-account communication via S3 cross-account bucket policies (read-only, log-shipping
  only).
