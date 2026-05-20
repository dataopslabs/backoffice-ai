---
id: STORY-A3-011
title: S3 hash-chain audit writer
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-A2-005, NFR-COMP-001]
covers-req: [REQ-PRD-003]
bdd: [BDD-A3-011.feature]
tests: [tests/adapters/test_audit_writer.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-011 — Audit writer

## Story

As the founder, I need a subscriber that consumes every domain event and produces hash-chained
audit records written to DDB and to an Object-Lock S3 bucket, so that the audit story is real
from the first run.

## Acceptance criteria

AC-1: Given a sequence of events for a single run,
      when persisted,
      then each `AuditEvent.prev_hash` matches the previous event's `this_hash`.

AC-2: Given a completed run,
      when `audit.jsonl` is produced in the bundle,
      then `scripts/verify_audit.py` confirms full chain integrity.

AC-3: Given an attempt to delete an audit object in S3,
      when the actor has any role short of breaking Object Lock,
      then the delete is denied; CloudTrail captures the attempt.

AC-4: Given an event sequence with a gap (sequence 5 missing),
      when the chain is verified,
      then verification reports the gap.

AC-5: Given the audit subscriber falls behind (event bus backlog),
      when a configurable threshold is exceeded,
      then a CloudWatch alarm fires; the orchestrator's circuit breaker pauses new runs.

## Technical notes

- Adapter in `src/adapters/audit/s3_hash_chain.py`.
- Canonical JSON serializer in `src/domain/serializers.py`.
- Verification script in `scripts/verify_audit.py` (open-sourced under MIT).

## Definition of done

- [ ] Verification script ships alongside the adapter.
- [ ] All ACs covered by tests.
- [ ] CloudTrail rule asserted in CDK synth test.
