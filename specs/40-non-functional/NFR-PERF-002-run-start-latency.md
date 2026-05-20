---
id: NFR-PERF-002
title: Run start latency
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: [REQ-PRD-006, REQ-PRD-007]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-PERF-002 — Run start latency

## Statement

Time from `startRun` API call (or scheduled trigger fire) to first step dispatch **shall not**
exceed 5 seconds at p95.

## Components of the budget

| Component | Budget |
|---|---|
| Run record creation in DDB | 50 ms |
| Workflow + policy bundle load (cached) | 200 ms |
| Opus plan call (with prompt cache hit) | 2.5 s |
| Plan validation + persistence | 200 ms |
| First Executor dispatch | 1.5 s |
| Buffer | 550 ms |

## Verification

- Latency histogram in CloudWatch; alarm on p95 > 5s for 5 consecutive minutes.
- End-to-end synthetic run executes every 15 minutes in UAT; measures full lifecycle including
  start latency.

## Tactics

- Prompt cache warmed via a periodic `warmRun` invocation per customer.
- Nova Act session pool maintains 1–2 warm sessions per active customer.
- Bedrock model is invoked with streaming where applicable to overlap planning with infrastructure.
