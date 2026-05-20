---
id: NFR-AVAIL-001
title: Availability SLO
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-AVAIL-001 — Availability SLO

## Statement

The control-plane API **shall** maintain a monthly availability of 99.5% measured at the
CloudFront edge. Workflow run completion **shall** maintain a 99.0% monthly success rate
(success = run reached terminal state without an `internal_error` abort reason).

## Error budget

- 99.5% monthly = 3.6 hours of unavailability per 30-day period.
- 99.0% workflow success = 7.2 hours equivalent of failure burn.

## What does and does not count

- **Counts against SLO**: BackOfficePilot service errors (5xx), failed Bedrock calls due to our
  configuration, AgentCore service errors we propagate without recovery.
- **Does not count**: Customer system unavailability (NetSuite down), customer credential
  rotation (expected pause), explicit operator-initiated maintenance windows.

## Verification

- CloudWatch synthetic Canary in 1-minute intervals.
- Monthly SLO report; if error budget burns >50%, freeze non-critical changes for the rest of
  the period.
- Public status page (`status.backofficepilot.ai`) with incident history.

## Incident response

- On-call rotation begins at the first paying customer.
- 30-minute first response SLO during business hours; 2-hour outside.
- Post-mortems within 5 business days for any customer-impacting incident.
