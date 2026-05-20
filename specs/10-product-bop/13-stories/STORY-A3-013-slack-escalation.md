---
id: STORY-A3-013
title: Slack escalation adapter
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-A2-004, REQ-PRD-008]
covers-req: [REQ-PRD-008]
bdd: [BDD-A3-013.feature]
tests: [tests/adapters/test_slack_escalation.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-013 — Slack escalation adapter

## Story

As the founder, I need a Slack channel adapter that posts escalation messages to the customer's
configured channel with one-click Approve/Reject buttons that round-trip through the
`resolveEscalation` API.

## Acceptance criteria

AC-1: Given an `ExceptionRaised` event,
      when the Slack adapter handles it,
      then a message is posted to the configured channel containing the Reasoner's narrative,
      a link to the run detail, and Approve/Reject buttons.

AC-2: Given a reviewer clicks Approve,
      when the Slack callback hits the API,
      then `resolveEscalation` is invoked with `decision=approve` and the run resumes within
      30 seconds.

AC-3: Given the Slack workspace's token is expired,
      when the adapter attempts to post,
      then a `SlackAuthExpired` event is emitted and the email fallback channel is used.

AC-4: Given the channel was deleted in Slack,
      when the adapter posts,
      then the failure is caught, an alert fires, and the operator can re-configure the channel.

## Technical notes

- Slack SDK (`slack-sdk`) with bot tokens stored per customer in Secrets Manager.
- Interactivity endpoint deployed as a Lambda; signature verified per Slack's request signing.
- Message format uses Block Kit; Approve/Reject buttons carry a signed payload identifying the
  escalation.

## Definition of done

- [ ] Adapter covered by ACs.
- [ ] End-to-end test against a sandbox Slack workspace in CI.
- [ ] Customer onboarding doc explains how to install the BackOfficePilot Slack app.
