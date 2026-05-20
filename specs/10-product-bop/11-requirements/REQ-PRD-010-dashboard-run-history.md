---
id: REQ-PRD-010
title: Customer dashboard — run history view
type: req
status: approved
owner: founder
depends-on: []
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-010 — Dashboard run history view

## Statement

The dashboard **shall** present a paginated, filterable list of runs for the active customer,
with a detail view that includes the full timeline, step list, cost meter, audit log, and a
link to the immutable run bundle.

## Acceptance criteria

AC-1: Given the customer is logged in,
      when they open the dashboard home,
      then they see a list of the 50 most recent runs across all their workflows.

AC-2: Given the run list,
      when the user applies filters (workflow, status, date range),
      then results update without a full page reload (App Router + SSR + suspense).

AC-3: Given a run detail page,
      when it opens,
      then a step-by-step timeline, the audit events, the cost meter, and a "Download bundle"
      action are visible.

AC-4: Given an in-flight run,
      when the user opens its detail page,
      then live updates stream via SSE; the page shows the active step.

AC-5: Given the user clicks "Download bundle",
      when authorized,
      then they receive a time-limited signed URL to the immutable bundle in S3.

## Implementation notes

- Implemented in `web/app/(customer)/runs/...`.
- Data via `listRuns`, `getRun`, `listSteps`, `getRunAudit`, `getRunCost`, `getRunBundle`
  operations.
- SSE endpoint `/runs/{runId}/stream`.

## Linked

- `DESIGN-A2-007` Dashboard architecture.
