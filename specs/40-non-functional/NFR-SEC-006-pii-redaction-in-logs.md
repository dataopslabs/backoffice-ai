---
id: NFR-SEC-006
title: PII redaction in logs and traces
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-SEC-006 — PII redaction in logs

## Statement

Application logs, CloudWatch logs, and OpenTelemetry trace payloads **shall not** contain
unredacted PII or PCI data. Customer transaction-level data **shall** be referenced by IDs in
logs, not by raw values; raw values live only in the encrypted run bundle.

## Coverage

- Account numbers, names, SSN-equivalents, card numbers: redacted by a logging filter.
- Email addresses: hashed for log keys; raw values only in audit log (encrypted).
- Free-form text from customer documents: redacted by a regex-based scrubber (with audit) before
  being added to traces.

## Verification

- A pre-commit hook runs `scripts/lint_logs.py` to detect log statements that pass raw model
  inputs/outputs.
- Sample-based runtime check: `LoggingPIIDetector` scans 1% of log lines; matches trigger an
  alert.
- Quarterly review of CloudWatch log groups by sampling and grepping for forbidden patterns.

## Where PII is allowed

- Encrypted run bundle in S3 (Object Lock).
- Audit log entries (encrypted, immutable).
- AgentCore Memory long-term context (per customer, BYOK eligible).

Anywhere else: redacted.
