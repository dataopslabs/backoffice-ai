---
id: REQ-B1-005
title: MVP success criteria
type: req
status: approved
owner: founder
depends-on: [REQ-B1-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-B1-005 — MVP success criteria

## Statement

The MVP **shall** be considered successful when the criteria below are met for at least five
consecutive business days in UAT.

## Quantitative targets

| Metric | Target | How measured |
|---|---|---|
| Match accuracy (clean cases) | ≥ 95% correct | Manual review of posted JEs |
| Exception classification accuracy | ≥ 90% correct category | Manual review of escalations |
| Per-run cost | ≤ $1.00 | Cost meter (`CostMeter.total_usd`) |
| Per-run duration | ≤ 8 minutes | Run record |
| Run-start latency (p95) | ≤ 5s | OpenTelemetry trace |
| Unattended successful runs | 5/5 over 5 business days | Run records |
| Audit chain integrity | 100% (no breaks) | `verify_audit.py` per run |

## Qualitative criteria

- A 90-second demo video can be recorded showing one full run end-to-end.
- A CFO of a mid-market bank can be walked through the dashboard and understand the cost
  savings story without further explanation.
- An auditor can be handed the run bundle and confirm chain integrity without our software.

## Anti-criteria (what does NOT count as success)

- A run that succeeds because the test data was overly easy.
- A run where the agent guessed correctly but with low confidence (we want confident or
  escalated, not lucky).
- A pass-rate achieved by silently ignoring the exception cases.
