---
id: STORY-A3-014
title: Dashboard — run list and run detail pages
type: story
status: approved
owner: founder
depends-on: [STORY-A3-004, DESIGN-A2-007, REQ-PRD-010]
covers-req: [REQ-PRD-010]
bdd: [BDD-A3-014.feature]
tests: [web/__tests__/runs.spec.ts]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-014 — Dashboard runs

## Story

As a customer, I need a run list and a run detail page so I can see what the agent did, what
went wrong, and download the immutable bundle.

## Acceptance criteria

AC-1: Given runs exist for my customer,
      when I open the dashboard home,
      then I see the 50 most recent runs with status, workflow, started_at, duration, cost.

AC-2: Given I apply filters (workflow=reconciliation, status=succeeded, last 7 days),
      when I submit,
      then results update without a full reload.

AC-3: Given I open a run,
      when it loads,
      then I see: status timeline, step list, cost breakdown, audit events, "Download bundle"
      button.

AC-4: Given the run is in-flight,
      when I open it,
      then live updates stream via SSE; the active step is highlighted; cost meter updates in
      real time.

AC-5: Given I click "Download bundle",
      when authorized,
      then I get a signed URL to a tar.gz that includes `audit.jsonl`, `plan.json`, every step's
      observations, and screenshots.

## Technical notes

- Server component fetches initial list; client component handles filter state.
- SSE handler in `web/lib/sse.ts` reconnects on disconnect.
- Bundle download triggers `getRunBundle` then redirects to the signed URL.

## Definition of done

- [ ] All ACs covered by Playwright tests.
- [ ] Loading skeletons; no spinner-only states.
- [ ] Brand palette per `DESIGN-A2-007`.
