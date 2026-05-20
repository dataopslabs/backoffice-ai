---
id: NFR-SEC-001
title: VPC isolation between Prod and UAT
type: nfr
status: approved
owner: founder
depends-on: [ADR-009]
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-SEC-001 — VPC isolation

## Statement

Prod and UAT environments **shall** run in separate AWS accounts with separate VPCs and **shall**
have no peering, transit gateway, PrivateLink, or shared subnet between them. The only
cross-account interaction permitted is a deploy role used by the CI pipeline.

## Verification

- CDK `infra/tests/test_synth.py` asserts there are no peering or transit gateway resources in
  the synthesized template.
- CloudFormation drift detection runs weekly; any drift creating cross-account network resources
  triggers an alert.
- AWS Config rule `vpc-peering-disallowed` enabled in both accounts.

## Evidence for audit

- IAM policies for the deploy role limit it to `cloudformation:*` and resource creation; cannot
  move data.
- VPC Flow Logs to S3 in each account.
- Account-level Service Control Policies (SCPs) block creation of cross-account VPC peering.
