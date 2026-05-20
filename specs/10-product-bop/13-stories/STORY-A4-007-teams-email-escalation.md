---
id: STORY-A4-007
title: Teams and email escalation channels
type: story
status: draft
owner: founder
depends-on: [STORY-A3-013]
covers-req: [REQ-PRD-008]
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-007 — Teams + email channels

## Story

As a customer not on Slack, I need Microsoft Teams and email escalation channels as
first-class options with the same Approve/Reject UX.

## Acceptance criteria

AC-1: Given a workflow configured with `channel=teams`,
      when an escalation opens,
      then a Teams card posts to the configured channel with Approve/Reject actions.

AC-2: Given an Approve action,
      when the user clicks it in Teams,
      then `resolveEscalation` is invoked; the run resumes within 30 seconds.

AC-3: Given a workflow configured with `channel=email`,
      when an escalation opens,
      then an email is sent with a signed-link Approve and Reject button.

AC-4: Given a signed-link click,
      when the link is opened,
      then `resolveEscalation` is invoked behind a CSRF-protected confirmation page.

## Definition of done

- [ ] Both channels implemented behind the same `EscalationChannelPort`.
- [ ] Idempotency on the resolve action (double-click safe).
