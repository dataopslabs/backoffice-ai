---
id: REQ-PRD-012
title: Customer dashboard — cost view
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-002]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-012 — Dashboard cost view

## Statement

The dashboard **shall** present per-agent and per-workflow cost views with daily, weekly, and
monthly aggregations, comparisons against labor-cost baselines provided by the customer, and
a forecast for the next billing period.

## Acceptance criteria

AC-1: Given the customer opens the cost page,
      when it loads,
      then it shows a stacked bar chart of cost by workflow over the last 30 days.

AC-2: Given a specific agent,
      when the user drills in,
      then per-run cost is plotted with annotations on outliers (>2σ runs).

AC-3: Given the customer has configured a labor-cost baseline,
      when the cost view renders,
      then a "savings vs labor" panel shows aggregated $ saved and a moving NPV.

AC-4: Given live runs are in progress,
      when the dashboard is open,
      then current-day spend updates within 60 seconds via SSE.

AC-5: Given an export request,
      when the user clicks "Export CSV",
      then a CSV of run-level cost data for the selected period is downloaded.

## Implementation notes

- Charts rendered with Recharts (component matches the proposal-deck color palette).
- Cost decomposition matches the categories in `CostMeter` schema.
- Labor-cost baseline stored in customer settings; default $35/hour fully-loaded.
