---
id: REQ-B1-003
title: Registered exception categories
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-008]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-B1-003 — Exception categories

## Categories

The MVP workflow registers four exception categories:

| Category | Definition | Severity (default) |
|---|---|---|
| `amount_mismatch` | Payment amount differs from any open invoice by more than configured tolerance ($0.05) | medium |
| `missing_invoice` | Payment cannot be matched to any open invoice within 60 days | medium |
| `duplicate_payment` | Two payments match the same invoice on the same day | high |
| `currency` | Currency code of payment does not match invoice (any USD/non-USD mismatch) | medium |

Any condition the Reasoner cannot classify into one of the above is `unknown` (severity high).

## Acceptance criteria

AC-1: Given a payment $9,847.32 and an open invoice $9,847.23,
      when the agent runs three-way match with $0.05 tolerance,
      then the exception is classified `amount_mismatch` and escalated.

AC-2: Given a payment that doesn't match any open invoice within 60 days,
      when the agent runs,
      then `missing_invoice` is raised with the closest fuzzy-match candidate attached for human
      review.

AC-3: Given two payments dated the same day for the same invoice id,
      when the agent runs,
      then `duplicate_payment` is raised before either is posted.

AC-4: Given a payment in EUR for an invoice in USD,
      when the agent runs,
      then `currency` is raised with both amounts and the prevailing FX rate attached.

AC-5: Given the Reasoner produces a low-confidence classification,
      when reconcile completes,
      then the exception is escalated as `unknown` with the Reasoner's full narrative attached.

## Routing

All four categories route to Slack channel `#bop-reconciliation`. Email fallback applies if the
SLA breaches per `REQ-PRD-009`.
