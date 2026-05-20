---
id: STORY-A4-004
title: Policy bundle versioning UI
type: story
status: draft
owner: founder
depends-on: [STORY-A3-015, DESIGN-A2-008]
covers-req: [REQ-PRD-013]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-004 — Policy bundle UI

## Story

As a customer compliance officer, I need a UI to upload, validate, diff, activate, and roll
back policy bundle versions, so I can manage policy without operator help.

## Acceptance criteria

AC-1: Given a compliance officer opens `/policy-bundles`,
      when the page loads,
      then they see a list of versions with timestamps, approver, and active status.

AC-2: Given two versions,
      when the officer clicks "Compare",
      then a markdown diff renders, with semantic highlighting on tolerance changes.

AC-3: Given an upload,
      when the user drops a tar.gz,
      then the validator runs in real time; errors appear next to the offending file.

AC-4: Given the user clicks "Activate" on a non-active version,
      when they confirm in a modal showing the diff,
      then the version becomes active; the change is audit-logged.

## Definition of done

- [ ] All ACs covered.
- [ ] Markdown diff library chosen and documented.
- [ ] Activation requires `customer:write` scope.
