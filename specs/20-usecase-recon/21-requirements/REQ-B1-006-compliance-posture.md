---
id: REQ-B1-006
title: Compliance posture for the MVP pilot
type: req
status: approved
owner: founder
depends-on: [NFR-COMP-001, NFR-SEC-003, NFR-SEC-006]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-B1-006 — Compliance posture

## Statement

The MVP pilot **shall** meet the security and compliance baseline expected of a BFSI mid-market
back-office tool, even though it runs only against synthetic data, so that the controls are
exercised in UAT before a real customer's data is involved.

## Required controls in UAT

| Control | Source NFR | UAT verification |
|---|---|---|
| VPC isolation | NFR-SEC-001 | UAT VPC distinct from any future Prod VPC; no peering |
| Least-privilege IAM | NFR-SEC-002 | Per-customer prefix on every resource; lint check in CI |
| Encryption at rest (CMK) | NFR-SEC-003 | All stores encrypted with `alias/bop-uat-cmk` |
| Encryption in transit | NFR-SEC-004 | TLS 1.3 enforced |
| Secrets rotation | NFR-SEC-005 | Mock-bank TOTP secret rotation tested every 30 days |
| PII redaction in logs | NFR-SEC-006 | Even synthetic data is redacted; the discipline is real |
| Audit log immutability | NFR-COMP-001 | S3 Object Lock compliance mode active in UAT |
| Audit retention | REQ-PRD-014 | 7-year retention on UAT audit bucket |

## Out of scope for the MVP

- SOC 2 Type II readiness (covered by full-blown stories in EPIC-A-002).
- HIPAA-equivalent controls (not applicable to this workflow).
- BYOK CMK (default CMK is fine for the MVP).
- Pen testing (scheduled for post-MVP).

## Acceptance criteria

AC-1: Given any UAT spec is reviewed,
      when measured against the baseline above,
      then it satisfies each row.

AC-2: Given a synthetic transaction is processed,
      when logs are inspected,
      then no PII-looking strings appear in CloudWatch (the regex scrubber's discipline holds
      even for fake data).

AC-3: Given the auditor scenario rehearsal,
      when we hand them a run bundle and the verification script,
      then they confirm chain integrity in under 5 minutes.
