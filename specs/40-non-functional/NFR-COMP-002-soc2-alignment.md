---
id: NFR-COMP-002
title: SOC 2 Type II control alignment
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-COMP-002 — SOC 2 Type II alignment

## Statement

The system **shall** be architected, operated, and documented to support a SOC 2 Type II audit
across the Security, Availability, and Confidentiality trust service criteria, with Type II
readiness targeted by Q3 2027.

## Control mapping (partial — illustrative)

| TSC | Control reference | BackOfficePilot evidence |
|---|---|---|
| CC6.1 — Logical access | IAM least privilege | `NFR-SEC-002`, IAM Access Analyzer reports |
| CC6.7 — Restricted access to information assets | Per-tenant resource prefixes | `REQ-PRD-001` |
| CC7.2 — Monitoring of system components | CloudWatch + GuardDuty + AgentCore Observability | `NFR-OBS-001` |
| CC7.3 — Evaluation of security events | Incident response runbook + 72h breach notice | Runbook in `B5` |
| CC9.1 — Risk mitigation | ADR set documents risk decisions | `30-architecture/32-adrs/` |
| A1.1 — Availability commitments | 99.5% SLO | `NFR-AVAIL-001` |
| C1.1 — Confidentiality | Encryption + audit log immutability | `NFR-SEC-003`, `NFR-COMP-001` |

## Pre-audit checklist (year 1)

1. Engage SOC 2 advisor (Vanta, Drata, or equivalent) — Q3 2026.
2. Implement continuous compliance monitoring — Q4 2026.
3. Complete Type I gap assessment — Q1 2027.
4. Type II observation period begins — Q2 2027.
5. Type II report issued — Q3 2027.

## Verification

- Quarterly internal audit run against the control list.
- Evidence repository maintained in `compliance/soc2/` (separate from this spec repo).
