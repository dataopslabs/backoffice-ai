---
id: STORY-B4-003
title: HIPAA-equivalent control mapping
type: story
status: draft
owner: founder
depends-on: [NFR-COMP-002]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-003 — HIPAA-equivalent controls

## Story

As the founder, I need to document a HIPAA-equivalent control posture for the reconciliation
workflow even though it does not process PHI, so the platform is positioned for future
healthcare workflows (claims reconciliation) without a full re-architecture.

## Acceptance criteria

AC-1: Given the HIPAA Security Rule's administrative, physical, and technical safeguards,
      when mapped to our current controls,
      then each safeguard has at least one implementation reference in the spec set.

AC-2: Given the mapping document,
      when reviewed,
      then gaps are listed with target close dates.

AC-3: Given the gaps,
      when worked through,
      then they reduce by ≥ 50% over 6 months without disrupting the recon workflow.

## Definition of done

- [ ] HIPAA mapping document committed under `compliance/hipaa/`.
- [ ] Gap list reviewed quarterly.
- [ ] BAA template drafted for future healthcare customers.
