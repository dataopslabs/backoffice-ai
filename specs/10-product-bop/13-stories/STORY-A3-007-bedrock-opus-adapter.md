---
id: STORY-A3-007
title: BedrockOpusReasoner adapter (plan + reconcile)
type: story
status: approved
owner: founder
depends-on: [STORY-A3-005, DESIGN-C4-004, ADR-004]
covers-req: []
bdd: [BDD-A3-007.feature]
tests: [tests/adapters/test_bedrock_opus.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-007 — BedrockOpusReasoner

## Story

As the founder, I need an adapter that implements `ReasonerPort` against Amazon Bedrock with
Claude Opus 4.6, supporting both `plan()` and `reconcile()` calls, prompt caching, JSON schema
validation, and one-retry on schema failure.

## Acceptance criteria

AC-1: Given a valid `PlanRequest` with a workflow + policy bundle,
      when `plan()` is invoked,
      then a `Plan` object that satisfies the plan JSON schema is returned.

AC-2: Given Bedrock returns malformed JSON,
      when `plan()` is invoked,
      then the adapter retries once with a "previous output was invalid" hint; on second
      failure it raises `SchemaValidationError`.

AC-3: Given the customer's policy bundle and workflow definition are stable across runs,
      when `plan()` is invoked,
      then the cached fraction reported by Bedrock is ≥ 80%.

AC-4: Given a successful invocation,
      when `plan()` returns,
      then `BedrockInvocationCompleted` is emitted with input/output token counts and cached
      fraction.

AC-5: Given a recorded VCR cassette,
      when CI runs,
      then deterministic test fixtures replay without hitting Bedrock.

## Technical notes

- SDK: AWS Bedrock Runtime (`bedrock-runtime:InvokeModel`).
- Cache prefix marked via Bedrock's prompt-caching syntax.
- Prompt templates in `src/adapters/bedrock/prompts/`.
- Live integration test runs nightly, not in PR CI.

## Definition of done

- [ ] Adapter passes contract tests against `ReasonerPort`.
- [ ] Cassette suite covers plan, reconcile, malformed-JSON retry, low-confidence escalation.
- [ ] Live test executes successfully in UAT.
