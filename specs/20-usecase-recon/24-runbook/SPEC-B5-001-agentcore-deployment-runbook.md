---
id: SPEC-B5-001
title: AgentCore deployment runbook (UAT and Prod)
type: spec
status: approved
owner: founder
depends-on: [ADR-005, ADR-009, ADR-010, DESIGN-B2-001, DESIGN-B2-002, DESIGN-B2-003, DESIGN-B2-004, DESIGN-B2-005]
covers-req: [REQ-PRD-005, REQ-B1-002, REQ-B1-006]
version: 0.1.0
last-updated: 2026-05-19
---

# AgentCore Deployment Runbook

This runbook is the step-by-step procedure for standing up BackOfficePilot on AWS Bedrock
AgentCore for both UAT (first) and Prod (second). Designed to be executed by one engineer in
under one day per environment.

## Prerequisites

| Item | Required |
|---|---|
| AWS Organizations setup with `backofficepilot-prod`, `backofficepilot-uat`, `backofficepilot-ci` accounts | Y |
| Cross-account `Deployer` role in each target account, trusted from `backofficepilot-ci` | Y |
| Bedrock model access requested for `anthropic.claude-opus-4-6` in `us-east-1` | Y |
| Nova Act API key obtained (via accelerator) | Y |
| ECR repositories `bop-orchestrator`, `bop-mock-bank` in the CI account | Y |
| Domain registered (`backofficepilot.ai`) and Route 53 hosted zone in CI account | Y |
| `cdk.context.json` populated with account IDs and region | Y |

## Phase 1 — Networking (`infra/stacks/network_stack.py`)

### Steps

1. Confirm `cdk.context.json` binds the right account ID for the target env.
2. Run `cdk deploy --env uat NetworkStack` (then `--env prod` later).
3. Verify in console: VPC with `10.20.0.0/16` (UAT) or `10.10.0.0/16` (Prod), three AZs,
   public + private + isolated subnets.
4. Confirm VPC Flow Logs are streaming to S3.

### Verification

```bash
aws ec2 describe-vpcs --filters Name=tag:Project,Values=BackOfficePilot
```

Result must show exactly one VPC per environment.

## Phase 2 — Data stores (`infra/stacks/data_stack.py`)

### Steps

1. `cdk deploy --env uat DataStack`.
2. Verify resources:
   - DDB table `bop-uat-state` (single-table).
   - S3 buckets: `bop-uat-runs`, `bop-uat-audit` (with Object Lock compliance), `bop-uat-policy`.
   - KMS CMK `alias/bop-uat-cmk`.
   - Secrets Manager parameter `Cust/template` (for new-tenant provisioning).
3. Apply IAM Access Analyzer policy review.
4. Confirm AWS Config rules are evaluating.

### Verification

```bash
aws s3api get-object-lock-configuration --bucket bop-uat-audit
```

Must return mode `COMPLIANCE`, retention 7 years.

## Phase 3 — AgentCore (`infra/stacks/agentcore_stack.py`)

### Steps

1. `cdk deploy --env uat AgentCoreStack`.
2. Resources created:
   - AgentCore Runtime resource group `bop-uat-runtimes`.
   - AgentCore Gateway with empty tool registry.
   - AgentCore Memory namespace `bop/uat`.
   - AgentCore Registry namespace `bop/uat`.
   - AgentCore Browser pool with 3 warm sessions.
   - AgentCore Observability connection to CloudWatch GenAI Observability dashboards.
3. Verify Bedrock model access from the AgentCore Runtime role:
   ```bash
   aws bedrock-runtime invoke-model --model-id anthropic.claude-opus-4-6 --body ... --region us-east-1
   ```
4. Push the orchestrator image to ECR; create initial Registry version `0.1.0`.

### Verification

- Visit AgentCore console; confirm all six services show "Healthy" in the UAT region.
- Use `scripts/smoke_agentcore.py` to call each service with a no-op request.

## Phase 4 — Per-customer provisioning (Acme)

### Steps

1. Run `scripts/onboard_customer.py --customer-id acme --tier pilot --env uat`.
   This idempotently:
   - Creates customer record in DDB.
   - Provisions per-customer IAM role with prefix scopes.
   - Creates Secrets Manager paths under `cust/acme/*`.
   - Creates Cognito app client.
   - Creates S3 prefixes under `bop-uat-runs/cust=acme/` and `bop-uat-audit/cust=acme/`.
   - Creates per-customer KMS alias if BYOK tier (not for `pilot`).
   - Seeds AgentCore Memory namespace `bop/uat/cust/acme`.
2. Upload Acme's bank credentials and TOTP secret:
   ```bash
   aws secretsmanager put-secret-value --secret-id cust/acme/bank/credentials ...
   ```
3. Upload NetSuite OAuth refresh token:
   ```bash
   aws secretsmanager put-secret-value --secret-id cust/acme/netsuite/oauth ...
   ```
