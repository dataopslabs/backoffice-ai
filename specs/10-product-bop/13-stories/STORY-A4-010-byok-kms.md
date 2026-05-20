---
id: STORY-A4-010
title: BYOK KMS support (enterprise tier)
type: story
status: draft
owner: founder
depends-on: [NFR-SEC-003]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-010 — BYOK KMS

## Story

As an enterprise customer, I need BackOfficePilot to use my own AWS KMS CMK for encryption at
rest, so that I retain key custody and revoke access by disabling the key.

## Acceptance criteria

AC-1: Given a customer provides a CMK ARN with the required key policy,
      when onboarding completes,
      then all per-customer encrypted resources (DDB items, S3 objects, OpenSearch indices)
      use that key.

AC-2: Given the customer disables the CMK,
      when an agent tries to read encrypted data,
      then the read fails cleanly; the dashboard shows a "customer key disabled" banner.

AC-3: Given a CMK rotation occurs on the customer side,
      when subsequent reads happen,
      then they succeed (KMS handles rotation transparently).

AC-4: Given a customer downgrades from enterprise tier,
      when configured,
      then their data is re-encrypted to the default CMK as part of an offboarding workflow.

## Definition of done

- [ ] Adapter layer abstracts CMK selection per customer.
- [ ] Documented key-policy template for customers to apply.
