---
id: ADR-003
title: Spec-driven development with Spec → BDD → TDD → Code chain
type: adr
status: approved
owner: founder
depends-on: []
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-003: Spec-driven development with Spec → BDD → TDD → Code chain

## Context

BackOfficePilot serves regulated BFSI customers whose auditors require traceability from
behavior to requirement. Engineering organizations that meet that bar usually do so via heavy
process — Jira hierarchies, document review gates, RTM spreadsheets — which is overhead the
solo founder cannot carry.

There is a lighter-weight approach: treat the specification documents as the source of truth,
write BDD feature files mechanically from spec acceptance criteria, write tests from BDD, and
write code last. Tools and AI assistants can perform each translation step with the previous
artifact as input.

## Decision

Adopt the four-stage chain `SPEC → BDD → TDD → CODE` as the engineering process. Concretely:

1. Specs live in `/specs/` as markdown with stable IDs.
2. Acceptance criteria are written in Given/When/Then form.
3. BDD lives in `/features/` as Gherkin, named by spec ID.
4. Tests live in `/tests/` using pytest-bdd, mapped to feature files by spec ID.
5. Code is written to make failing tests pass.
6. The traceability matrix is auto-generated from frontmatter and verified in CI.

Process detail in `00-meta/01-spec-driven-process.md`.

## Alternatives considered

- **Conventional Agile + Jira**: works at scale; overweight for a solo founder.
- **Pure TDD without BDD**: loses the audit-friendly behavior contract; harder for non-engineers
  to read.
- **Pure BDD without TDD**: leaves unit-level correctness underspecified for the orchestrator and
  parsing code.
- **No process — just ship**: appropriate for a hackathon. Not appropriate for a product whose
  customers will demand SOC 2 within 18 months.

## Consequences

**Positive**

- Every behavior is traceable to a written requirement. Audit story is structural.
- AI assistants (Claude Code) can execute each translation step deterministically. The codebase
  becomes AI-buildable.
- Onboarding new engineers is a matter of pointing them at `/specs/` and `00-meta/`.
- The same docs serve product (acceptance), engineering (tests), and audit (evidence).

**Negative**

- Front-loaded effort. Spec writing comes before any green code.
- Discipline required: it is always possible to skip the spec and write code that "looks right."
  Code review must enforce.
- Some kinds of exploratory work (UI experiments, performance spikes) chafe against the process.
  Allowed via a `spike/` folder that lives outside the chain, with an expiry date.

## Adopted in

- `CLAUDE.md` (Claude Code instructions)
- `00-meta/01-spec-driven-process.md` (the process spec)
- CI: `scripts/build_traceability.py` enforces the contract.
