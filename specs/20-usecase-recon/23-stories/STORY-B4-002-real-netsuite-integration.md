---
id: STORY-B4-002
title: Real customer NetSuite integration
type: story
status: draft
owner: founder
depends-on: [STORY-B3-002, STORY-A4-001]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-002 — Real NetSuite

## Story

As the founder, I need to move from the seeded sandbox to a real customer's NetSuite production
account for the first paid pilot, including credential exchange, dry-run, and read-only canary.

## Acceptance criteria

AC-1: Given a signed pilot agreement,
      when credentials are exchanged via a documented secure-handoff procedure,
      then they land in Secrets Manager under the customer's prefix; the founder never sees
      them in plaintext.

AC-2: Given an initial dry-run,
      when invoked against the real NetSuite,
      then it runs read-only (no posts) for 1 week and the customer reviews the would-be journal
      entries.

AC-3: Given customer sign-off on the dry-run,
      when posting is enabled,
      then the first 3 days run with explicit human approval per posted JE.

AC-4: Given clean runs for 5 days post-approval,
      when reviewed,
      then unattended autonomy is granted and audit-logged.

## Definition of done

- [ ] Credential handoff procedure documented and exercised.
- [ ] Dry-run gates documented.
- [ ] Customer signs off on each stage of progressive autonomy.
