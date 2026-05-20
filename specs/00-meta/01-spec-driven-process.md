---
id: META-001
title: Spec-Driven Development Process
type: spec
status: approved
owner: founder
depends-on: []
version: 0.1.0
last-updated: 2026-05-19
---

# Spec-Driven Development Process

## Why spec-driven

In a BFSI product, the spec is also the audit artifact. Every implemented behavior must be
traceable back to a written requirement, that requirement must be testable, and the test must
prove the behavior. Spec-driven development with a `Spec → BDD → TDD → Code` chain enforces this
without ceremony, because each step is a small, mechanical translation of the step before it.

It also makes the project AI-buildable. A well-formed spec contains everything Claude Code (or any
engineer) needs to write the BDD; a well-formed BDD contains everything needed to write the test;
a well-formed test contains everything needed to write the code. No step requires creative leaps.

## The chain

```
┌─────────┐    ┌─────────┐    ┌─────────┐    ┌─────────┐
│  SPEC   │ →  │   BDD   │ →  │   TDD   │ →  │  CODE   │
└─────────┘    └─────────┘    └─────────┘    └─────────┘
   .md            .feature       .py            .py / .ts
   /specs         /features      /tests         /src, /web
```

Each artifact is in its own folder, named by the same stable ID, and linked by frontmatter.

## Lifecycle of a feature

### 1. Spec authoring

A new feature begins with one or more spec files. Common starting points:

- A new requirement → `REQ-PRD-NNN-<slug>.md` in `10-product-bop/11-requirements/`.
- A new non-functional concern → `NFR-<CATEGORY>-NNN-<slug>.md` in `40-non-functional/`.
- A new design decision needed → `DESIGN-<DOMAIN>-NNN-<slug>.md` in `12-design/` or `22-design/`.
- A new story to implement → `STORY-<DOMAIN>-NNN-<slug>.md` in `13-stories/` or `23-stories/`.

The author writes the spec in markdown with the standard frontmatter. Acceptance criteria are
written in **Given/When/Then** form. The spec starts at `status: draft`.

### 2. Spec review and approval

The owner (or a peer) reviews the spec. When approved, `status` becomes `approved` and the spec
is considered the contract. Subsequent changes are version-bumped and noted in the file's
revision history.

### 3. BDD translation

For every approved story, a corresponding `.feature` file is created at
`/features/<DOMAIN>/BDD-<DOMAIN>-NNN.feature`.

Because spec ACs are already in Given/When/Then form, this is mechanical:

```markdown
<!-- in STORY-B3-027-wire-agentcore-gateway-tool.md -->

## Acceptance criteria

AC-1: Given a workflow definition with a `netsuite_export` tool reference,
      when the orchestrator dispatches step `s2`,
      then the AgentCore Gateway proxies the call to the configured Nova Act browser session
      and returns the exported CSV path within 30 seconds.
```

Translates to:

```gherkin
# features/recon/BDD-B3-027.feature
Feature: AgentCore Gateway proxies NetSuite export tool calls

  Scenario: Successful export
    Given a workflow definition with a "netsuite_export" tool reference
    When the orchestrator dispatches step "s2"
    Then the AgentCore Gateway proxies the call to the configured Nova Act browser session
    And returns the exported CSV path within 30 seconds
```

The story's frontmatter is updated to reference the feature file: `bdd: [BDD-B3-027.feature]`.

### 4. TDD scaffolding (red)

For every feature file, pytest-bdd step definitions and test fixtures are created at
`/tests/<domain>/test_<slug>.py`.

```python
# tests/recon/test_gateway_netsuite.py
from pytest_bdd import scenarios, given, when, then

scenarios("../../features/recon/BDD-B3-027.feature")

@given('a workflow definition with a "netsuite_export" tool reference')
def workflow_with_netsuite_tool(workflow_registry): ...

@when('the orchestrator dispatches step "s2"')
def dispatch_step(orchestrator, workflow): ...

@then('the AgentCore Gateway proxies the call to the configured Nova Act browser session')
def verify_gateway_proxied(gateway_spy): ...

@then("returns the exported CSV path within 30 seconds")
def verify_csv_path_returned(result): ...
```

