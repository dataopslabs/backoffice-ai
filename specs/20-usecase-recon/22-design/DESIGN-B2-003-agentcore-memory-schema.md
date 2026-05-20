---
id: DESIGN-B2-003
title: AgentCore Memory schema
type: design
status: approved
owner: founder
depends-on: [ADR-005, DESIGN-DATA-002]
covers-req: [REQ-B1-001]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-B2-003 — Memory schema

## Goal

Define what state lives in AgentCore Memory for the reconciliation agent, in three tiers:
session (per-run), project (per-customer-workflow), long-term (per-customer).

## Tier 1: Session (per-run)

| Key | Type | Purpose |
|---|---|---|
| `run.id` | string | Active run identifier |
| `run.plan` | json | The current plan (immutable per run) |
| `run.current_step_id` | string | Step the agent is mid-execution on |
| `run.observations` | json (per step) | Step results so far |
| `run.exceptions` | array | Exceptions raised in this run |
| `run.cost_usd_so_far` | number | Running cost tally |
| `run.policy_bundle_version` | string | Version pinned for this run |

TTL: run duration + 30 minutes. Garbage-collected after commit.

## Tier 2: Project (per-customer-workflow)

| Key | Type | Purpose |
|---|---|---|
| `workflow.definition` | yaml-parsed | Active workflow definition |
| `workflow.policy_bundle` | markdown bundle | Active policy text (cached for Reasoner prompt) |
| `workflow.recent_run_ids` | array (last 100) | For audit cross-reference |
| `workflow.executor_health` | json | Last health probe results for routing |
| `workflow.last_run_at` | timestamp | Used by schedule monitor |

TTL: indefinite while workflow is active; cleared on workflow deletion.

## Tier 3: Long-term (per-customer)

| Key | Type | Purpose |
|---|---|---|
| `customer.vendor_alias_map` | json | Learned vendor-name alias mappings |
| `customer.tolerance_overrides` | json | Per-vendor tolerance overrides set by the customer |
| `customer.preferred_channels` | json | Escalation channel preferences |
| `customer.cost_baseline_usd_per_hr` | number | Labor baseline for ROI calculation |

TTL: indefinite. Customer-data; subject to BYOK if customer is on enterprise tier.

## Access patterns

- Reads: most frequent are `workflow.definition` and `workflow.policy_bundle` (every run).
- Writes: session tier writes per-step; project tier writes per-run; long-term writes are rare
  (operator or learning loop).

## Failure modes

| Failure | Detection | Response |
|---|---|---|
| Memory namespace inaccessible | AWS error on read | Pause run; alert |
| Session record exceeds size cap | API error 4xx | Compact observations; persist deltas to S3 |
| Stale project tier (workflow updated mid-run) | Snapshot semantics (REQ-PRD-004) | Use cached snapshot; refresh on next run |

## Privacy

- Session memory may contain PII (raw transaction lines): redacted before any log emission.
- Long-term memory is per-customer; no cross-customer learning by default. Cross-customer
  insight is opt-in (planned in v2).
