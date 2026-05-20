---
id: DESIGN-B2-001
title: AgentCore Runtime configuration for the reconciliation agent
type: design
status: approved
owner: founder
depends-on: [ADR-005, REQ-B1-001]
covers-req: [REQ-B1-001]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-B2-001 — AgentCore Runtime configuration

## Goal

Concrete configuration for hosting the BackOfficePilot reconciliation orchestrator on
AgentCore Runtime.

## Resource definition (illustrative IaC)

```python
RecAgent = agentcore.Agent(
    self, "AcmeReconAgent",
    name="bop-uat-acme-recon",
    runtime_image=ecr_image,                  # Python 3.12 + orchestrator code
    instance_profile=acme_recon_role,
    network=NetworkConfig(
        vpc=uat_vpc, subnets=isolated_subnets, security_group=recon_sg,
    ),
    memory=MemoryConfig(
        namespace="bop/uat/cust/acme",
        session_ttl_minutes=30,
        long_term_enabled=True,
    ),
    gateway=GatewayConfig(
        tools=[netsuite_export_tool, bank_csv_tool, slack_escalation_tool],
    ),
    registry=RegistryConfig(
        version_label="0.1.0",
        canary_capable=True,
    ),
    observability=ObservabilityConfig(
        otel_enabled=True,
        trace_sampling_rate=1.0,
        log_level="INFO",
    ),
    secrets=[
        SecretRef("cust/acme/bank/credentials"),
        SecretRef("cust/acme/netsuite/oauth"),
        SecretRef("cust/acme/slack/bot_token"),
    ],
    cost_budget=CostBudget(per_run_usd=1.00, per_day_usd=10.00),
)
```

## Sizing

- One agent instance per customer per workflow.
- For Acme MVP: 1 agent (this one).
- Resource: 1 vCPU, 2 GB RAM, ephemeral storage 8 GB.
- Concurrency: 3 simultaneous runs (only one is expected during the MVP daily window).

## Networking

- The agent sits in isolated subnets; egress only through NAT for customer-system reachability.
- VPC endpoints for Bedrock, Secrets Manager, KMS, S3, DynamoDB.

## Identity

- Agent instance profile has access only to:
  - `cust/acme/*` secrets
  - `s3://bop-uat-runs/cust=acme/*`
  - `s3://bop-uat-audit/cust=acme/*`
  - DDB items with PK leading `CUSTOMER#acme` or `RUN#acme_*`.

## Health

- AgentCore Runtime health probes hit `/healthz` on the orchestrator's HTTP surface.
- Unhealthy instance is rotated by Runtime within 90 seconds.

## Cost telemetry

- Runtime emits `AgentRuntimeTicked` per minute the agent is provisioned.
- Subscriber in the cost meter applies pricing × duration to compute Runtime cost component.

## Failure mode coverage

| Failure | Detection | Response |
|---|---|---|
| Image fails health check | AgentCore Runtime | Roll back to previous Registry version |
| Out of memory | OOM kill | Page operator; auto-restart |
| Egress blocked by misconfig | Network probe in `/healthz` | Mark unhealthy; pause workflows |
