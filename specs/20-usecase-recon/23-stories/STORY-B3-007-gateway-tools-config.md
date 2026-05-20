---
id: STORY-B3-007
title: Configure AgentCore Gateway tools (NetSuite, bank, Slack)
type: story
status: approved
owner: founder
depends-on: [STORY-B3-006, DESIGN-B2-002]
covers-req: []
bdd: [BDD-B3-007.feature]
tests: [tests/gateway/test_tools_register.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-007 — Gateway tool configs

## Story

As the founder, I need the three Gateway tool definitions registered and reachable from the
orchestrator: `netsuite_export`, `bank_csv_download`, `slack_escalation`.

## Acceptance criteria

AC-1: Given `infra/gateway/tools/*.yaml` exist,
      when `cdk deploy` runs,
      then all three tools register in AgentCore Gateway.

AC-2: Given a Bedrock invocation that produces a tool call for `netsuite_export`,
      when the orchestrator dispatches it through Gateway,
      then Nova Act (primary) executes; on health degradation AgentCore Browser executes.

AC-3: Given a tool call for `slack_escalation`,
      when invoked,
      then a message posts to the configured channel; thread_ref returned.

AC-4: Given tool definitions need an update,
      when YAML is changed and `cdk deploy` runs,
      then versions advance; the orchestrator picks up the new schemas on next pull.

## Definition of done

- [ ] All three tools registered, callable, and tested end-to-end.
- [ ] IAM execution roles scoped per tool.
- [ ] Tool-definition change-management documented in the runbook.
