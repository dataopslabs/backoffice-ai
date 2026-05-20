---
id: STORY-A4-006
title: SLA tracker and breach notifications
type: story
status: draft
owner: founder
depends-on: [STORY-A3-013, DESIGN-A2-004]
covers-req: [REQ-PRD-009]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-006 — SLA tracker

## Story

As an ops manager at the customer, I need open escalations to honor an SLA timer with a fallback
notification on breach, so that no exception sits forgotten.

## Acceptance criteria

AC-1: Given an escalation opens with `reviewer_sla_minutes=60`,
      when 60 minutes elapse without resolution,
      then `SlaBreached` is emitted; an email is sent to the configured fallback recipient.

AC-2: Given a breach occurs,
      when the dashboard renders the escalation,
      then a red badge highlights it and the elapsed minutes are shown.

AC-3: Given multiple breaches in 24 hours,
      when the daily digest is generated,
      then it summarizes counts by category and assignee.

## Definition of done

- [ ] Email fallback configurable per customer.
- [ ] Digest email matches brand styling.
- [ ] SLA breach metric exposed in CloudWatch.
