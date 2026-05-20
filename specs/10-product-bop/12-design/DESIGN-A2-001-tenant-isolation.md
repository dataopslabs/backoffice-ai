---
id: DESIGN-A2-001
title: Tenant isolation design
type: design
status: approved
owner: founder
depends-on: [ADR-009, REQ-PRD-001]
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-A2-001 — Tenant isolation design

## Goal

Translate `REQ-PRD-001` into concrete naming conventions, IAM templates, and code patterns that
make cross-tenant access either impossible or detectable.

## Per-tenant resource naming

| Resource | Pattern |
|---|---|
| DDB partition keys | `CUSTOMER#{id}`, `RUN#{id}`, ... — tenant ID is the first segment |
| S3 paths | `s3://bop-<env>-{purpose}/cust={id}/...` |
| Secrets Manager | `cust/{id}/{system}/{secret_name}` |
| KMS aliases | `alias/bop-{env}-cust-{id}` (BYOK tier) |
| AgentCore Memory namespace | `bop/{env}/cust/{id}` |
| OpenSearch index | `bop-{env}-cust-{id}-runs` |
| EventBridge custom bus | shared per environment; events carry `customer_id` for routing |
| Cognito app client | one per customer |

## IAM policy template

A CDK construct `PerCustomerIamRole` generates:

```python
PerCustomerIamRole(self, "AgentRole",
    customer_id="acme",
    allowed_secret_prefix=f"cust/acme/*",
    allowed_s3_prefix=f"s3://bop-prod-runs/cust=acme/*",
    allowed_ddb_lead_keys=["CUSTOMER#acme", "RUN#acme_*"],
    allowed_kms_alias="alias/bop-prod-cust-acme",
)
```

The construct emits a role with statements like:

```json
{
  "Effect": "Allow",
  "Action": ["secretsmanager:GetSecretValue"],
  "Resource": "arn:aws:secretsmanager:*:*:secret:cust/acme/*"
}
```

Any attempt to read another customer's secret is denied at the IAM layer, with CloudTrail
evidence.

## Code-level safeguards

- Domain repositories accept a mandatory `tenant_id` parameter; constructors reject `None`.
- A `TenantContext` middleware on FastAPI extracts the customer ID from the JWT and binds it to
  the request context. Repositories pull tenant ID from the context, never from request body.
- A custom mypy plugin (or pyright config) flags repository calls that bypass the context.

## Operator role exception

Operator users (`operator` scope) can read across tenants. Every operator action is logged with:
- Actor email/ID.
- Action and target tenant.
- Justification (free text required for write actions).

Operator actions are visible to affected customers in a "Operator activity" audit view.

## Failure modes considered

| Failure | Detection | Recovery |
|---|---|---|
| Misconfigured workflow references another tenant's policy bundle | Validation rejects at upsert | None — never deployed |
| Cross-tenant DDB scan attempted | IAM denies; CloudTrail event | Page on-call |
| Operator forgets to scope an action to one tenant | Action logged across tenants; review surfaces it | Quarterly operator review |
| Bug in repository binding misuses tenant context | Property-based test fails CI | N/A — caught pre-merge |

## Test coverage

Required tests for any new repository:

1. Tenant `A` cannot read tenant `B`'s records.
2. Tenant ID is mandatory; constructor raises on omission.
3. IAM policy synth-test enforces resource prefix.
