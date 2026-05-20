---
id: REQ-PRD-003
title: Audit log immutability and chain integrity
type: req
status: approved
owner: founder
depends-on: [ADR-006, DESIGN-DATA-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-003 — Audit log immutability

## Statement

The system **shall** record every state transition of every run as an `AuditEvent` in an
append-only, hash-chained, tamper-evident store from which no record can be deleted by any
party (including BackOfficePilot itself) for the retention window.

## Rationale

The audit log is the customer's evidence in regulatory exams and the differentiator versus
incumbents. If the chain is breakable, the differentiator collapses.

## Acceptance criteria

AC-1: Given a completed run,
      when the run bundle is fetched,
      then it contains a sequenced list of `AuditEvent` records covering every state transition.

AC-2: Given any audit event in the chain,
      when its `prev_hash` is recomputed and compared,
      then it matches the previous event's `this_hash`; any mismatch is detectable.

AC-3: Given the audit bucket and an authorized administrator,
      when they attempt to delete or modify an audit object,
      then the S3 Object Lock policy (compliance mode) denies the operation.

AC-4: Given an audit export request,
      when an authorized compliance officer requests an export,
      then they receive a signed URL and a SHA-256 manifest enabling independent integrity
      verification.

AC-5: Given a run that was aborted mid-flight,
      when the audit log is inspected,
      then the abort reason, the partial step states, and the cost meter snapshot are present.

## Implementation notes

- S3 bucket with Object Lock in compliance mode (not governance mode).
- SHA-256 hash chain implemented in `S3HashChainAuditLog` adapter
  (`DESIGN-C4-003`, `DESIGN-A2-005`).
- Retention default 7 years; configurable per customer per `REQ-PRD-014`.

## Linked NFRs

- `NFR-COMP-001` Audit log immutability.
- `NFR-COMP-002` Data retention 7 years.
