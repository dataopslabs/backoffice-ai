---
id: STORY-B4-008
title: Vendor-name fuzzy match tuning and per-customer learning
type: story
status: draft
owner: founder
depends-on: [DESIGN-B2-003]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-008 — Fuzzy match tuning + learning loop

## Story

As a customer, I need vendor-name fuzzy match to improve over time as I correct misclassifications,
so reconciliation accuracy converges upward.

## Acceptance criteria

AC-1: Given an operator corrects a fuzzy-match miss,
      when the correction is saved,
      then a vendor-alias entry is added to long-term memory.

AC-2: Given subsequent runs,
      when an inbound vendor string matches the alias,
      then matching succeeds without manual review.

AC-3: Given a customer admin opens the alias map,
      when they review,
      then they can edit or remove aliases; changes are audit-logged.

## Definition of done

- [ ] Learning loop wired through `MemoryPort`.
- [ ] Alias map editable from dashboard.
- [ ] Effectiveness measured: 90-day accuracy lift attributable to the learning loop.
