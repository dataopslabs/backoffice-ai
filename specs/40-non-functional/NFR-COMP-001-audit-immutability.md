---
id: NFR-COMP-001
title: Audit log immutability (S3 Object Lock compliance mode)
type: nfr
status: approved
owner: founder
depends-on: [DESIGN-DATA-002]
covers-req: [REQ-PRD-003, REQ-PRD-014]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-COMP-001 — Audit immutability

## Statement

Audit log objects in S3 **shall** be protected by Object Lock in compliance mode for the
configured retention period. No identity, including the AWS account root, **shall** be able to
delete or modify a locked object before its retention expires.

## Verification

- AWS Config rule `s3-bucket-default-lock-enabled` for the audit bucket.
- Synth-time test asserts compliance mode (not governance).
- A red-team simulation runs quarterly: attempt to delete a locked object using a privileged
  role; the attempt must fail.

## Operational notes

- Compliance mode is irrevocable; mis-set retention periods cost real money. Test settings in
  UAT for at least 1 week before applying to Prod.
- Bucket policy denies `s3:DeleteObjectVersion` on locked objects regardless of IAM grants.
- Glacier Deep Archive transition at retention expiry (`REQ-PRD-014`).
