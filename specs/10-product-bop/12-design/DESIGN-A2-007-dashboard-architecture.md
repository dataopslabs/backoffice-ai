---
id: DESIGN-A2-007
title: Dashboard architecture
type: design
status: approved
owner: founder
depends-on: [REQ-PRD-010, REQ-PRD-011, REQ-PRD-012, REQ-PRD-016]
covers-req: [REQ-PRD-010, REQ-PRD-011, REQ-PRD-012]
version: 0.1.0
last-updated: 2026-05-19
---

# DESIGN-A2-007 — Dashboard architecture

## Stack

- **Next.js 14** App Router, TypeScript strict mode.
- **Tailwind CSS** + **shadcn/ui** for component primitives. Brand palette from the pitch deck
  (navy, ice, gold, teal accent).
- **TanStack Query** for client-side data caching; React Server Components for first paint.
- **OpenAPI-generated client** in `web/lib/_generated/`.
- **Recharts** for cost charts.
- **Server-Sent Events** for live run/escalation streaming.

## App Router structure

```
web/app/
├── layout.tsx                      ← shell, auth boundary
├── (auth)/login/page.tsx
├── (customer)/
│   ├── layout.tsx                  ← customer-scoped shell
│   ├── page.tsx                    ← dashboard home (run list)
│   ├── runs/[runId]/page.tsx
│   ├── escalations/page.tsx
│   ├── cost/page.tsx
│   ├── workflows/page.tsx
│   ├── workflows/[wfId]/page.tsx
│   └── policy-bundles/page.tsx
└── (operator)/
    ├── layout.tsx
    ├── customers/page.tsx
    ├── customers/[id]/page.tsx
    └── reconciliation/page.tsx
```

## Authentication

- Cognito hosted UI for sign-in.
- JWT in `httpOnly` secure cookie (rotated via short-lived refresh token).
- Next.js middleware validates the cookie and injects `customer_id` into the request scope.
- The (customer) group enforces customer membership; the (operator) group requires `operator`
  scope.

## Data fetching pattern

- **First paint**: server component fetches initial data via the generated client (using a
  service token derived from the user's cookie).
- **Hydration**: client component takes over with TanStack Query for updates.
- **Streaming**: SSE component opens a connection on mount and `invalidateQueries` on event.

## Component-level conventions

- All forms use react-hook-form + zod schemas generated from OpenAPI.
- All charts in `components/charts/`; never inline chart logic.
- Brand colors in `tailwind.config.ts`:
  - `navy: "#0B1F4D"`
  - `ice: "#CADCFC"`
  - `gold: "#F9C846"`
  - `teal: "#0D9488"`
  - `coral: "#F96167"` (accent)

## Pages

| Page | Primary data |
|---|---|
| Home (run list) | `listRuns` + filters |
| Run detail | `getRun`, `listSteps`, `getRunAudit`, `getRunCost`, SSE stream |
| Exception queue | `listEscalations`, SSE stream |
| Cost | `getRunCost` (recent) + `CostDaily` rollup endpoints |
| Workflows | `listWorkflows`, `getWorkflow`; operator can `upsertWorkflow` |
| Policy bundles | `listPolicyBundles`, `getPolicyBundle`; upload via `upsertPolicyBundle` |

## Performance

- App Router prefetch on hover for navigation targets.
- Suspense + skeleton placeholders for slow data; never spinner-only.
- Brotli-compressed responses; CDN-cached static assets.
- Target LCP < 1.5s; INP < 200ms.

## Accessibility

- WCAG 2.2 AA target.
- Keyboard-only navigation tested in CI via `axe-core` Playwright suite.
- High-contrast mode honored.
