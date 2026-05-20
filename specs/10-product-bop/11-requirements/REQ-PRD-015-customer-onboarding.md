---
id: REQ-PRD-015
title: Customer onboarding workflow
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-001, REQ-PRD-013, REQ-PRD-016]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-015 — Customer onboarding

## Statement

The system **shall** support an operator-driven onboarding workflow that provisions a new
customer's tenant resources (DDB partitions, S3 prefixes, KMS aliases, IAM roles, AgentCore
Memory namespace, Cognito user pool app client) and registers the customer's policy bundle
and first workflow, in under 30 minutes of operator wall-clock time.

## Acceptance criteria

AC-1: Given a new customer record,
      when the operator initiates onboarding,
      then a CDK custom resource (or scripted run) provisions all per-tenant resources idempotently.

AC-2: Given onboarding completes,
      when the customer accesses the dashboard for the first time,
      then they can authenticate, view an empty run list, and see prompts to configure their
      first workflow.

AC-3: Given onboarding fails partway,
      when the operator retries,
      then provisioning is idempotent — already-created resources are not duplicated.

AC-4: Given an operator-initiated deprovision (customer churn),
      when invoked with explicit confirmation,
      then runtime resources are torn down; archive S3 bucket and audit records are retained
      per `REQ-PRD-014`.

## Implementation notes

- Onboarding orchestrated by `scripts/onboard_customer.py` against the operator API.
- IAM templates parameterized per customer (`DESIGN-A2-001`).
- Onboarding actions are audit-logged.