4. Install BackOfficePilot Slack app to Acme's workspace; capture bot token; store under
   `cust/acme/slack/bot_token`.

## Phase 5 — Policy bundle + workflow

### Steps

1. Run `scripts/upload_policy_bundle.py --customer-id acme --bundle policy/acme/2026-05`.
   The script validates and uploads; new version is recorded.
2. Activate the bundle: `scripts/activate_policy_bundle.py --customer-id acme --version 1.0.0`.
3. Run `scripts/upsert_workflow.py --customer-id acme --file workflows/daily_reconciliation.yaml`.
4. Confirm validator accepts; version `1.0.0` is recorded.

## Phase 6 — Gateway tool registration

### Steps

1. `cdk deploy --env uat GatewayToolsStack` deploys `infra/gateway/tools/*.yaml`.
2. Verify in AgentCore Gateway console: `netsuite_export`, `bank_csv_download`,
   `slack_escalation` tools exist with the right schemas.
3. Use the Gateway test bench to invoke each tool once with sample input.

## Phase 7 — First end-to-end run

### Steps

1. Trigger a manual run:
   ```bash
   curl -X POST https://api.uat.backofficepilot.ai/customers/acme/workflows/daily_reconciliation/runs \
     -H "Authorization: Bearer <operator-jwt>" \
     -H "Idempotency-Key: $(uuidgen)" \
     -d '{"trigger":"manual"}'
   ```
2. Watch the run in the dashboard:
   `https://app.uat.backofficepilot.ai/runs/<run-id>`.
3. Confirm:
   - All steps complete.
   - At least one synthetic exception escalates to Slack.
   - Total cost < $1.00.
   - Run bundle uploaded with hash chain valid.
4. Run `scripts/verify_audit.py s3://bop-uat-audit/cust=acme/.../audit.jsonl`.
   Must return `chain OK`.

## Phase 8 — Observability + alarms

### Steps

1. `cdk deploy --env uat ObservabilityStack` deploys CloudWatch dashboards + alarms.
2. Open the "Recon Agent Health" dashboard; confirm metrics flowing.
3. Fire a test alarm by simulating high latency in `scripts/synthetic_alarm.py`.
4. Confirm SNS notification reaches the operator pager.

## Phase 9 — Schedule + 5-day unattended run

### Steps

1. Confirm the cron schedule fires at 07:00 CT next weekday.
2. Observe runs for 5 consecutive business days.
3. Document observations in `evidence/uat-five-day-run/`.
4. If criteria from `REQ-B1-005` are met, mark `STORY-B3-011` done.

## Promotion to Prod

### Pre-flight

- All UAT criteria met.
- Rollback drill rehearsed in UAT (`DESIGN-B2-004`).
- Customer pilot agreement signed.
- Operator on-call rotation defined.

### Steps

1. Repeat Phases 1–8 with `--env prod`.
2. Begin with read-only dry-run on the customer's real NetSuite (per `STORY-B4-002`).
3. Gradually unlock posting; per-JE approval for 3 days.
4. Promote to full autonomy on operator + customer sign-off.

## Disaster recovery

### RTO/RPO targets

- RTO: 4 hours from outage detection to running in secondary region (`us-west-2`).
- RPO: ≤ 30 minutes of run-state loss.

### Procedure

1. Operator declares incident, posts to status page.
2. Trigger failover stack `cdk deploy --env prod-dr` (pre-staged stacks in `us-west-2`).
3. Update Route 53 to point API to `us-west-2`.
4. Resume scheduled triggers in DR region.
5. Customer notified; post-mortem within 5 business days.

### Failback

1. Confirm primary region healthy.
2. Replay any DR-region-only runs into primary audit store.
3. Switch Route 53 back.
4. Disable DR-region stacks until next drill.

## Common failures + responses

| Failure | Symptom | First action |
|---|---|---|
| Bedrock quota exhausted | Bedrock 429s in trace | Page operator; raise quota via AWS support |
| NetSuite session timeout | Repeated `auth_expired` errors | Verify Secrets Manager has valid OAuth refresh token |
| Nova Act region unavailable | Nova Act API 5xx for > 5 min | Router fails over to AgentCore Browser automatically; verify |
| AgentCore Memory throttling | Memory adapter timeouts | Inspect CloudWatch; raise namespace concurrency limit |
| Object Lock denied delete (expected) | `AccessDenied` on `DeleteObject` | This is correct behavior; investigate why a delete was attempted |
| Pricing.yaml stale | Cost reconciliation alarm > 5% variance | Update `config/pricing.yaml`; PR with variance report attached |
| Trace gap on a run | Some spans missing in GenAI Observability | Inspect OTel exporter; restart if needed; runs aren't blocked |

## Change management

- Every deploy is logged to `deploys/YYYY-MM-DD.md` in this repo.
- Major version changes to `pricing.yaml`, workflow YAML, or policy bundles require an ADR or
  spec change.
- Quarterly review of the runbook itself; staleness is the enemy of recovery.
