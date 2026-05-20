---
id: STORY-B4-006
title: Customer-portal observability surfaces (per-agent cost view)
type: story
status: draft
owner: founder
depends-on: [STORY-B3-010, REQ-PRD-012]
covers-req: [REQ-PRD-012]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-006 — Customer-portal observability

## Story

As a customer, I need a per-agent observability page that mirrors the operator dashboard's
relevant sections (run volume, success rate, p95 latency, cost trend, exception trend),
filtered to my tenancy, so I can self-serve troubleshooting.

## Acceptance criteria

AC-1: Given the customer opens `/observability`,
      when authorized,
      then they see metrics for their agents only.

AC-2: Given a metric crosses a customer-visible threshold,
      when the page renders,
      then a contextual badge highlights it (e.g. "Higher than usual cost this week").

AC-3: Given a customer-facing alarm fires,
      when the customer is notified,
      then the email links to the relevant section of the observability page.

## Definition of done

- [ ] Page implemented in Next.js.
- [ ] Tenancy enforcement tested.
- [ ] Customer-side alarm thresholds documented.
