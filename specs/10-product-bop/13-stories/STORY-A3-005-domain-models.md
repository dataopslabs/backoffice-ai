---
id: STORY-A3-005
title: Domain models with property-based tests
type: story
status: approved
owner: founder
depends-on: [STORY-A3-001, DESIGN-DATA-001, ADR-002]
covers-req: []
bdd: [BDD-A3-005.feature]
tests: [tests/domain/test_models.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-005 — Domain models

## Story

As the founder, I need pure Python domain models (`Run`, `Plan`, `Step`, `Exception`,
`AuditEvent`, `CostMeter`) implemented as Pydantic v2 models with property-based tests, so
that the orchestrator has well-defined value types before any adapter is written.

## Acceptance criteria

AC-1: Given an attempt to construct any model with missing required fields,
      when the constructor runs,
      then a `ValidationError` is raised with a clear message.

AC-2: Given a `Run` with status `pending`,
      when the orchestrator transitions it to `running`,
      then the transition is permitted; transitioning from `succeeded` back to `running` is
      rejected.

AC-3: Given a `Plan` with N steps,
      when each step's success criteria are inspected,
      then each criterion is a dict with at least one key.

AC-4: Given hypothesis-generated event sequences,
      when applied to a `Run`,
      then the resulting state is reachable by a legal transition sequence.

AC-5: Given a domain model,
      when imported,
      then it has no transitive dependency on `boto3`, `anthropic`, or any HTTP library
      (enforced by `import-linter`).

## Technical notes

- Models live in `src/domain/`.
- State transitions implemented as a Mealy machine; transitions validated in `__post_init__`.
- Tests use hypothesis with custom strategies for each model.
- `import-linter` config in `pyproject.toml`.

## Definition of done

- [ ] All models in `src/domain/`.
- [ ] Hypothesis suite covers state machines.
- [ ] `import-linter` CI check is green.
