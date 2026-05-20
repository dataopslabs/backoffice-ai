---
id: STORY-A4-005
title: Run replay (timeline scrubber)
type: story
status: draft
owner: founder
depends-on: [STORY-A3-014]
covers-req: [REQ-PRD-010]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-005 — Run replay

## Story

As a compliance officer, I need a timeline scrubber on the run detail page that lets me step
forward and backward through every action with the agent's reasoning shown at each point.

## Acceptance criteria

AC-1: Given a completed run,
      when I open replay,
      then a horizontal timeline shows every domain event in temporal order.

AC-2: Given I drag the scrubber to time T,
      when it lands,
      then the page shows the agent state, screenshot, and reasoning trace as of T.

AC-3: Given a step had screenshots attached,
      when I scrub through it,
      then the screenshots appear in sequence.

AC-4: Given I click "Export PDF" on a replay state,
      when invoked,
      then a single-page PDF summarizing that moment (state, reasoning, screenshot) downloads.

## Definition of done

- [ ] Smooth scrubbing performance under runs with 50+ events.
- [ ] PDF export works in Chrome and Safari.
