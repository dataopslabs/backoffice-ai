---
id: STORY-A3-003
title: FastAPI control-plane scaffold and OpenAPI codegen
type: story
status: approved
owner: founder
depends-on: [STORY-A3-001, ADR-008]
covers-req: [REQ-PRD-016]
bdd: [BDD-A3-003.feature]
tests: [tests/api/test_codegen.py, tests/api/test_auth.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-003 — FastAPI scaffold and codegen

## Story

As the founder, I need the FastAPI control plane scaffold wired to the OpenAPI document, with
JWT auth (Cognito) and a passing health check, so that subsequent stories can add operations.

## Acceptance criteria

AC-1: Given the OpenAPI document at `specs/30-architecture/34-api-contract/openapi.yaml`,
      when I run `make codegen`,
      then Pydantic models are generated to `src/api/_generated/models.py`
      and TypeScript types to `web/lib/_generated/api.d.ts`.

AC-2: Given the FastAPI app,
      when an unauthenticated request hits any non-health endpoint,
      then a 401 with RFC 9457 Problem response is returned.

AC-3: Given a Cognito-issued JWT with `customer:read` scope and `customer_id=A`,
      when the request hits `GET /customers/A/runs`,
      then it is allowed.

AC-4: Given the same token,
      when it hits `GET /customers/B/runs`,
      then a 403 with `cross-tenant-access` Problem type is returned.

AC-5: Given any push to main,
      when `make verify-codegen` runs in CI,
      then a diff in generated code causes the build to fail.

## Technical notes

- `uvicorn` + `mangum` for Lambda compatibility.
- Cognito JWKS cached for 1 hour.
- Tenant-claim enforcement in a custom dependency `require_tenant(customer_id)`.
- Health endpoint `/healthz` returns 200 without auth.

## Definition of done

- [ ] `make codegen` produces clean diff on first run.
- [ ] BDD `BDD-A3-003.feature` covers auth + tenant scope ACs.
- [ ] Codegen-verification CI step active.
