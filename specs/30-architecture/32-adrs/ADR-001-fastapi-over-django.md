---
id: ADR-001
title: Use FastAPI for the Python backend
type: adr
status: approved
owner: founder
depends-on: []
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-001: Use FastAPI for the Python backend

## Context

BackOfficePilot's backend is a coordinator that:

- Issues many concurrent calls to Bedrock (Claude Opus) and AgentCore (Runtime, Gateway, Memory).
- Coordinates Nova Act sessions whose duration is highly variable (seconds to minutes).
- Exposes a REST API to the Next.js dashboard.
- Generates OpenAPI 3.1 contracts that the frontend consumes via codegen.
- Performs schema validation on every Reasoner/Executor handoff using Pydantic-friendly models.

The two realistic Python options are FastAPI and Django (with Django REST Framework).

## Decision

Use **FastAPI** + **Pydantic v2** + **uvicorn** as the HTTP layer.

## Alternatives considered

- **Django + DRF**: more "batteries included" (admin, ORM, auth scaffolding), better suited to
  CRUD-heavy products. Async support is bolted-on and not first-class.
- **Starlette directly**: lower level than FastAPI; lose auto OpenAPI generation and dependency
  injection.
- **Litestar / Sanic**: viable but smaller ecosystems and fewer enterprise reference deployments.

## Consequences

**Positive**

- Native async/await throughout, which matches the Bedrock+AgentCore+Nova Act calling pattern.
- Pydantic v2 models double as request/response schemas and as the handoff JSON Schema source.
- Auto OpenAPI generation reduces drift between contract and code (though we are choosing
  spec-first regardless, see `ADR-008`).
- Excellent typing story; mypy clean.

**Negative**

- No built-in admin UI. We build the admin surface in Next.js instead.
- ORM is BYO. We will use SQLAlchemy 2.0 async where a relational store is needed, but most state
  lives in DynamoDB or S3.
- The community is smaller than Django's; expect to handle some operational concerns ourselves.

**Operational**

- Deployment targets uvicorn-on-Lambda (for low-volume paths) and uvicorn-on-Fargate (for
  long-running orchestrators) until AgentCore Runtime hosts the application directly.
