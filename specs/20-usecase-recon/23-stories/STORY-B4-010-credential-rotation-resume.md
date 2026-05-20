---
id: STORY-B4-010
title: Auto-resume after credential rotation
type: story
status: draft
owner: founder
depends-on: [NFR-SEC-005]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-010 — Auto-resume after credential rotation

## Story

As a customer, I need the agent to detect credential rotation (401/403 from a customer system),
pull the fresh secret, and resume without operator intervention.

## Acceptance criteria

AC-1: Given a NetSuite OAuth token expires mid-run,
      when the next API call receives 401,
      then the agent triggers credential refresh via Secrets Manager and retries within 30
      seconds.

AC-2: Given the rotated credential is invalid (typo, format error),
      when refresh fails,
      then the run pauses on `reason=credential_refresh_failed`; the operator is alerted.

AC-3: Given a successful rotation,
      when subsequent steps proceed,
      then no manual intervention is needed; the audit log captures the rotation event.

## Definition of done

- [ ] Auto-resume tested against simulated 401s.
- [ ] Alert path tested.
- [ ] Customer-onboarding documentation describes rotation expectations.
