---
id: STORY-A3-015
title: Dashboard — exception queue and cost view
type: story
status: approved
owner: founder
depends-on: [STORY-A3-004, DESIGN-A2-007, REQ-PRD-011, REQ-PRD-012]
covers-req: [REQ-PRD-011, REQ-PRD-012]
bdd: [BDD-A3-015.feature]
tests: [web/__tests__/escalations.spec.ts, web/__tests__/cost.spec.ts]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-015 — Exception queue + cost view

## Story

As a customer, I need an exception queue page where I can triage escalations inline, and a cost
page where I can see per-agent, per-workflow, and savings-vs-labor figures.

## Acceptance criteria

AC-1: Given open escalations exist,
      when I open `/escalations`,
      then they appear grouped by severity, oldest-first within severity.

AC-2: Given I expand an escalation,
      when it loads,
      then I see the Reasoner's narrative, screenshots, originating run link, and three
      actions: Approve, Reject, Edit.

AC-3: Given I choose Edit,
      when I modify the payload and submit,
      then `resolveEscalation` is called with the edited payload and the page reflects the
      resolved state.

AC-4: Given I open `/cost`,
      when it loads,
      then a stacked bar chart shows cost-by-workflow over the last 30 days using Recharts.

AC-5: Given I have configured a labor-cost baseline,
      when the cost page renders,
      then a "savings vs labor" panel shows $ saved and a moving average.

## Technical notes

- Exception queue uses SSE to update on new arrivals.
- Cost view fetches `CostDaily` rollups for the 30-day chart; current-day uses `CostMeter`.
- Edit form is auto-generated from the step result JSON Schema.

## Definition of done

- [ ] All ACs covered.
- [ ] Charts match brand palette.
- [ ] Tested under at least 100 escalations to ensure list virtualization is correct.
