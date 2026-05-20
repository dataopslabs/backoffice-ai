---
id: REQ-B1-004
title: Mock data specification
type: req
status: approved
owner: founder
depends-on: [REQ-B1-001, REQ-B1-003]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-B1-004 — Mock data specification

## Purpose

During the MVP build, we use synthetic data only. No customer data touches the system. The mock
data must be realistic enough to exercise every code path including all four exception
categories.

## Dataset

The mock dataset comprises:

| File | Contents |
|---|---|
| `bank_transactions_YYYY-MM-DD.csv` | ~50 daily transactions; debits and credits; vendor descriptions in free-form text |
| `netsuite_open_invoices.csv` | ~90 open invoices spanning 60 days, with vendor name, amount, due date, invoice number |
| `netsuite_payments_recent.csv` | ~15 payments posted in the prior 7 days for de-duplication |
| `vendor_master.csv` | ~25 vendors with name variants ("Acme Industries", "ACME Inds.", "ACME Industries Inc.") for fuzzy match |

## Injected exception cases

Each daily batch includes 4–6 exception cases:

- 2 × `amount_mismatch` (one within $5, one within $50)
- 1 × `missing_invoice` (payment with no matching invoice within 60 days)
- 1 × `duplicate_payment` (two payments for the same invoice on the same day)
- 1 × `currency` (EUR payment for USD invoice)
- Optional 1 × `unknown` (ambiguous case for testing the low-confidence path)

## Generation

A script `scripts/gen_mock_recon_data.py` produces a fresh batch on demand. Reproducibility is
provided by a `--seed` flag.

## Acceptance criteria

AC-1: Given the generator with seed 42,
      when invoked,
      then it produces the same exact CSVs every time.

AC-2: Given a generated batch,
      when fed to the agent end-to-end,
      then all five exception categories surface at least once across a 5-day rolling window.

AC-3: Given the dataset,
      when inspected,
      then no field contains real-world PII; vendor names are fictional or derived from public
      sample data.
