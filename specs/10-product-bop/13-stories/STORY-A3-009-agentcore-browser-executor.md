---
id: STORY-A3-009
title: AgentCoreBrowserExecutor adapter (fallback)
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-C4-005, ADR-011]
covers-req: []
bdd: [BDD-A3-009.feature]
tests: [tests/adapters/test_agentcore_browser_executor.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-009 — AgentCoreBrowserExecutor

## Story

As the founder, I need a second implementation of `ExecutorPort` against AgentCore Browser
that can run the same set of test workflows the Nova Act adapter handles. This is the fallback
executor for UAT and for cases where Nova Act is degraded.

## Acceptance criteria

AC-1: Given the same `Step` the Nova Act adapter accepts,
      when executed by AgentCore Browser,
      then it returns an equivalent `StepResult` (status, observations, screenshots).

AC-2: Given a workflow declares a capability `dom-replay`,
      when the router considers executors,
      then it routes to AgentCore Browser (Nova Act does not advertise it).

AC-3: Given a Nova Act health probe reports `degraded`,
      when a new run starts,
      then the router falls back to AgentCore Browser.

AC-4: Given the same workflow runs against both executors in turn,
      when results are compared,
      then status and observations are within a defined tolerance (allow minor formatting
      differences); deviations cause an alert.

## Technical notes

- AgentCore Browser session provisioned per run; torn down on commit.
- Playbook synthesized from the step intent using a small templating layer.
- Screenshot capture identical to Nova Act adapter.
- Health probe runs every 5 minutes and writes to a `executor_health` DDB record.

## Definition of done

- [ ] Contract tests against `ExecutorPort` pass.
- [ ] A/B test suite confirms parity on five reference workflows.
- [ ] Health-probe-driven routing demonstrated in CI.
