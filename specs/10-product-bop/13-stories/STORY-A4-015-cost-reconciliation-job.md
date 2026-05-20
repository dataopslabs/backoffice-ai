---
id: STORY-A4-015
title: Weekly cost reconciliation job
type: story
status: draft
owner: founder
depends-on: [STORY-A3-012, NFR-COST-002]
covers-req: [REQ-PRD-002]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-015 — Weekly cost reconciliation

## Story

As the founder, I need a weekly job that compares the synthetic cost meter against AWS Cost &
Usage Reports per customer per component, surfaces variances, and alerts on > 5% drift.

## Acceptance criteria

AC-1: Given the job runs every Monday 09:00 UTC,
      when it completes,
      then a report at `s3://bop-prod-reports/cost-recon/YYYY-MM-DD.json` is produced.

AC-2: Given any per-component variance > 5%,
      when the report is generated,
      then a CloudWatch alarm fires; the operator is paged.

AC-3: Given a pricing.yaml update is needed,
      when the variance investigation completes,
      then the PR updating pricing.yaml references the variance report and the alarm clears.

## Definition of done

- [ ] Job deployed as Lambda + EventBridge schedule.
- [ ] Alarm tested end-to-end with a forced variance.
- [ ] Quarterly internal review of variance trends.
