---
id: STORY-A4-020
title: Data residency selector (region pinning)
type: story
status: draft
owner: founder
depends-on: [STORY-A4-001]
covers-req: [REQ-PRD-001]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-020 — Data residency selector

## Story

As an EU customer, I need to pin all data storage (DDB, S3, OpenSearch, AgentCore Memory) and
all Bedrock invocations to a specific AWS region, so that data does not cross sovereign
boundaries.

## Acceptance criteria

AC-1: Given a customer record with `region=eu-central-1`,
      when onboarding provisions resources,
      then every persisted store lives in `eu-central-1`; Bedrock calls are region-pinned.

AC-2: Given an audit query,
      when an operator asks "where does customer X's data live,"
      then the answer is deterministic and shown in the operator dashboard.

AC-3: Given a customer's region preference changes,
      when the change is requested,
      then it is treated as a migration (multi-week project), not a config flip; documented in
      the operations runbook.

## Definition of done

- [ ] Per-customer region attribute respected at every adapter.
- [ ] Region pinning verified by synth-test against a customer in eu-central-1.
- [ ] Operations runbook documents the data-residency migration flow.
