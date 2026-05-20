---
id: STORY-A4-016
title: AgentCore Identity adapter
type: story
status: draft
owner: founder
depends-on: [ADR-005]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-016 — AgentCore Identity adapter

## Story

As an enterprise customer, I need end-user identity propagated through agent calls so that an
action taken by an agent is attributable to "agent X acting on behalf of end-user Y."

## Acceptance criteria

AC-1: Given a customer has enabled identity propagation,
      when a manual run is triggered by user Y,
      then every Bedrock / AgentCore Gateway / Nova Act call carries Y's identity context.

AC-2: Given a scheduled run,
      when it runs without a human trigger,
      then identity is `system:scheduled` and downstream calls are tagged accordingly.

AC-3: Given the audit log surface,
      when an event is reviewed,
      then the acting identity is visible at every step.

## Definition of done

- [ ] Identity context plumbed through orchestrator + adapters.
- [ ] Backward compatible: identity is optional in v1.
