---
id: NFR-COST-002
title: Cost meter accuracy against AWS billing
type: nfr
status: approved
owner: founder
depends-on: [ADR-012]
covers-req: [REQ-PRD-002]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-COST-002 — Cost meter accuracy

## Statement

The synthetic cost meter **shall** track within 5% of the corresponding AWS billing line items
when reconciled weekly. Drift exceeding 5% **shall** trigger an alert.

## Reconciliation procedure

A weekly job (`scripts/reconcile_costs.py`):

1. Pulls AWS Cost & Usage Report for the prior week.
2. Pulls synthetic cost-meter rollups for the same week from DDB.
3. Joins on `customer_id` and `environment`.
4. Computes per-component variance:
   - Bedrock (input + output tokens × published rate)
   - Nova Act (agent-hours × $4.75)
   - AgentCore Browser, Runtime, infrastructure
5. Emits a report; alerts on variance > 5%.

## Action on drift

- Drift > 5% but < 15%: update `config/pricing.yaml`; verify the update; investigate cause.
- Drift > 15%: page the operator; halt cost-meter publication to customer dashboards pending
  reconciliation.

## Verification

- Reconciliation job runs every Monday 09:00 UTC.
- Last-N-week variance is shown on an internal admin dashboard.
- Quarterly external review against billing data is part of the SOC 2 evidence pack.
