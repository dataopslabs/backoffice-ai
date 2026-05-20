---
id: STORY-B3-006
title: Deploy recon orchestrator to AgentCore Runtime
type: story
status: approved
owner: founder
depends-on: [STORY-A3-010, DESIGN-B2-001]
covers-req: []
bdd: []
tests: [tests/infra/test_agentcore_stack.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-006 — AgentCore Runtime deployment

## Story

As the founder, I need the reconciliation orchestrator deployed to AgentCore Runtime in UAT per
`DESIGN-B2-001`, registered in Registry, with healthy `/healthz` and emitting traces.

## Acceptance criteria

AC-1: Given `cdk deploy --env uat AgentCoreStack` succeeds,
      when AWS console is inspected,
      then an Agent named `bop-uat-acme-recon` exists, healthy.

AC-2: Given a synthetic ping,
      when the orchestrator's `/healthz` is hit,
      then it returns 200 with the active workflow version, policy bundle version, and
      orchestrator code SHA.

AC-3: Given a manual `startRun` call,
      when issued,
      then the run completes against the mock + sandbox systems end-to-end and emits all
      expected events.

AC-4: Given traces in CloudWatch GenAI Observability,
      when filtered to the test run's trace ID,
      then spans for plan, execute (each step), reconcile, commit are present.

## Definition of done

- [ ] Agent provisioned and healthy.
- [ ] First manual run reaches `succeeded` status.
- [ ] Trace continuity verified end-to-end.
