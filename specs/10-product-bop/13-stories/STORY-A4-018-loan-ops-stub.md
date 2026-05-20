---
id: STORY-A4-018
title: Loan-ops workflow stub (library expansion)
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

# STORY-A4-018 — Loan-ops workflow stub

## Story

As the founder, I need a loan application processing workflow stub — extract data from a PDF
application, validate against underwriting policy, populate the lending CRM — to support the
third-pilot customer.

## Acceptance criteria

AC-1: Given a PDF loan application,
      when the workflow runs,
      then key fields are extracted; validation against the policy bundle is logged.

AC-2: Given validation passes,
      when the workflow continues,
      then ~120 fields are populated in the lending CRM and the application is routed for
      approval.

AC-3: Given an underwriting exception,
      when raised,
      then it escalates with the relevant policy citation in the narrative.

## Definition of done

- [ ] Workflow YAML committed.
- [ ] Policy bundle for underwriting drafted.
- [ ] Synthetic-data demo run completes end-to-end.
