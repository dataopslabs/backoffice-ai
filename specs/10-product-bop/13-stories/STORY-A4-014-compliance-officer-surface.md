---
id: STORY-A4-014
title: Compliance officer audit-export surface
type: story
status: draft
owner: founder
depends-on: [STORY-A4-009]
covers-req: [REQ-PRD-014]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-014 — Compliance officer surface

## Story

As a compliance officer, I need a dedicated page that shows policy version history, recent runs
against each version, and a one-click audit export with date range and workflow filters.

## Acceptance criteria

AC-1: Given the compliance officer opens `/compliance`,
      when the page loads,
      then they see a timeline of policy bundle versions with associated run counts.

AC-2: Given they click a policy version,
      when it expands,
      then they see runs that executed against it with a summary of decisions and exceptions.

AC-3: Given they configure an export (date range, workflow, version),
      when they submit,
      then the audit export pipeline (`STORY-A4-009`) is triggered with the filter applied.

## Definition of done

- [ ] All ACs covered.
- [ ] Page restricted to `customer:write` + a `compliance` group claim.
