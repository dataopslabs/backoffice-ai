---
id: STORY-B4-004
title: Multi-region failover for the recon agent
type: story
status: draft
owner: founder
depends-on: [STORY-A4-020]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-004 — Multi-region failover

## Story

As an enterprise customer, I need confidence that BackOfficePilot can continue running in a
secondary region if the primary fails, with RTO under 4 hours and RPO under 30 minutes.

## Acceptance criteria

AC-1: Given a primary-region outage simulation,
      when the operator initiates failover,
      then runs resume in `us-west-2` within 4 hours.

AC-2: Given the run bundle in S3,
      when cross-region replication is enabled,
      then audit data is recoverable in the secondary region with ≤ 30 minutes of lag.

AC-3: Given normal operation,
      when the customer reads run history,
      then both regions' contributions are visible without surfacing the failover boundary.

## Definition of done

- [ ] Failover runbook documented.
- [ ] Cross-region replication enabled on critical buckets.
- [ ] Annual DR drill scheduled.
