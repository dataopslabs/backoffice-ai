---
id: META-003
title: Traceability Matrix (template + generator contract)
type: meta
status: approved
owner: founder
depends-on: [META-001, META-002]
version: 0.1.0
last-updated: 2026-05-19
---

# Traceability Matrix

This file is **machine-maintained** by `scripts/build_traceability.py`. Do not edit the matrix
tables by hand. Edit the source specs and rerun the generator.

The matrix exists to answer four questions on demand:

1. For every requirement (`REQ-*`) and non-functional requirement (`NFR-*`), which stories implement it?
2. For every story, which BDD feature file exercises it?
3. For every feature file, which test functions assert it?
4. For every requirement, is there at least one passing test? (Coverage gaps light up red.)

## Sections

The generator produces five sections, regenerated from frontmatter on every CI run.

### Section A — Requirement → Story map

| Requirement ID | Title | Stories that cover it | Status |
|---|---|---|---|
| _to be populated_ | | | |

A row is `OK` (green) if at least one story has the requirement in `covers-req` **and** that story
has at least one passing test. `MISSING-STORY` if no story claims it. `NO-TEST` if a story exists
but has no `tests:` entry. `FAILING` if tests exist but the latest CI run shows failures.

### Section B — Story → BDD → Test map

| Story ID | Title | BDD file | Test references | Status |
|---|---|---|---|---|
| _to be populated_ | | | | |

A row is `OK` if BDD exists, all scenarios have step definitions, and tests pass.
`NO-BDD` if `bdd` is empty. `NO-TESTS` if `tests` is empty. `RED` if any test fails.

### Section C — NFR → Coverage map

| NFR ID | Category | Title | Stories / designs that address it | Test or evidence |
|---|---|---|---|---|
| _to be populated_ | | | | |

NFRs cover cross-cutting concerns (security, performance, cost, observability, compliance).
Some are addressed by test (e.g. a cost ceiling NFR has a unit test). Others are addressed by
evidence (e.g. a compliance NFR points to an ADR + a runbook section).

### Section D — ADR cross-reference

| ADR ID | Title | Specs that depend on it |
|---|---|---|
| _to be populated_ | | | |

Useful for understanding what breaks if an ADR is reconsidered. If `ADR-007` (pytest-bdd) were
superseded, every story whose tests use pytest-bdd appears in this row.

### Section E — Orphans and warnings

A list of failure conditions, each blocking CI:

- Stories with no covering BDD or test.
- BDD feature files in `/features/` not referenced by any story.
- Test functions in `/tests/` not referenced by any story.
- Requirements with no covering story.
- Specs with `status: in-progress` whose `last-updated` is over 14 days old.
- Frontmatter schema violations.
- Filename ↔ `id` mismatches.

## Generator contract

`scripts/build_traceability.py` MUST:

1. Walk `/specs/**/*.md` and parse YAML frontmatter from each file.
2. Walk `/features/**/*.feature` and parse the `Feature:` line and scenario titles.
3. Walk `/tests/**/*.py`, collect test function names by file.
4. Cross-reference `covers-req`, `bdd`, and `tests` fields against discovered artifacts.
5. Optionally consume the last CI test report (`reports/junit.xml`) for pass/fail status.
6. Replace the five sections above between matching `<!-- BEGIN/END auto -->` markers.
7. Exit non-zero if any orphan or warning is found.

The generator is invoked:

- locally via `make traceability` (alias for `python scripts/build_traceability.py`).
- on every push via `.github/workflows/ci.yml`.
- automatically in pre-commit if the user installs the pre-commit hook (`scripts/install-hooks.sh`).

## Conventions for missing data

While the project is bootstrapping (Session 1–3), most sections will be empty. The generator
should produce a meaningful, non-empty file regardless — a header row with a "no rows yet" note
is acceptable. Empty matrix sections are not CI failures during bootstrap. They become failures
once the first story moves out of `status: draft`.

## Example expected output (illustrative)

```markdown
<!-- BEGIN auto -->
### Section A — Requirement → Story map

| Requirement ID | Title | Stories that cover it | Status |
|---|---|---|---|
| REQ-PRD-001 | Multi-tenancy isolation | STORY-A-007, STORY-A-008 | OK |
| REQ-PRD-002 | Per-agent cost meter | STORY-A-011 | NO-TEST |
| REQ-PRD-003 | Audit log immutability | STORY-A-014 | FAILING |
| REQ-PRD-004 | Workflow YAML schema validation | _none_ | MISSING-STORY |
...
<!-- END auto -->
```

## Why this matters

In a BFSI context, "did we build what the auditor required?" is a question that has to be
answerable in seconds. The traceability matrix is that answer. It also forces hygiene: a story
that claims to cover REQ-X but has no tests will fail CI on the next push, so the gap closes
immediately rather than accumulating.
