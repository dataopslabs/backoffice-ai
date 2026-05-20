---
id: STORY-B3-005
title: Mock data generator script
type: story
status: approved
owner: founder
depends-on: [REQ-B1-004]
covers-req: [REQ-B1-004]
bdd: []
tests: [tests/scripts/test_gen_mock_data.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-005 — Mock data generator

## Story

As the founder, I need `scripts/gen_mock_recon_data.py` to produce a daily batch of synthetic
data that exercises every exception category, with deterministic seeds for reproducibility.

## Acceptance criteria

AC-1: Given the generator is invoked with `--seed 42 --date 2026-06-01`,
      when re-run with the same arguments,
      then the produced CSVs are byte-identical.

AC-2: Given a generated batch,
      when inspected,
      then it includes 4–6 injected exceptions covering all four categories.

AC-3: Given the generator outputs CSVs to S3,
      when the mock bank portal restarts,
      then it loads the latest batch.

AC-4: Given the script's PII check,
      when run,
      then no field matches the PII regex set (catches developer typos that paste real data).

## Definition of done

- [ ] Script committed under `scripts/`.
- [ ] CI test verifies determinism on three seeds.
- [ ] Mock bank portal reads from S3 on startup.
