---
id: STORY-B4-005
title: Per-workflow cost guardrails with daily and monthly caps
type: story
status: draft
owner: founder
depends-on: [STORY-A3-012, NFR-COST-001]
covers-req: [REQ-PRD-002]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-005 — Cost guardrails

## Story

As a CFO at the customer, I need daily and monthly cost caps per workflow that pause the agent
when exceeded, so a runaway cost cannot accrue.

## Acceptance criteria

AC-1: Given a daily cap of $50 on the recon workflow,
      when the day's cumulative cost crosses it,
      then new runs are paused with `reason=daily_budget_exceeded`; the customer is notified.

AC-2: Given a monthly cap,
      when crossed,
      then the workflow is suspended until the operator approves an override or the calendar
      rolls.

AC-3: Given the customer dashboard,
      when a cap is approached (≥ 80%),
      then a warning banner appears with projection-to-cap.

## Definition of done

- [ ] Caps configurable per customer + per workflow.
- [ ] Pause + override flows tested.
- [ ] Customer onboarding covers the default caps.
