---
id: EPIC-A-001
title: MVP — Platform foundation
type: epic
status: approved
owner: founder
depends-on: []
covers-req: [REQ-PRD-001, REQ-PRD-002, REQ-PRD-003, REQ-PRD-004, REQ-PRD-006, REQ-PRD-007, REQ-PRD-008, REQ-PRD-010, REQ-PRD-012, REQ-PRD-016]
version: 0.1.0
last-updated: 2026-05-19
---

# EPIC-A-001 — MVP Platform Foundation

## Goal

Stand up enough of BackOfficePilot in 3 weeks (solo founder) to demonstrate the multi-model
orchestration on an end-to-end BFSI invoice reconciliation workflow, with cost telemetry,
audit log, and a working customer dashboard, deployed to UAT only.

## Stories (MVP)

- `STORY-A3-001` Bootstrap repo, dev environment, CI skeleton.
- `STORY-A3-002` CDK network + data + AgentCore stacks for UAT.
- `STORY-A3-003` FastAPI control-plane scaffold + OpenAPI codegen.
- `STORY-A3-004` Next.js dashboard scaffold + Cognito auth.
- `STORY-A3-005` Domain models (Run, Plan, Step, Exception) with property-based tests.
- `STORY-A3-006` Workflow YAML validator (REQ-PRD-004).
- `STORY-A3-007` BedrockOpusReasoner adapter (plan + reconcile).
- `STORY-A3-008` NovaActExecutor adapter (primary).
- `STORY-A3-009` AgentCoreBrowserExecutor adapter (fallback).
- `STORY-A3-010` PERC loop orchestrator + cost ceiling enforcer.
- `STORY-A3-011` S3 hash-chain audit writer (DESIGN-A2-005).
- `STORY-A3-012` Cost meter subscriber + pricing.yaml (DESIGN-A2-006).
- `STORY-A3-013` Slack escalation adapter (DESIGN-A2-004).
- `STORY-A3-014` Dashboard — run list + run detail pages.
- `STORY-A3-015` Dashboard — exception queue + cost view.
- `STORY-A3-016` Traceability matrix generator script.

## Definition of done (epic-level)

- All 16 stories `status: done`.
- Reconciliation workflow runs unattended in UAT for 5 consecutive business days.
- p95 run-start latency under 5 seconds (NFR-PERF-002).
- Per-run cost under $1.00 (NFR-COST-001).
- Dashboard renders run list, run detail, exception queue, cost view.
- CI green; traceability matrix shows ≥ 90% REQ coverage.
- Demo video recorded (90 seconds, narrating one full run).
