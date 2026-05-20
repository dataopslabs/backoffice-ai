---
id: STORY-B4-007
title: Reconciliation accuracy SLA monitoring
type: story
status: draft
owner: founder
depends-on: [STORY-B3-010]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-007 — Accuracy SLA monitoring

## Story

As a CFO at the customer, I need a contracted SLA on match accuracy (≥ 95% of clean cases
correctly matched) with monthly reporting and remediation if breached.

## Acceptance criteria

AC-1: Given sampled monthly review of posted JEs,
      when accuracy < 95% in a month,
      then a remediation report is produced; service credit applies per contract.

AC-2: Given a daily accuracy metric (computed via spot-checks),
      when the dashboard renders,
      then a rolling 30-day average is shown.

AC-3: Given the accuracy metric drifts down,
      when the trailing 7-day value drops below 95%,
      then the operator is alerted before the monthly breach occurs.

## Definition of done

- [ ] Sampling procedure documented.
- [ ] Accuracy metric live on the dashboard.
- [ ] SLA terms drafted into the customer contract template.
