---
id: STORY-A4-003
title: AgentCore Registry canary and rollback flow
type: story
status: draft
owner: founder
depends-on: [STORY-A3-010, DESIGN-A2-003]
covers-req: [REQ-PRD-005]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-003 — Registry canary + rollback

## Story

As the founder, I need the canary and rollback flow per `DESIGN-A2-003` implemented end-to-end,
including the auto-rollback rule, the canary metrics dashboard, and the one-click rollback
operator UI.

## Acceptance criteria

AC-1: Given a new version is pushed,
      when an operator initiates a 10% canary,
      then 10% of new runs route to the new version; metrics dashboard shows side-by-side
      health.

AC-2: Given the auto-rollback rule sees > 5% canary errors within 30 minutes,
      when the threshold trips,
      then the canary is reverted automatically; the operator is paged.

AC-3: Given a stable canary,
      when an operator promotes to 100%,
      then all new runs use the new version; old version is retained 30 days.

AC-4: Given an operator clicks Rollback,
      when the action is confirmed,
      then within 60 seconds new runs use the previous version; audit log captures the action.

## Definition of done

- [ ] Auto-rollback rule active.
- [ ] Canary metrics dashboard exists.
- [ ] One-click rollback works end-to-end.
