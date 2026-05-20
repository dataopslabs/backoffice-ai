---
id: REQ-PRD-005
title: Agent deployment, rollout, and rollback via AgentCore Registry
type: req
status: approved
owner: founder
depends-on: [ADR-005]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-005 — Agent rollout via AgentCore Registry

## Statement

The system **shall** deploy every agent to AgentCore Runtime via AgentCore Registry, **shall**
support canary and full rollouts, and **shall** support rollback to any previous registered
version within 60 seconds of operator action.

## Acceptance criteria

AC-1: Given a new agent build is ready,
      when the deploy pipeline pushes it,
      then a new Registry version is created and tagged with the build's git SHA.

AC-2: Given a Registry version exists,
      when the operator initiates a canary rollout at 10%,
      then 10% of new runs route to the new version; existing runs are unaffected.

AC-3: Given a canary in progress,
      when the operator promotes to 100%,
      then all subsequent runs use the new version and the previous version is kept as a
      rollback candidate for 30 days.

AC-4: Given a fault is detected after promotion,
      when the operator triggers rollback to the previous version,
      then within 60 seconds new runs use the previous version; the rollback is logged as an
      audit event.

## Implementation notes

- Registry interaction wrapped in `AgentRegistryAdapter` (`DESIGN-A2-003`).
- Canary routing controlled by an environment-scoped feature flag.
- Rollback is data-only (version pin); no rebuild required.
