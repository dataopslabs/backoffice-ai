---
id: STORY-A4-013
title: CFO savings dashboard
type: story
status: draft
owner: founder
depends-on: [STORY-A3-015]
covers-req: [REQ-PRD-012]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-013 — CFO savings dashboard

## Story

As a CFO at the customer, I need a board-ready savings dashboard showing labor cost avoided,
per-workflow ROI, and a 12-month projection so I can defend the BackOfficePilot spend.

## Acceptance criteria

AC-1: Given a labor-cost baseline configured per workflow,
      when the dashboard renders,
      then aggregated $ saved over a configurable period is shown with a moving-average trend.

AC-2: Given multiple workflows,
      when the CFO views the page,
      then per-workflow ROI (savings / BackOfficePilot cost) is shown as a sortable table.

AC-3: Given the user clicks "Board PDF",
      when invoked,
      then a one-page PDF summarizing savings, ROI, and trends downloads, brand-styled.

## Definition of done

- [ ] Charts use the brand palette.
- [ ] PDF export renders identically in Chrome and Safari.
- [ ] At least one customer's labor baseline is configured.
