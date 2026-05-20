---
id: STORY-A4-008
title: Run history search via OpenSearch
type: story
status: draft
owner: founder
depends-on: [STORY-A3-014, DESIGN-DATA-002]
covers-req: [REQ-PRD-010]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-008 — Run history search

## Story

As a customer, I need full-text search across the last 12 months of runs, including exception
narratives and observation summaries, so that I can answer "find me runs where X happened."

## Acceptance criteria

AC-1: Given an OpenSearch index of runs,
      when I search `vendor:Acme amount:>10000`,
      then matching runs are returned ranked by recency.

AC-2: Given an exception narrative contains "duplicate payment",
      when I search "duplicate payment",
      then the originating run appears in results.

AC-3: Given retention boundary at 12 months,
      when older runs are searched,
      then they are not in the index but a hint links to the immutable bundle in S3.

## Definition of done

- [ ] Index schema documented.
- [ ] Indexer runs as a DDB-stream → OpenSearch subscriber.
- [ ] Search bar wired into the dashboard.
