---
id: STORY-B4-001
title: Multi-currency exception handling with live FX
type: story
status: draft
owner: founder
depends-on: [STORY-B3-004, REQ-B1-003]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B4-001 — Multi-currency exception handling

## Story

As a customer with international vendors, I need cross-currency payments handled deterministically
with an FX-rate lookup and a clear exception narrative when conversion-tolerance breaches.

## Acceptance criteria

AC-1: Given a EUR payment for a USD invoice with EUR/USD FX rate at run-time,
      when reconciliation runs,
      then the payment is converted using the prevailing FX rate (within 1% tolerance) and
      matched if within amount tolerance.

AC-2: Given the FX rate is outside the configured tolerance,
      when the agent runs,
      then a `currency` exception is raised with both amounts, the FX rate used, and the
      tolerance applied.

AC-3: Given the FX rate source is unavailable,
      when a multi-currency case is encountered,
      then the run pauses on a structured `fx_source_down` reason; cost ceiling is paused too.

## Definition of done

- [ ] FX-rate adapter implemented behind a port.
- [ ] At least two FX sources configured (primary + fallback).
- [ ] Test data includes 5 currency-mismatch scenarios.
