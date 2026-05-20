---
id: STORY-B3-008
title: AgentCore Memory namespace + initial seeds
type: story
status: approved
owner: founder
depends-on: [STORY-B3-006, DESIGN-B2-003]
covers-req: []
bdd: []
tests: [tests/adapters/test_memory_adapter.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-008 — Memory namespace

## Story

As the founder, I need AgentCore Memory namespace `bop/uat/cust/acme` provisioned and seeded
with the initial project tier (workflow definition + policy bundle) and long-term tier (empty
vendor alias map).

## Acceptance criteria

AC-1: Given the namespace exists,
      when the orchestrator starts a run,
      then it reads `workflow.definition` and `workflow.policy_bundle` from project tier.

AC-2: Given a step completes,
      when its observations are saved,
      then they appear in session tier with TTL.

AC-3: Given a run commits,
      when garbage collection runs,
      then session tier records for that run are cleared.

AC-4: Given an operator updates the long-term tier (e.g. adds a vendor alias),
      when subsequent runs occur,
      then the alias is respected by fuzzy match.

## Definition of done

- [ ] Namespace provisioned in CDK.
- [ ] Seed script idempotent.
- [ ] All four ACs covered by tests.
