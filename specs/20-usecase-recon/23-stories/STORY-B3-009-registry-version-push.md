---
id: STORY-B3-009
title: Push initial agent version to AgentCore Registry
type: story
status: approved
owner: founder
depends-on: [STORY-B3-006, DESIGN-B2-004]
covers-req: [REQ-PRD-005]
bdd: []
tests: [tests/infra/test_registry.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-009 — Registry version push

## Story

As the founder, I need version `0.1.0` of the recon orchestrator image pushed to AgentCore
Registry with full metadata, and the canary auto-rollback rule active.

## Acceptance criteria

AC-1: Given the CI pipeline builds the orchestrator image,
      when the deploy step runs,
      then the image is pushed to ECR and a corresponding Registry version is created.

AC-2: Given a canary attempt with > 5% errors in 30 minutes,
      when the auto-rollback rule trips,
      then Registry reverts to the previous version automatically.

AC-3: Given an operator manual rollback,
      when invoked from the dashboard,
      then within 60 seconds the active version reverts.

## Definition of done

- [ ] Registry version `0.1.0` registered and active in UAT.
- [ ] Auto-rollback rule deployed and tested against an artificial failure.
- [ ] Operator runbook covers the rollback flow.
