---
id: REQ-PRD-004
title: Workflow definition versioning and validation
type: req
status: approved
owner: founder
depends-on: [DESIGN-DATA-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-004 — Workflow definition versioning and validation

## Statement

The system **shall** validate every workflow definition against a published JSON Schema before
acceptance, **shall** record every accepted definition as an immutable version, and **shall**
allow at most one active version per (customer, workflow) at any time.

## Rationale

Workflow definitions are configuration-as-code. They are the customer's contract with the agent.
A broken definition silently activated would cause silent automation failure — unacceptable.

## Acceptance criteria

AC-1: Given a workflow YAML that fails schema validation,
      when an operator submits it via API or UI,
      then the upsert returns 400 with a Problem response listing all validation errors;
      no version is created.

AC-2: Given an accepted workflow,
      when the operator submits a modified version,
      then a new version is recorded with an incremented semver and a diff against the prior
      version is computed for audit.

AC-3: Given two versions of the same workflow,
      when the operator activates the new one,
      then in-flight runs continue against the old version (snapshot semantics) and only new
      runs use the new version.

AC-4: Given an active workflow version,
      when an operator initiates rollback,
      then the previous version becomes active; the rollback is logged as an audit event.

## Implementation notes

- JSON Schema lives at `30-architecture/34-api-contract/schemas/workflow.schema.json`.
- Validator: pydantic + `jsonschema` library.
- Each run carries a `workflow_version` field referencing the snapshot it ran against.

## Linked

- `DESIGN-A2-002` Workflow definition format and versioning.
- `STORY-A3-006` Implement workflow validator (MVP).
