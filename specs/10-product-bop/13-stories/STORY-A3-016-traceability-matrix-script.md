---
id: STORY-A3-016
title: Traceability matrix generator script
type: story
status: approved
owner: founder
depends-on: [META-003]
covers-req: []
bdd: [BDD-A3-016.feature]
tests: [tests/scripts/test_traceability.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-016 — Traceability matrix generator

## Story

As the founder, I need `scripts/build_traceability.py` to walk `/specs`, `/features`, and
`/tests`, parse frontmatter, and produce the auto-section of `00-meta/03-traceability-matrix.md`
that CI verifies on every push.

## Acceptance criteria

AC-1: Given the script is invoked on the current spec set,
      when it runs,
      then it produces five sections (Req→Story, Story→BDD→Test, NFR→Coverage,
      ADR cross-ref, Orphans).

AC-2: Given a story whose `bdd:` field is empty,
      when the script runs,
      then the orphans section lists the story.

AC-3: Given a feature file `BDD-X-NNN.feature` not referenced by any story,
      when the script runs,
      then it is reported as an orphan.

AC-4: Given the script is run in CI and the regenerated matrix differs from the committed
      version,
      when the diff is non-empty,
      then the build fails.

AC-5: Given a `.md` file with frontmatter that fails schema validation (e.g. invalid `type`),
      when the script runs,
      then it reports the file path and the violation, and exits non-zero.

## Technical notes

- Pure Python, no external deps beyond `PyYAML` and `frontmatter` libraries.
- Cached file walk; idempotent.
- Emits markdown between `<!-- BEGIN/END auto -->` markers.

## Definition of done

- [ ] Script handles all five sections.
- [ ] Pre-commit hook configured.
- [ ] CI step `make verify-traceability` enforces no-diff invariant.