Tests are run and must fail (red). Failure must be **for the right reason** — i.e. the assertion
fails, not a NameError. If the test passes on first run, the test is wrong, not the code.

### 5. Implementation (green)

The minimum code to make the failing tests pass is written. Code lives in `/src` (Python) or
`/web` (Next.js). New modules respect the hexagonal layout:

- Pure domain logic → `src/domain/<context>/`
- Use cases / orchestration → `src/application/<context>/`
- AWS/AgentCore/Nova Act integration → `src/adapters/<vendor>/`
- HTTP/FastAPI surface → `src/api/`

Tests pass. CI is green. Pull request opened.

### 6. Refactor

With tests green, refactor for clarity. Tests stay green throughout. No new behavior introduced.

### 7. Spec closure

The story's frontmatter is updated:

```yaml
status: done
version: 1.0.0
last-updated: 2026-06-12
```

The traceability matrix (`00-meta/03-traceability-matrix.md`) is regenerated by
`scripts/build_traceability.py`. The matrix shows, for every REQ and NFR, which stories cover it,
which feature files exercise it, and which test functions assert it. Any REQ with zero test
coverage shows up as a red row.

## When to write an ADR

Open `30-architecture/32-adrs/ADR-NNN-<slug>.md` whenever:

- A choice is non-obvious or has long-term consequences.
- A choice closes off other reasonable options.
- A future engineer will likely ask "why did we do it this way?"

ADRs are short: context, decision, consequences. They are immutable once accepted; a later ADR
supersedes an earlier one.

Examples of when to write one:
- Choosing FastAPI over Django (already done: `ADR-001`).
- Choosing pytest-bdd over behave (planned: `ADR-007`).
- Choosing to run a separate VPC for Prod vs UAT instead of subnet isolation.
- Choosing to route most workflows through Nova Act and reserve AgentCore Browser for fallback.

## When to update the traceability matrix

Whenever any of these change:

- A new REQ, NFR, story, or feature file is added.
- A story's frontmatter changes status, bdd ref, or test ref.
- A spec is superseded.

A small script under `scripts/build_traceability.py` reads frontmatter from every `.md` file in
`/specs` and every `.feature` file in `/features` and regenerates the matrix. CI runs this and
fails the build if the committed matrix is out of date.

## Definitions of done

| Stage | Definition of done |
|---|---|
| Spec | Approved, all ACs in Given/When/Then, dependencies declared, owner assigned. |
| BDD | Feature file exists, all scenarios cover all ACs, lints clean (`gherkin-lint`). |
| TDD | Tests fail in red for the right reason, fixtures and step definitions exist, no skipped scenarios. |
| Code | All tests green, CI green, no untested branches in coverage report, ADRs filed for non-obvious choices, spec frontmatter updated. |

## Versioning specs

Specs follow semver-lite:

- **Patch** (0.0.x): typo, clarification, formatting.
- **Minor** (0.x.0): new ACs added, scenarios broadened, no behavior contradicted.
- **Major** (x.0.0): an existing AC changes or is removed. **Major changes require superseding**:
  copy the spec to a new ID, mark the old one `status: superseded`, link from old to new.

Implementations track spec versions in commit messages so you can reconstruct which spec version
was being built against.

## Anti-patterns to avoid

- **Writing code that "looks reasonable" without a spec.** If there's no spec, stop and write one.
- **Treating the BDD as documentation.** It is executable behavior. If it's wrong, it fails CI.
- **Skipping the red phase of TDD.** If you can't see the test fail, you don't know it works.
- **Refactoring the spec to match the code.** The spec leads; the code follows. If a spec is wrong,
  it gets superseded with a new version, not silently rewritten.
- **Leaving `status: draft` after merge.** Status must reflect reality.
