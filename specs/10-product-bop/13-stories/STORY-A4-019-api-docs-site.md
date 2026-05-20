---
id: STORY-A4-019
title: Public customer-facing API documentation site
type: story
status: draft
owner: founder
depends-on: [SPEC-API-001]
covers-req: []
bdd: []
tests: []
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A4-019 — API docs site

## Story

As a customer engineer, I need a published documentation site for the BackOfficePilot API so I
can integrate webhooks, automation, and BI tools without help from the operator team.

## Acceptance criteria

AC-1: Given the OpenAPI document,
      when the docs site is built,
      then it renders every operation, request/response example, error schema, and scope
      requirement.

AC-2: Given a customer wants to "try it,"
      when they paste a JWT into the site,
      then they can execute live requests against the UAT API.

AC-3: Given the OpenAPI document changes,
      when CI runs,
      then the docs site is rebuilt and redeployed automatically.

## Definition of done

- [ ] Docs site live at `docs.backofficepilot.ai`.
- [ ] "Try it" feature uses UAT only, never Prod.
- [ ] Custom 404 + brand-styled.
