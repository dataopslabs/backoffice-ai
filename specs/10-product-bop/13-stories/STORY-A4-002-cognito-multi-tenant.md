---
id: STORY-A4-002
title: Cognito multi-tenant auth with scopes
type: story
status: draft
owner: founder
depends-on: [STORY-A3-003, STORY-A3-004]
covers-req: [REQ-PRD-016, REQ-PRD-001]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-002 — Cognito multi-tenant

## Story

As the founder, I need Cognito hardened for multi-tenancy: per-customer app clients, custom
attributes for tenant claim, scopes for `customer:read`/`customer:write`/`operator`, and
self-serve invite flow for the customer admin.

## Acceptance criteria

AC-1: Given a new customer,
      when onboarding completes,
      then a Cognito app client exists scoped to their tenant id.

AC-2: Given a customer admin invites a user,
      when the invitee accepts,
      then they receive a token with the correct `customer_id` claim and `customer:read` scope
      by default.

AC-3: Given a customer admin promotes a user to `customer:write`,
      when the change is applied,
      then subsequent tokens carry the new scope.

AC-4: Given an operator,
      when they impersonate (read-only) a customer's view,
      then both the operator and the impersonated tenant id appear in audit logs.

## Definition of done

- [ ] Invite + promote flows complete.
- [ ] Audit log captures every role change.
- [ ] Multi-factor auth required for `operator` scope holders.
