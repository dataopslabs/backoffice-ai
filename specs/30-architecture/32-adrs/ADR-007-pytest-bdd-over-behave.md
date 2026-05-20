---
id: ADR-007
title: Use pytest-bdd over behave for BDD execution
type: adr
status: approved
owner: founder
depends-on: [ADR-003]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-007: Use pytest-bdd over behave for BDD execution

## Context

Two mature Python options exist for executing Gherkin feature files: **behave** and
**pytest-bdd**. They differ in execution model, fixture sharing, and ecosystem integration.

We will write a large pytest test suite for unit/integration concerns regardless. BDD must
coexist with it cleanly.

## Decision

Use **pytest-bdd**. All BDD scenarios are pytest tests that happen to be driven by Gherkin.

## Alternatives considered

- **behave**: a more "pure" BDD framework with its own runner. Forces a separate runner and
  duplicate fixtures. Tooling integration (coverage, mypy, parallelism) is weaker.
- **Lettuce**: unmaintained.
- **No BDD, plain pytest**: loses the executable behavior contract that audit relies on.

## Consequences

**Positive**

- Single runner (`pytest`). One CI step. One coverage report.
- Pytest fixtures reused by both BDD and unit tests.
- Plays well with pytest-xdist for parallelism and pytest-asyncio for async step definitions.
- IDE integration is universally good.

**Negative**

- The Gherkin parser is slightly less strict than behave's. We lint with `gherkin-lint`
  separately to compensate.
- Some online behave examples don't translate one-to-one.

## Conventions

- One feature file per story, named `BDD-<DOMAIN>-<NUMBER>.feature`.
- Step definitions live next to the test module, not in a global `steps/` folder. This keeps
  related code close.
- `pytest_bdd_apply_tag` is configured so scenario tags (`@slow`, `@integration`, `@network`)
  filter into the standard pytest tag system.
