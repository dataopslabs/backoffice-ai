---
id: EPIC-B-001
title: MVP — Reconciliation use case build
type: epic
status: approved
owner: founder
depends-on: [EPIC-A-001]
covers-req: [REQ-B1-001, REQ-B1-002, REQ-B1-003, REQ-B1-004, REQ-B1-005, REQ-B1-006]
version: 0.1.0
last-updated: 2026-05-19
---

# EPIC-B-001 — MVP recon build

## Goal

Apply EPIC-A-001's platform foundation to the Acme reconciliation use case end-to-end so that a
5-day unattended UAT run can be demonstrated with a 90-second recorded demo.

## Stories

- `STORY-B3-001` Mock Acme bank portal (Vercel-hosted Flask).
- `STORY-B3-002` NetSuite sandbox setup + seed data.
- `STORY-B3-003` Policy bundle (three_way_match.md + journal_posting + tolerances + categories).
- `STORY-B3-004` Reconciliation workflow YAML definition.
- `STORY-B3-005` Mock data generator script with seed.
- `STORY-B3-006` AgentCore Runtime deployment for recon agent.
- `STORY-B3-007` AgentCore Gateway tool configs (NetSuite, bank, Slack).
- `STORY-B3-008` AgentCore Memory namespace + initial state seeds.
- `STORY-B3-009` AgentCore Registry initial version push.
- `STORY-B3-010` Per-workflow cost-meter dashboard + alarms.
- `STORY-B3-011` 5-day unattended UAT run + observation log.
- `STORY-B3-012` 90-second demo video recording.

## Definition of done

- 5 consecutive business-day runs in UAT, fully unattended.
- All success criteria from `REQ-B1-005` met.
- Demo video recorded and reviewed.
- Operator dashboard alarms tuned (no false positives over the run).
