---
id: DESIGN-B2-002
title: AgentCore Gateway tool definitions
type: design
status: approved
owner: founder
depends-on: [ADR-005, REQ-B1-002]
covers-req: [REQ-B1-002]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-B2-002 — Gateway tool definitions

## Goal

Define the three tools exposed via AgentCore Gateway for the reconciliation workflow:
`netsuite_export`, `bank_csv_download`, and `slack_escalation`.

## Tool: `netsuite_export`

```yaml
name: netsuite_export
description: |
  Logs into NetSuite, navigates to the specified report, exports it as CSV,
  uploads the CSV to S3, and returns the S3 URI.
input_schema:
  type: object
  required: [report, date_range]
  properties:
    report: { type: string, enum: [open_invoices, recent_payments, vendor_master] }
    date_range:
      type: object
      properties:
        from: { type: string, format: date }
        to: { type: string, format: date }
output_schema:
  type: object
  required: [s3_uri, row_count, executor]
  properties:
    s3_uri: { type: string, format: uri }
    row_count: { type: integer }
    executor: { type: string, enum: [novaact, agentcore_browser] }
implementation:
  type: executor_call
  primary: novaact
  fallback: agentcore_browser
  capability_requirements: [web-login, csv-export, two-factor-auth]
timeout_seconds: 120
```

## Tool: `bank_csv_download`

```yaml
name: bank_csv_download
description: |
  Logs into the Acme bank treasury portal, navigates to the daily transactions page,
  downloads the CSV for the requested date, uploads to S3, returns the S3 URI.
input_schema:
  type: object
  required: [transaction_date]
  properties:
    transaction_date: { type: string, format: date }
output_schema:
  type: object
  required: [s3_uri, row_count]
  properties:
    s3_uri: { type: string, format: uri }
    row_count: { type: integer }
implementation:
  type: executor_call
  primary: novaact
  fallback: agentcore_browser
  capability_requirements: [web-login, file-download, two-factor-auth]
timeout_seconds: 60
```

## Tool: `slack_escalation`

```yaml
name: slack_escalation
description: |
  Posts an escalation message to the customer's Slack channel with Approve/Reject buttons.
input_schema:
  type: object
  required: [channel, exception_id, narrative]
  properties:
    channel: { type: string }
    exception_id: { type: string }
    narrative: { type: string }
    screenshots: { type: array, items: { type: string, format: uri } }
output_schema:
  type: object
  required: [thread_ref]
  properties:
    thread_ref: { type: string }
implementation:
  type: lambda_invoke
  function_arn: arn:aws:lambda:us-east-1:<acct>:function:bop-uat-slack-escalator
timeout_seconds: 10
```

## Authorization

Each tool's IAM execution role is scoped to:

- `netsuite_export`, `bank_csv_download`: invoke the configured Executor on behalf of the agent;
  read secrets matching `cust/acme/*`; write to `s3://bop-uat-runs/cust=acme/*`.
- `slack_escalation`: invoke the Slack-escalator Lambda; that Lambda holds the bot token.

## Versioning

Tool definitions live as YAML in `infra/gateway/tools/` and are versioned alongside the
Gateway resource. Adding a new tool is a config change, not a code change.
