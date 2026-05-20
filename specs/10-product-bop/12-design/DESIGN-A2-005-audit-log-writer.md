---
id: DESIGN-A2-005
title: Audit log writer (hash chain + WORM)
type: design
status: approved
owner: founder
depends-on: [REQ-PRD-003, NFR-COMP-001]
covers-req: [REQ-PRD-003]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-A2-005 — Audit log writer

## Goal

A subscriber that consumes every domain event and produces an immutable, tamper-evident audit
record. Without this, the audit story is marketing.

## Algorithm

```python
def write_audit_event(event: DomainEvent, prev_hash: str) -> AuditEvent:
    payload = canonicalize(event)                  # deterministic JSON
    content = f"{prev_hash}\n{payload}"
    this_hash = sha256(content.encode()).hexdigest()
    return AuditEvent(
        event_id=ulid(),
        run_id=event.run_id,
        sequence=event.sequence,
        event_type=event.event_type,
        prev_hash=prev_hash,
        this_hash=this_hash,
        payload=payload,
        timestamp=event.occurred_at,
    )
```

Canonicalization: sorted keys, UTF-8 encoding, no whitespace, ISO-8601 timestamps. Stable across
Python runtime versions.

## Storage

- DDB write of the audit event for fast query (single-table, PK `RUN#{run_id}`, SK
  `AUDIT#{sequence:08d}`).
- S3 PUT of the same record as a JSON object under `s3://bop-{env}-audit/cust={id}/run={run_id}/`
  with Object Lock retention.
- On `RunCompleted` or `RunAborted`, the full event stream is also packaged as a single
  `audit.jsonl` and stored alongside the run bundle.

## Sequencing

Each run has its own sequence counter starting at 0. Counter is DDB-atomic-incremented.
Out-of-order events fail the chain integrity check and trigger an alert.

## Verification path

- Customer or auditor downloads `audit.jsonl` and the SHA-256 manifest.
- A simple replay script (`scripts/verify_audit.py`) recomputes the hash chain and confirms
  every `this_hash` matches the expected value.
- The replay script is open-sourced (or at minimum, made available to customers) so the
  verification is not dependent on our software.

## Failure modes

| Failure | Detection | Recovery |
|---|---|---|
| DDB write succeeds, S3 PUT fails | S3 transaction retry; if it persistently fails, the run is paused | Page on-call |
| S3 succeeds, DDB fails | Eventually consistent re-sync from S3 | Background reconcile job |
| Sequence skipped due to bug | Chain verification fails on the next event | Halt audit writes; investigate |
| Adversarial attempt to insert a forged event | Hash chain verification fails | Detected at any consumer; alerts |

## Performance

- Single S3 PUT per event (~30ms p95).
- Batched DDB writes where possible (sequence is strict, so batching is per-run).
- Async write pattern: orchestrator does not block on audit write; audit failure trips a
  circuit breaker that pauses new runs but does not retroactively break completed ones.
