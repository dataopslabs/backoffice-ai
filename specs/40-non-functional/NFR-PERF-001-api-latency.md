---
id: NFR-PERF-001
title: API latency p95 budget
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: [REQ-PRD-010, REQ-PRD-011, REQ-PRD-012]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-PERF-001 — API latency

## Statement

Control-plane API operations **shall** meet the following p95 latency targets, measured at the
client edge (CloudFront / dashboard):

| Operation class | p95 target |
|---|---|
| Read (list/get) | 250 ms |
| Write (POST/PUT) | 500 ms |
| Long-running start (startRun) | 800 ms (returns 202 immediately) |
| Signed URL generation (getRunBundle) | 300 ms |
| Cost rollup (getRunCost) | 500 ms |

## Verification

- CloudWatch RUM (Real User Monitoring) reports p50/p95/p99 per operation.
- Synthetic Canary runs hit each operation every 5 minutes; failures alert.
- CI performance regression test: comparison against the prior week's p95 baseline; >20% drift
  fails the build.

## Tactics

- DDB single-table reads are O(1) for hot paths.
- Cost aggregations are pre-materialized; the API serves from rollups, not raw events.
- API Gateway response caching enabled for stable reads (5 second TTL).
- Lambda cold start mitigated by provisioned concurrency on hot operations.
