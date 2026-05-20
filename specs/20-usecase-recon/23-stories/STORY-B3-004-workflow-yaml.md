---
id: STORY-B3-004
title: Reconciliation workflow YAML definition
type: story
status: approved
owner: founder
depends-on: [STORY-A3-006, REQ-B1-001, DESIGN-A2-002]
covers-req: [REQ-B1-001, REQ-PRD-004]
bdd: [BDD-B3-004.feature]
tests: [tests/workflows/test_daily_reconciliation.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-004 — Workflow YAML

## Story

As the founder, I need `workflows/daily_reconciliation.yaml` written, validated, and persisted
as the active workflow for Acme, so the orchestrator has a definition to run against.

## Acceptance criteria

AC-1: Given the YAML at `workflows/daily_reconciliation.yaml`,
      when the validator runs,
      then it accepts; version `1.0.0` is recorded.

AC-2: Given the validated YAML,
      when an end-to-end run is triggered,
      then all 5 declared steps execute in order (pull bank → pull invoices → match → post →
      escalate).

AC-3: Given the YAML declares `budget.max_cost_per_run_usd: 1.00`,
      when a run exceeds the budget,
      then the cost ceiling enforces an abort.

AC-4: Given the YAML declares `triggers.cron: "0 13 * * MON-FRI"`,
      when the schedule fires,
      then a new run is created with `trigger=scheduled`.

## Definition of done

- [ ] YAML committed and validated.
- [ ] Snapshot semantics tested (mid-run update → in-flight unaffected).
- [ ] Tests pass against the mock + sandbox systems.
