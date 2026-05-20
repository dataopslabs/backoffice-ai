---
id: DESIGN-A2-002
title: Workflow definition format and versioning
type: design
status: approved
owner: founder
depends-on: [REQ-PRD-004]
covers-req: [REQ-PRD-004, REQ-PRD-006]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-A2-002 — Workflow definition format and versioning

## Goal

Define the YAML grammar customers (or operators on behalf of customers) use to author
workflows, the validator that enforces it, and the versioning semantics that protect in-flight
runs.

## YAML grammar (illustrative)

```yaml
name: daily_reconciliation
version: 1.4.2                    # set automatically on upsert; rejected if hand-specified
triggers:
  cron: "0 13 * * MON-FRI"
  timezone: "UTC"                 # optional, defaults to UTC

policy_documents:
  - policy/three_way_match.md
  - policy/journal_posting_rules.md

systems:
  - id: bank_portal
    type: web
    base_url: ${BANK_PORTAL_URL}  # env-style template, resolved at run-start
    credentials_secret: bank/creds
    capabilities_required: [pdf-download]

  - id: netsuite
    type: web
    base_url: ${NETSUITE_URL}
    credentials_secret: netsuite/creds

executor_requirements: [pdf-download, file-upload]

steps:
  - id: pull_bank_file
    system: bank_portal
    intent: "Download yesterday's transaction file"
    success: { file_present: true, min_rows: 20 }
    timeout_seconds: 60

  - id: pull_open_invoices
    system: netsuite
    intent: "Export open invoices for the last 60 days"
    timeout_seconds: 90

  - id: match
    type: reason
    inputs: [pull_bank_file, pull_open_invoices]

  - id: post_matches
    system: netsuite
    intent: "Post journal entries for matched transactions"

  - id: escalate_exceptions
    type: escalate
    channel: slack

exception_categories:
  - amount_mismatch
  - missing_invoice
  - duplicate_payment
  - currency

budget:
  max_cost_per_run_usd: 1.00
  max_duration_minutes: 15

escalation:
  channel: "#bop-reconciliation"
  reviewer_sla_minutes: 60
  fallback: email
```

## JSON Schema

Lives at `30-architecture/34-api-contract/schemas/workflow.schema.json`. Generated from a
Pydantic `WorkflowDefinition` model that doubles as the runtime parser.

## Versioning rules

| Change kind | Version bump |
|---|---|
| Add an optional field, add a new step at end, add a new category | patch |
| Reorder steps, change tolerances, modify trigger schedule | minor |
| Remove a step, change a system id, restructure | major |

Each `upsertWorkflow` call computes the bump from the diff against the prior version and rejects
if the operator-declared bump is too low.

## Snapshot semantics

When a run starts, the workflow definition + policy bundle hashes are recorded on the `Run`
record. The run executes against that snapshot for its lifetime, even if the definition is
updated mid-run. This is what makes versioning safe in production.

## Validation pipeline

1. YAML parse (PyYAML safe loader).
2. JSON Schema validate.
3. Pydantic model construction (type coercion).
4. Cross-field validation:
   - Every `step.system` references a declared `systems[].id`.
   - Every `inputs:` references a prior `step.id`.
   - Every `policy_documents[]` exists in the customer's policy bundle.
   - `budget.max_cost_per_run_usd` ≤ customer's plan ceiling.
5. Persist as new version; emit `WorkflowUpserted` event.

Failures at any step return a structured error list, not a single message.

## Backward compatibility for orchestrators

Workflow YAML carries a `schema_version` field (separate from `version`). Orchestrators tolerate
older `schema_version` for at least two minor releases. Upgrades to the schema follow ADR
process.
