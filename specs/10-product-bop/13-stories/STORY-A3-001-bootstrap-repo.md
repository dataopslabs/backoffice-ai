---
id: STORY-A3-001
title: Bootstrap repo, dev environment, and CI skeleton
type: story
status: approved
owner: founder
depends-on: []
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-001 — Bootstrap repo

## Story

As the founder, I need a Git repository with a working Python + Next.js dev environment and a
CI pipeline that runs linters and tests on every push, so that subsequent stories can rely on
green-build feedback from day one.

## Acceptance criteria

AC-1: Given a fresh clone of the repo,
      when I run `make bootstrap`,
      then `uv sync` installs Python deps, `pnpm install` installs Next.js deps, and a
      `.env.example` is copied to `.env`.

AC-2: Given a push to any branch,
      when CI runs,
      then ruff, mypy, pytest (with -m unit), eslint, tsc, and prettier checks all run.

AC-3: Given a PR opens,
      when CI completes,
      then the traceability matrix is regenerated and any orphan stories cause failure.

AC-4: Given the repository,
      when an engineer inspects the README,
      then they see how to install, run, and contribute, with links to `specs/CLAUDE.md`.

## Technical notes

- Layout per `CLAUDE.md`: `/src`, `/web`, `/specs`, `/features`, `/tests`, `/infra`, `/workflows`.
- Pre-commit hooks: ruff format, eslint --fix, trailing whitespace.
- GitHub Actions workflows: `ci.yml` (lint+test), `deploy-uat.yml` (manual trigger),
  `traceability.yml` (PR check).

## Definition of done

- [ ] `make bootstrap` works on a clean machine.
- [ ] CI passes on the first PR.
- [ ] README explains the spec-driven workflow.
- [ ] `.editorconfig` and `.gitattributes` committed.
