---
id: STORY-B3-003
title: Policy bundle (three_way_match + journal_posting + tolerances + categories)
type: story
status: approved
owner: founder
depends-on: [REQ-B1-001, REQ-B1-003, DESIGN-A2-008]
covers-req: [REQ-B1-001]
bdd: []
tests: [tests/policy/test_bundle_loads.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-003 — Policy bundle

## Story

As the founder, I need a complete policy bundle for the Acme reconciliation workflow so that
the Reasoner has stable guidance for matching, tolerances, journal posting, and exception
categorization.

## Acceptance criteria

AC-1: Given the bundle at `policy/acme/2026-05/`,
      when validated by the platform validator,
      then it accepts the bundle and creates version `1.0.0`.

AC-2: Given the bundle,
      when loaded into the Reasoner prompt prefix,
      then Bedrock prompt-cache reports ≥ 80% cached fraction on the second and subsequent
      calls.

AC-3: Given an edge case (e.g. $0.04 mismatch vs $0.05 tolerance),
      when applied,
      then `three_way_match.md` clearly determines whether to match or escalate, with no
      ambiguity.

## Bundle contents

- `index.yaml` (manifest).
- `three_way_match.md` (matching rules + tolerance).
- `journal_posting_rules.md` (account mapping, memo format).
- `tolerance_definitions.md` (per-vendor overrides if any).
- `escalation_categories.md` (four categories + examples).

## Definition of done

- [ ] Bundle committed.
- [ ] Reasoner integration test passes against bundle.
- [ ] Bundle approval: signed by founder (placeholder for actual customer BSA officer).
