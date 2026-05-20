---
id: STORY-A4-017
title: KYC workflow stub (library expansion)
type: story
status: draft
owner: founder
depends-on: [STORY-A3-006]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-017 — KYC workflow stub

## Story

As the founder, I need a working KYC workflow stub — sanctions screening + adverse-media check
+ review memo — sufficient for a second-customer pilot, demonstrating that the platform supports
workflows beyond reconciliation without code changes to the core.

## Acceptance criteria

AC-1: Given a `kyc_review.yaml` workflow definition,
      when the platform validates it,
      then it passes with no schema changes to the core.

AC-2: Given a KYC pilot run,
      when it executes,
      then it produces a review memo, an OFAC/PEP/adverse-media check log, and a structured
      risk score.

AC-3: Given an ambiguous result,
      when the Reasoner reports low confidence,
      then it escalates per the same exception pipeline as reconciliation.

## Definition of done

- [ ] Workflow YAML committed.
- [ ] Policy bundle for KYC drafted.
- [ ] End-to-end pilot run completes against synthetic customer data.
