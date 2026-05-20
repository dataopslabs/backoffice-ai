---
id: REQ-PRD-016
title: API authentication and authorization
type: req
status: approved
owner: founder
depends-on: [REQ-PRD-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-016 — API authentication and authorization

## Statement

The system **shall** authenticate every API request via JWT bearer tokens issued by Amazon
Cognito, **shall** authorize each request against a scope-and-tenant policy, and **shall**
deny any request whose token does not present matching customer scope and tenant claim.

## Acceptance criteria

AC-1: Given a request without a bearer token,
      when it reaches the API,
      then it is rejected with 401 and a Problem response of type `unauthenticated`.

AC-2: Given a token with `customer:read` scope and `customer_id=A`,
      when it requests `GET /customers/B/runs`,
      then it is denied with 403 and Problem type `cross-tenant-access`.

AC-3: Given a token with `operator` scope,
      when it requests any customer endpoint,
      then it is allowed; the operator role is logged on every action it takes.

AC-4: Given a token has expired,
      when used,
      then 401 is returned; the client refreshes via Cognito and retries.

AC-5: Given a write operation,
      when the request lacks an `Idempotency-Key` header,
      then the API still accepts it but logs a warning; presence is required only for retried
      writes per RFC 9457 best practices.

## Implementation notes

- Cognito user pool per environment; app client per customer.
- API Gateway JWT authorizer validates issuer, audience, signature, expiry.
- Custom Lambda authorizer enforces tenant claim match against URL path.
- Scopes: `customer:read`, `customer:write`, `operator`.
