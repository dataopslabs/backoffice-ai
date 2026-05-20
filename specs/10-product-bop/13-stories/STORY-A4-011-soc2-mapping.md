---
id: STORY-A4-011
title: SOC 2 control mapping and evidence pipeline
type: story
status: draft
owner: founder
depends-on: [NFR-COMP-002]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-011 — SOC 2 evidence pipeline

## Story

As the founder, I need an automated evidence pipeline that maps every relevant control to a
queryable artifact (CloudTrail logs, IAM Access Analyzer reports, Config rule status), so that
the SOC 2 Type II observation period is auditor-ready.

## Acceptance criteria

AC-1: Given the control map in `compliance/soc2/controls.yaml`,
      when the evidence collector runs nightly,
      then each control has at least one fresh artifact within the past 24 hours.

AC-2: Given a control without an artifact for > 48 hours,
      when the report runs,
      then it shows a `MISSING` status and pages the operator.

AC-3: Given an auditor request,
      when an operator queries the evidence collector,
      then they receive a time-bounded export of all matching artifacts.

## Definition of done

- [ ] Control map populated for the Type I gap assessment scope.
- [ ] Collector deployed and proven over a 30-day window.
- [ ] Quarterly internal audit dry-run completed.
