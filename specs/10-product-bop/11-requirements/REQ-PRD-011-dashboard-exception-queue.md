---
id: REQ-PRD-011
title: Customer dashboard — exception queue
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-008, REQ-PRD-009]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-011 — Dashboard exception queue

## Statement

The dashboard **shall** present a queue of open escalations for the active customer, grouped by
severity, with an inline review surface that allows approving, rejecting, or editing the step
payload without leaving the page.

## Acceptance criteria

AC-1: Given open escalations exist,
      when the customer opens the queue,
      then they see escalations ordered by severity (high > medium > low) and oldest first
      within severity.

AC-2: Given an escalation in the list,
      when the customer expands it,
      then they see the Reasoner's narrative, the relevant step context, attached screenshots,
      and the originating run link.

AC-3: Given the customer selects `Approve`,
      when they submit,
      then the escalation is resolved with `decision=approve` and the run resumes.

AC-4: Given the customer selects `Edit`,
      when they edit the proposed step payload and submit,
      then the run resumes with the edited payload as the step result.

AC-5: Given an SLA-breached escalation,
      when displayed,
      then a red badge highlights it and the breach duration is shown.

## Implementation notes

- Implemented in `web/app/(customer)/escalations/...`.
- Real-time updates via SSE on `/customers/{id}/escalations/stream`.
- Slack/Teams resolution actions also reach this surface via the same `resolveEscalation` API.
