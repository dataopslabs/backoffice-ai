---
id: REQ-PRD-009
title: Escalation SLA tracking
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-008]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-009 — Escalation SLA tracking

## Statement

The system **shall** track an SLA timer on every open escalation, **shall** emit a breach event
when the SLA expires, and **shall** record the resolution actor and decision in the audit log.

## Acceptance criteria

AC-1: Given a workflow with `escalation.reviewer_sla_minutes = 60`,
      when an escalation is opened,
      then a timer is started; SLA breached at minute 60.

AC-2: Given an SLA breach occurs,
      when the timer expires without resolution,
      then an `SlaBreached` event is emitted and a follow-up notification is sent to the
      configured fallback channel (email by default).

AC-3: Given an escalation is resolved,
      when the reviewer's decision is recorded,
      then the resolution actor (email or user ID), decision (`approve`/`reject`/`edit`), and
      optional note are captured in an `EscalationResolved` event and the corresponding
      `AuditEvent`.

AC-4: Given an escalation is resolved with `decision=edit`,
      when the edited payload is provided,
      then the orchestrator resumes the run with the edited payload as the step result.

## Implementation notes

- SLA tracking lives in a separate Lambda subscriber to `EscalationOpened` and
  `EscalationResolved` events.
- Timer is a DDB TTL attribute backed by a Step Functions wait state for fallback notification.
