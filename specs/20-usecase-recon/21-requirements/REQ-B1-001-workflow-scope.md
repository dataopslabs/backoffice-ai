---
id: REQ-B1-001
title: Acme reconciliation — workflow scope
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-004]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-B1-001 — Acme reconciliation workflow scope

## Customer profile

**Acme Community Bank** — fictional $5B-asset community bank used as the design partner for
the MVP demo. Two-person finance/ops team currently performs daily reconciliation manually.

## Scope (in)

The agent **shall** perform daily three-way matching across the following data sources:

1. Acme's transaction feed (CSV) downloaded from the bank treasury portal.
2. Open invoices and recent payments exported from NetSuite.
3. Vendor master in NetSuite (for fuzzy-name matching).

For matched transactions, the agent **shall** post journal entries to NetSuite. For exceptions,
the agent **shall** escalate to a Slack channel.

## Scope (out)

- Vendor onboarding.
- Tax classification.
- Multi-currency reconciliation (deferred to `STORY-B4-001`).
- Posting to any system other than NetSuite.

## Acceptance criteria

AC-1: Given the daily reconciliation runs at 07:00 CT on a weekday,
      when 50 transactions are available,
      then the agent completes the run within 8 minutes and posts at least 80% as matched.

AC-2: Given a run includes 4–6 injected exception cases,
      when reconcile completes,
      then each exception is classified into one of the four registered categories.

AC-3: Given a posted journal entry,
      when the customer reviews it in NetSuite,
      then it includes a reference back to the BackOfficePilot run ID.
