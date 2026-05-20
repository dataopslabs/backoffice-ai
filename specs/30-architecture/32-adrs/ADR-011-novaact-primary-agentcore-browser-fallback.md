---
id: ADR-011
title: Nova Act is the primary Executor; AgentCore Browser is the fallback
type: adr
status: approved
owner: founder
depends-on: [ADR-004]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-011: Nova Act primary, AgentCore Browser fallback

## Context

Two execution surfaces are viable:

- **Amazon Nova Act**: a production-grade UI execution agent, $4.75/agent-hour, GA Dec 2025.
  Designed for unattended workflows.
- **AgentCore Browser**: a sandboxed browser-host service. Suited to deterministic page
  scripting and headless testing. Less mature for full unattended automation.

Both can drive the same kinds of customer systems. They overlap.

## Decision

- **Primary**: every workflow's `ExecutorPort` is implemented by `NovaActExecutor`.
- **Fallback**: `AgentCoreBrowserExecutor` is provisioned for every workflow as a backup.
  The orchestrator routes to fallback in these cases:
  1. Nova Act SDK quota exhausted or service degraded.
  2. The customer system is one Nova Act does not handle (rare; logged for product feedback).
  3. A dev/test environment where reproducible browser scripting is desirable.
- **Dev/test default**: AgentCore Browser is the default Executor in UAT for repeatability;
  promotion to Prod implicitly switches to Nova Act.

## Alternatives considered

- **AgentCore Browser primary**: cleaner AgentCore alignment, but loses the Nova Act SLA story
  that the proposal leans on.
- **Nova Act only, no fallback**: simpler, but a single Nova Act outage halts all production
  workflows.
- **Bring our own Playwright**: builds on widely-known tooling but loses both AWS contracts and
  the resilience the dual-executor pattern provides.

## Consequences

**Positive**

- Production gets Nova Act reliability; failure modes get AgentCore Browser as bridge.
- Tests can run on AgentCore Browser without spending Nova Act credits.
- The dual implementation forces the `ExecutorPort` interface to remain truly vendor-agnostic.

**Negative**

- Two adapters to maintain.
- Routing logic in the orchestrator adds a small amount of complexity. We isolate it in a
  `ExecutorRouter` class behind the port, so swapping the policy is a one-class change.
- Behavior may differ in edge cases between executors. Integration tests run against both.

## Routing policy

```python
def select_executor(workflow, run_context) -> ExecutorPort:
    if run_context.environment == "uat":
        return agentcore_browser_executor
    if novaact_health.is_degraded():
        return agentcore_browser_executor
    if workflow.requires_capability_absent_in_novaact():
        return agentcore_browser_executor
    return novaact_executor  # default
```

The capability matrix is data, not code: `workflows/<name>.yaml` may declare
`executor_requirements: [pdf-download, file-upload, csv-export]`; each adapter advertises its
capabilities and the router matches them.
