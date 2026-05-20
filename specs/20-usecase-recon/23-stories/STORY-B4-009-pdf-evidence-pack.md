---
id: STORY-B4-009
title: PDF reconciliation evidence pack (regulator-grade)
type: story
status: draft
owner: founder
depends-on: [STORY-A3-011, REQ-PRD-014]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-009 — PDF evidence pack

## Story

As a compliance officer, I need a one-PDF-per-month evidence pack summarizing reconciliation
activity (runs, exceptions, decisions, costs, policy version) that I can hand to my regulator
or auditor without further preparation.

## Acceptance criteria

AC-1: Given a calendar month closes,
      when the evidence pack is generated,
      then it summarizes: run counts, success rate, exception counts by category, agent cost
      total, policy versions in effect, and a chain-integrity certificate.

AC-2: Given the PDF,
      when inspected,
      then it includes a SHA-256 manifest covering every linked run bundle.

AC-3: Given the PDF is generated,
      when the customer compliance officer opens it,
      then the format matches a pre-approved template under `compliance/templates/`.

## Definition of done

- [ ] Generator deployed on monthly schedule.
- [ ] Template reviewed by an external advisor with auditor experience.
- [ ] Customer can request ad-hoc pack via dashboard button.
