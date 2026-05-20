---
id: REQ-PRD-014
title: Audit log retention
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-003]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-014 — Audit log retention

## Statement

The system **shall** retain audit logs for a minimum of 7 years, **shall** support customer-
configurable retention up to 10 years, and **shall** export an extended retention copy on demand
prior to the retention boundary expiring.

## Acceptance criteria

AC-1: Given a default-tier customer,
      when audit objects are written,
      then S3 Object Lock applies a 7-year retention period in compliance mode.

AC-2: Given an enterprise-tier customer with a 10-year retention setting,
      when audit objects are written,
      then the retention period is 10 years.

AC-3: Given a record reaches its retention expiry,
      when the system marks it eligible for transition,
      then it moves to S3 Glacier Deep Archive (cheaper tier) without losing immutability.

AC-4: Given a customer requests early export of audit records prior to expiry,
      when authorized,
      then the system produces a tarred bundle, computes a SHA-256 manifest, and provides a
      signed URL.

## Implementation notes

- Retention policy is configured per S3 prefix per customer.
- Lifecycle rules transition to Glacier Deep Archive 7 years post-write (default tier).
- Retention configuration is itself audit-logged.
