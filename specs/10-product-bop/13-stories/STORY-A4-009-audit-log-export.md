---
id: STORY-A4-009
title: Audit log export with SHA-256 manifest
type: story
status: draft
owner: founder
depends-on: [STORY-A3-011]
covers-req: [REQ-PRD-014]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-009 — Audit log export

## Story

As a compliance officer, I need to export the audit log for any date range in a portable format
(jsonl + manifest) suitable for handing to an external auditor.

## Acceptance criteria

AC-1: Given a date range,
      when I request an export,
      then a background job produces a tar.gz containing `audit.jsonl` and a SHA-256 manifest.

AC-2: Given the export completes,
      when I receive the email notification,
      then it contains a time-limited signed URL to download the bundle.

AC-3: Given the open-source `scripts/verify_audit.py`,
      when run against the export,
      then chain integrity verifies without depending on BackOfficePilot.

## Definition of done

- [ ] Export job covered by tests.
- [ ] Manifest format documented in `audit-export-format.md`.
- [ ] Verification script ships under MIT license.
