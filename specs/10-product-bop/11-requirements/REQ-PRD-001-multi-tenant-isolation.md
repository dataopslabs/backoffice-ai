---
id: REQ-PRD-001
title: Multi-tenant isolation
type: req
status: approved
owner: founder
depends-on: [ADR-009]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-001 — Multi-tenant isolation

## Statement

The system **shall** ensure that no data, credential, or runtime artifact belonging to one
customer is ever accessible to another customer, under any failure mode or configuration error.

## Rationale

BackOfficePilot serves regulated BFSI institutions whose data cannot mingle. Tenant isolation is
not a feature; it is a precondition for sale. A single cross-tenant data exposure ends the
company.

## Acceptance criteria

AC-1: Given two customers A and B with active workflows,
      when an operator queries any DDB table without a customer filter,
      then the IAM policy denies the call and emits a CloudTrail event.

AC-2: Given a misconfigured workflow definition that references another customer's policy
      bundle URI,
      when the orchestrator loads the workflow,
      then validation rejects the workflow before any agent runtime is created.

AC-3: Given a run for customer A,
      when the agent attempts to read a secret,
      then the secret name must be prefixed `cust/A/` and IAM denies any prefix mismatch.

AC-4: Given the customer dashboard for customer B,
      when the frontend issues an API call,
      then the JWT's `customer_id` claim must match the path's `customerId`; mismatch returns 403.

## Implementation notes

- Tenant ID is in every DDB partition key (see `DESIGN-DATA-001`).
- IAM policies are templated per-customer via CDK (`DESIGN-A2-001`).
- Secrets Manager paths are prefixed by customer ID.
- The dashboard JWT claim is the source of truth in the API gateway authorizer.

## Linked NFRs

- `NFR-SEC-001` VPC isolation.
- `NFR-SEC-002` Least-privilege IAM.
- `NFR-SEC-006` PII redaction in logs.
