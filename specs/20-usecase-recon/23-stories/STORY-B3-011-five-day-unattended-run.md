---
id: STORY-B3-011
title: 5-day unattended UAT run + observation log
type: story
status: approved
owner: founder
depends-on: [STORY-B3-001, STORY-B3-002, STORY-B3-003, STORY-B3-004, STORY-B3-005, STORY-B3-006, STORY-B3-007, STORY-B3-008, STORY-B3-009, STORY-B3-010]
covers-req: [REQ-B1-005]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-011 — 5-day unattended UAT run

## Story

As the founder, I need a 5-business-day window where the reconciliation agent runs every
morning, unattended, with daily review of the resulting runs to confirm success criteria from
`REQ-B1-005` are met.

## Acceptance criteria

AC-1: Given the 5-day window,
      when each day's run completes,
      then all `REQ-B1-005` quantitative targets are met for that day.

AC-2: Given any failure during the window,
      when discovered,
      then the failure is fixed; the 5-day clock resets.

AC-3: Given the run log,
      when reviewed at the end of the window,
      then 5 consecutive successful runs are documented; the document is committed under
      `evidence/uat-five-day-run/`.

AC-4: Given any exception escalations during the window,
      when reviewed,
      then ≥ 90% are correctly categorized (per `REQ-B1-005`).

## Definition of done

- [ ] 5 consecutive successful runs.
- [ ] Observation log + run bundles archived under `evidence/`.
- [ ] No false-positive alarms over the window.
