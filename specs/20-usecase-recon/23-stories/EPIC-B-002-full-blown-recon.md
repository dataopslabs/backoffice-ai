---
id: EPIC-B-002
title: Full-blown — Reconciliation production hardening
type: epic
status: approved
owner: founder
depends-on: [EPIC-B-001, EPIC-A-002]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# EPIC-B-002 — Reconciliation production hardening

## Goal

Take the MVP recon use case from a UAT demo to two paid pilot customers running daily in Prod
with multi-currency support, audit-ready evidence, and performance optimization.

## Stories

- `STORY-B4-001` Multi-currency exception handling (real FX rates).
- `STORY-B4-002` Real NetSuite integration (production sandbox → real customer).
- `STORY-B4-003` HIPAA-equivalent control mapping for the workflow.
- `STORY-B4-004` Multi-region failover for the recon agent.
- `STORY-B4-005` Per-workflow cost guardrails with daily + monthly caps.
- `STORY-B4-006` Customer-portal observability surfaces (per-agent cost view).
- `STORY-B4-007` Reconciliation accuracy SLA monitoring.
- `STORY-B4-008` Vendor-name fuzzy-match tuning + per-customer learning loop.
- `STORY-B4-009` PDF reconciliation evidence pack (regulator-grade).
- `STORY-B4-010` Auto-resume after credential rotation.

## Definition of done

- 2 paying customers running daily.
- Audit pack delivered to one auditor with no follow-ups.
- Multi-currency workflow runs in Prod against a real customer.
- 99.0% workflow success rate over 90 days.
