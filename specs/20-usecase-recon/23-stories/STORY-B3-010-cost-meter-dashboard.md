---
id: STORY-B3-010
title: Per-workflow cost-meter dashboard and alarms
type: story
status: approved
owner: founder
depends-on: [STORY-A3-012, DESIGN-B2-005]
covers-req: [REQ-PRD-002, REQ-PRD-012]
bdd: []
tests: [tests/infra/test_observability_stack.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-010 — Cost-meter dashboard + alarms

## Story

As the founder, I need both the operator CloudWatch dashboard ("Recon Agent Health") and the
customer-facing cost view populated with per-workflow metrics and alarms for the recon agent.

## Acceptance criteria

AC-1: Given a sequence of runs,
      when CloudWatch dashboard renders,
      then it shows the seven sections from `DESIGN-B2-005` populated with live data.

AC-2: Given an alarm condition (p95 latency > 5s for 15 min),
      when crossed,
      then SNS notifies the operator pager.

AC-3: Given the customer dashboard,
      when it loads,
      then per-workflow cost and savings panels show data sourced from `CostMeter` /
      `CostDaily` tables.

AC-4: Given a synthetic cost-spike injection,
      when applied,
      then the `CostPerRunHigh` alarm fires within 1 hour.

## Definition of done

- [ ] CloudWatch JSON dashboards committed under `infra/observability/`.
- [ ] All four alarms tested end-to-end.
- [ ] Customer dashboard surfaces per-workflow cost.
