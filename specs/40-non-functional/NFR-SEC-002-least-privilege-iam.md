---
id: NFR-SEC-002
title: Least-privilege IAM
type: nfr
status: approved
owner: founder
depends-on: [ADR-005, ADR-009]
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-SEC-002 — Least-privilege IAM

## Statement

Every IAM role **shall** be scoped to the minimum permissions required for its task, **shall**
use per-customer resource prefixes where applicable, and **shall** never grant `*` on a resource
ARN unless the resource type is intrinsically wildcard (e.g. CloudWatch metric publish).

## Verification

- IAM Access Analyzer enabled in both accounts; weekly review of unused permissions.
- All roles synthesize from CDK; `aws-cdk.aws_iam.PolicyStatement` review on every PR.
- Custom linter `scripts/lint_iam.py` parses synthesized CFN and flags any `Resource: "*"` on
  S3, DDB, KMS, Secrets Manager, or Lambda invoke.

## Patterns

- Per-customer S3 prefix: `s3://bop-<env>-runs/cust=<customer_id>/*`.
- Per-customer DDB conditions: `dynamodb:LeadingKeys` IAM condition with the customer's PK.
- Per-customer secret ARN pattern: `arn:aws:secretsmanager:<region>:<account>:secret:cust/<id>/*`.
- Per-customer KMS alias: `alias/bop-<env>-cust-<id>`.

## Evidence for audit

- CloudTrail logs all `AssumeRole` events.
- Access Analyzer reports stored as audit artifacts.
- Quarterly review checklist in `scripts/iam_review_checklist.md`.
