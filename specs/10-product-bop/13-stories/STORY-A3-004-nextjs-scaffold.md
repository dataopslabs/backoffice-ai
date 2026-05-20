---
id: STORY-A3-004
title: Next.js dashboard scaffold and Cognito auth
type: story
status: approved
owner: founder
depends-on: [STORY-A3-003, DESIGN-A2-007]
covers-req: [REQ-PRD-016]
bdd: [BDD-A3-004.feature]
tests: [web/__tests__/auth.spec.ts]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-A3-004 — Next.js dashboard scaffold

## Story

As the founder, I need a Next.js 14 App Router application with Cognito-backed authentication,
Tailwind + shadcn/ui set up, and a stub home page that uses the generated OpenAPI client, so
that subsequent dashboard stories build on it.

## Acceptance criteria

AC-1: Given a fresh `pnpm dev`,
      when I open `http://localhost:3000`,
      then I am redirected to Cognito sign-in.

AC-2: Given a signed-in user,
      when they navigate to `/`,
      then the dashboard home page renders with the brand palette (navy header, gold accent).

AC-3: Given a customer user (not operator),
      when they attempt to visit `/operator`,
      then they are redirected to `/` with a "not authorized" toast.

AC-4: Given the generated TypeScript client,
      when the home page server-component calls `listRuns`,
      then a stubbed response is rendered (real data wired in later stories).

AC-5: Given a Playwright test,
      when it exercises the auth flow,
      then it succeeds in CI against a mock Cognito.

## Technical notes

- App Router with route groups `(customer)` and `(operator)`.
- shadcn/ui configured per `DESIGN-A2-007`.
- Auth cookie is `httpOnly`, `Secure`, `SameSite=Lax`.
- Mock Cognito: aws-jwt-verify with a local issuer for tests.

## Definition of done

- [ ] `pnpm dev` runs cleanly.
- [ ] Auth flow E2E test passes in CI.
- [ ] Brand palette in `tailwind.config.ts`.
- [ ] Lighthouse score ≥ 90 on home page.
