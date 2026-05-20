---
id: STORY-B3-002
title: NetSuite sandbox setup and seed data
type: story
status: approved
owner: founder
depends-on: [REQ-B1-002, REQ-B1-004]
covers-req: [REQ-B1-002]
bdd: []
tests: [tests/netsuite/test_export.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-002 — NetSuite sandbox

## Story

As the founder, I need a NetSuite sandbox account configured with the mock data set (open
invoices, recent payments, vendor master) so that Nova Act has a real NetSuite UI to drive.

## Acceptance criteria

AC-1: Given the NetSuite sandbox account exists,
      when an operator runs `scripts/seed_netsuite_sandbox.py`,
      then ~90 open invoices, ~15 recent payments, and ~25 vendors are inserted via the
      NetSuite REST API.

AC-2: Given the data is seeded,
      when Nova Act logs in and exports "Open Invoices",
      then a CSV with ~90 rows is downloaded.

AC-3: Given the agent posts a journal entry,
      when an operator checks the JE in the NetSuite UI,
      then it appears with the BackOfficePilot run ID as the JE memo.

## Technical notes

- NetSuite Suiteflow access for sandbox account.
- OAuth 2.0 setup with refresh tokens persisted to Secrets Manager.
- Seed script idempotent — re-runs do not duplicate records.

## Definition of done

- [ ] Sandbox seeded with all four data files.
- [ ] OAuth setup documented in the deployment runbook.
- [ ] Test verifies one round trip (export → post → re-export → confirm posted JE present).
