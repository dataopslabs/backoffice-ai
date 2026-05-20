---
id: STORY-A3-006
title: Workflow YAML validator
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-A2-002]
covers-req: [REQ-PRD-004]
bdd: [BDD-A3-006.feature]
tests: [tests/application/test_workflow_validator.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-006 — Workflow validator

## Story

As the founder, I need a validator that accepts a YAML workflow and produces either a fully-
typed `WorkflowDefinition` or a structured list of errors, so that bad definitions never reach
the orchestrator.

## Acceptance criteria

AC-1: Given a valid workflow YAML,
      when validated,
      then a `WorkflowDefinition` is returned and a digest is recorded.

AC-2: Given a YAML missing a required field,
      when validated,
      then a 400 Problem response is returned with one error per missing field.

AC-3: Given a YAML where `step.system` references an undeclared system,
      when validated,
      then a `cross_reference_error` is returned naming both the step and the bad reference.

AC-4: Given a YAML where `budget.max_cost_per_run_usd` exceeds the customer's plan ceiling,
      when validated,
      then validation fails with `plan_ceiling_exceeded`.

AC-5: Given an `upsertWorkflow` call with a definition equivalent to the current version,
      when validated,
      then no new version is created; the existing version is returned.

## Technical notes

- Validator in `src/application/workflow_validator.py`.
- Stage order: YAML parse → JSON Schema validate → Pydantic construct → cross-field check.
- Version bump computed by diffing previous version's serialized form.

## Definition of done

- [ ] All five ACs covered by tests.
- [ ] Validator wired to `upsertWorkflow` operation.
- [ ] BDD scenarios cover the failure paths.
