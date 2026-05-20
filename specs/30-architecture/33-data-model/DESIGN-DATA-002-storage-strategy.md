---
id: DESIGN-DATA-002
title: Storage strategy and tier choices
type: design
status: approved
owner: founder
depends-on: [DESIGN-DATA-001, ADR-009]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# Storage strategy and tier choices

## Storage tiers in use

| Tier | Service | Purpose | Encryption | Retention |
|---|---|---|---|---|
| **Hot state** | DynamoDB (on-demand) | Run state, exception queue, cost rollups, workflow metadata. | KMS CMK | 90 days, then archived |
| **Immutable archive** | S3 + Object Lock (compliance mode) | Audit logs, run bundles, screenshots. | KMS CMK | 7 years default; per-customer configurable |
| **Search index** | OpenSearch Serverless | Run history search, exception text search. | KMS CMK | Mirrors DDB hot state retention |
| **Long-term context** | AgentCore Memory | Customer-specific long-term agent knowledge. | AWS-managed | Lifecycle controlled by AgentCore |
| **Configuration** | S3 (versioned bucket) | Policy bundles, workflow YAML history. | KMS CMK | Indefinite, with object versioning |
| **Secrets** | AWS Secrets Manager | Customer-system credentials. | AWS-managed | Rotation policy per secret type |
| **Operational metrics** | CloudWatch Logs + Metrics | App logs, infra metrics. | AWS-managed | 30 days hot, 13 months cold |
| **Trace data** | AgentCore Observability + CloudWatch GenAI Observability | Agent traces, model usage, derived cost events. | AWS-managed | 90 days |

## Why DynamoDB for hot state

- Predictable per-operation latency.
- Per-customer partition keys make tenant isolation trivial.
- TTL attributes let us auto-expire short-lived items (e.g. stale escalation locks).
- Streams feed event-bridge for cross-process subscribers (`ADR-006`).

## Why S3 + Object Lock for audit

- Object Lock with compliance mode prevents deletion even by the account root user.
- Cheap at retention scale.
- Native integration with Athena for ad-hoc audit queries.
- A signed URL is the standard way to hand a customer their run bundle.

## Why OpenSearch (not DynamoDB) for search

- DDB query patterns are predefined. Full-text search across run history is not.
- OpenSearch Serverless removes operational burden at our scale.

## Why AgentCore Memory (not DDB) for long-term context

- AgentCore Memory has agent-aware semantics (session vs project vs long-term).
- Tight integration with AgentCore Runtime; no custom wiring.
- Recall/forget operations are native, not bolted on.

## Data residency

- All stores live in `us-east-1` for v1.
- Per-customer residency override planned for v2: a `customer.region` field will route stores to
  the customer's preferred region. The architecture supports it; we don't pay the cost until a
  customer requires it.

## Encryption keys

- **Default**: a single KMS CMK per environment (`bop-prod-cmk`, `bop-uat-cmk`).
- **Customer-managed (BYOK)**: enterprise tier may supply their own CMK. Our adapter wraps
  per-customer KMS aliases.
- **AgentCore Memory** uses AWS-managed keys; BYOK availability is on the AWS roadmap and we
  will adopt it when GA.

## Backup posture

- DynamoDB: PITR enabled, 35-day window.
- S3: versioning enabled on all buckets; cross-region replication to `us-west-2` for the audit
  bucket only (regulator scenario).
- OpenSearch: weekly snapshots to S3.
- Secrets Manager: rotation-on-create policy; cross-region replication for break-glass secrets.

## Cost shape (modeling, monthly, per customer at typical pilot scale)

| Component | Estimate |
|---|---|
| DynamoDB (~50K runs/mo at 5 items each) | $5 |
| S3 standard + Object Lock (~50 GB) | $2 |
| OpenSearch Serverless (small) | $90 |
| AgentCore Memory (per AWS pricing) | $30 |
| CloudWatch retention | $10 |
| KMS | $1 |
| Secrets Manager | $5 |
| **Subtotal storage / customer / month** | **~$143** |

Negligible against the per-run inference + execution cost. Storage is not the cost driver.

## Migration discipline

- Schema changes are backwards-compatible by default; breaking changes get a major version bump.
- DDB single-table changes: never break existing access patterns; add new GSIs rather than
  modifying existing ones.
- S3 keys are versioned in the bucket; new schema versions live alongside old; readers tolerate
  both.
- A `schema_version` field is present on every persisted record.

## What we explicitly do not use

- **RDS / Aurora**: relational store is unnecessary at this scale and adds operational surface.
  Revisit if/when reporting workloads need it.
- **ElastiCache / Redis**: no caching tier in v1. Latency budget does not require it.
- **Kinesis Data Streams**: EventBridge is sufficient for our event volume.
- **Glue / EMR**: no batch analytics in v1.

Any addition of the above requires an ADR with concrete justification.
