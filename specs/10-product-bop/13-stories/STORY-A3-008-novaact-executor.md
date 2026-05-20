---
id: STORY-A3-008
title: NovaActExecutor adapter (primary)
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-C4-005, ADR-011]
covers-req: []
bdd: [BDD-A3-008.feature]
tests: [tests/adapters/test_novaact_executor.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-008 — NovaActExecutor

## Story

As the founder, I need an adapter implementing `ExecutorPort` against Amazon Nova Act,
capable of logging into a mock banking portal and a sandboxed NetSuite, executing a step's
intent, and returning a structured `StepResult` with screenshots and observed values.

## Acceptance criteria

AC-1: Given a `Step` whose intent is "Download yesterday's transaction file from the bank
      portal",
      when `execute(step)` is invoked,
      then Nova Act logs in via TOTP, navigates to the export page, downloads the CSV, and the
      `StepResult.observations.file_s3_uri` is set.

AC-2: Given Nova Act cannot locate a required element,
      when `execute()` is invoked,
      then `StepResult.status == "failure"` and `failure_class == "element_not_found"`.

AC-3: Given a step's `timeout_seconds` is reached,
      when execution is still running,
      then the adapter cancels the session and returns `status=partial` with the partial
      observations.

AC-4: Given the executor advertises capabilities,
      when `capabilities()` is called,
      then it returns a non-empty set including at minimum `web-login`, `file-download`,
      `form-fill`.

AC-5: Given the run is in UAT environment,
      when the `ExecutorRouter` selects an executor,
      then `AgentCoreBrowserExecutor` is preferred over `NovaActExecutor` unless overridden.

## Technical notes

- Session pool: warm session per customer, idle timeout 5 minutes.
- TOTP secret in Secrets Manager; resolved at step dispatch.
- Screenshots stored to S3 under the run's prefix.
- Nova Act API errors mapped to structured `executor_error` codes.

## Definition of done

- [ ] Contract tests against `ExecutorPort` pass.
- [ ] Integration test against a mock banking portal in CI.
- [ ] Live test in UAT with a sandboxed NetSuite tenant.
