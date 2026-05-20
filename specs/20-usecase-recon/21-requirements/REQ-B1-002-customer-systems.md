---
id: REQ-B1-002
title: Customer systems and integration points
type: req
status: approved
owner: founder
depends-on: [REQ-B1-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-B1-002 — Customer systems

## Systems

| System | Type | Auth | Notes |
|---|---|---|---|
| Acme bank treasury portal (mock) | Web | username + password + TOTP | Our Vercel-hosted Flask app for the MVP |
| NetSuite sandbox | Web | OAuth + 2FA | NetSuite-issued sandbox account |
| Slack workspace | Web/API | OAuth bot token | Customer's `#bop-reconciliation` channel |
| Email fallback | SMTP via SES | API key | Used only on SLA breach |

## Capability requirements

The workflow declares `executor_requirements: [web-login, file-download, csv-export,
form-fill, two-factor-auth]`. Any executor selected by the router must advertise all of these.

## Connectivity

- All three customer systems are public-internet web apps. No VPN or private connectivity
  required for the MVP.
- Outbound traffic from agents originates from the AgentCore Runtime's NAT egress in the UAT/
  Prod VPC.
- Customer IP allow-lists must include the relevant NAT IPs (documented per-environment in the
  runbook).

## Credentials

- Bank portal: username + password + TOTP secret in Secrets Manager under
  `cust/acme/bank/credentials`.
- NetSuite: OAuth tokens refreshed automatically; long-lived refresh token in
  `cust/acme/netsuite/oauth`.
- Slack bot token: `cust/acme/slack/bot_token`.

## Acceptance criteria

AC-1: Given the systems above,
      when the agent runs,
      then each authentication step succeeds without manual intervention.

AC-2: Given a credential is rotated,
      when the agent next runs,
      then it reads the fresh value from Secrets Manager (no agent restart).

AC-3: Given any one system is unreachable,
      when the agent attempts to use it,
      then the orchestrator's circuit breaker opens for that system and the run pauses (no retry
      storm).
