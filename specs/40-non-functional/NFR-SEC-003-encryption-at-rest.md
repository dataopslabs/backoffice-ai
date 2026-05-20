---
id: NFR-SEC-003
title: Encryption at rest with customer-managed KMS keys
type: nfr
status: approved
owner: founder
depends-on: [DESIGN-DATA-002]
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-SEC-003 — Encryption at rest

## Statement

All persistent data **shall** be encrypted at rest with AES-256 using AWS KMS customer-managed
keys (CMKs). Enterprise-tier customers **shall** have the option to supply their own CMK (BYOK).
No data may be persisted unencrypted at any storage tier.

## Coverage

| Store | Key |
|---|---|
| DynamoDB | `alias/bop-<env>-cmk` (default) or per-customer alias (BYOK) |
| S3 (audit, run bundles, policy bundles) | Same, default-encrypted via bucket policy |
| OpenSearch | Same |
| Secrets Manager | AWS-managed CMK by default; customer-managed for BYOK tier |
| AgentCore Memory | AWS-managed (BYOK support tracked on AWS roadmap) |
| EBS for Fargate tasks | Same |

## Verification

- AWS Config rule `s3-bucket-server-side-encryption-enabled`.
- AWS Config rule `dynamodb-table-encryption-enabled` with KMS check.
- Synth-time test in CDK asserts every persisted resource has `encryption` configured to
  customer-managed CMK.

## BYOK implementation

- Customer provides a KMS key ARN with a key policy allowing BackOfficePilot's role to encrypt
  and decrypt.
- Per-customer adapter swaps the default CMK alias for the BYOK key.
- BYOK key rotation is the customer's responsibility; we log a notice if rotation is overdue.
