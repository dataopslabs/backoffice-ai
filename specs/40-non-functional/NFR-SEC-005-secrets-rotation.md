---
id: NFR-SEC-005
title: Customer-system credentials are rotated
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-SEC-005 — Secrets rotation

## Statement

Credentials used by agents to access customer systems (ERPs, banking portals, lending CRMs)
**shall** be stored in AWS Secrets Manager and **shall** be rotated on a schedule (manual or
automatic) appropriate to the system's policy.

## Rotation cadence

| Credential type | Default rotation | Notes |
|---|---|---|
| API tokens | 90 days | Where customer system supports automated rotation |
| OAuth refresh tokens | Automatic per system | Refreshed on use |
| Username + password | 90 days (manual) | We notify the customer when rotation is due |
| TOTP secrets | On enrollment | Replaced when the customer rotates their MFA device |

## Verification

- AWS Config rule `secretsmanager-rotation-enabled-check`.
- Secrets older than their rotation window emit a `SecretRotationOverdue` event.
- Failed rotation attempts page the operator.

## Handling

- The agent receives a fresh secret on every step dispatch; never caches longer than 60 seconds.
- On a 401/403 from a customer system, the orchestrator pauses the run, requests credential
  refresh, and resumes.
