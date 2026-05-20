---
id: STORY-A4-012
title: Customer onboarding wizard
type: story
status: draft
owner: founder
depends-on: [REQ-PRD-015]
covers-req: [REQ-PRD-015]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-012 — Onboarding wizard

## Story

As an operator, I need a guided wizard that captures a new customer's details, provisions all
tenant resources, registers their first policy bundle, and stands up their first workflow,
all in a single session of under 30 minutes.

## Acceptance criteria

AC-1: Given an operator opens `/operator/onboarding/new`,
      when they fill out customer details and submit,
      then a per-tenant CDK custom resource is applied and resources provision.

AC-2: Given provisioning completes,
      when the wizard advances,
      then a policy bundle upload step appears; the operator can attach the customer's bundle.

AC-3: Given the bundle is validated,
      when the workflow YAML is uploaded,
      then it is validated against the customer's plan ceiling and persisted.

AC-4: Given any step fails,
      when the operator retries,
      then progress is preserved; idempotent provisioning skips already-created resources.

## Definition of done

- [ ] Provisioning idempotent.
- [ ] Onboarding actions audit-logged with operator id.
