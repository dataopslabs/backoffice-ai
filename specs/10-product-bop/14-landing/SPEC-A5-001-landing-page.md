---
id: SPEC-A5-001
title: BackOfficePilot landing page — specification
type: spec
status: approved
owner: founder
depends-on: []
covers-req: []
version: 0.2.0
last-updated: 2026-05-20
revision-history:
  - { version: 0.1.0, date: 2026-05-19, author: founder, note: "Initial spec for landing.html." }
  - { version: 0.2.0, date: 2026-05-20, author: founder, note: "Added what_it_solves.html companion page; landing.html updated with nav link + problem-section deep-dive link." }
---

# SPEC-A5-001 — Landing page specification

## Goal

A single-page, self-contained HTML landing page deployed at `backofficepilot.ai`, designed to
convert mid-market BFSI CFOs, COOs, and operations directors from a cold link into a
discovery call.

## Audience

Primary: CFOs, COOs, Heads of Operations at $1B–$50B-asset banks and regional insurers.
Secondary: investors finding us via the YC or Anthropic-AWS accelerator showcase.

## Brand

Brand matches the pitch deck (midnight executive palette):

| Token | Value |
|---|---|
| `--navy` | `#0B1F4D` (primary) |
| `--navy-dark` | `#07153A` |
| `--ice` | `#CADCFC` |
| `--gold` | `#F9C846` (accent) |
| `--teal` | `#0D9488` |
| `--coral` | `#F96167` |
| `--muted` | `#8FA3C1` |
| `--text-dark` | `#1E293B` |
| Font (headers) | Calibri / system-ui fallback |
| Font (body) | Calibri / system-ui fallback |

## Sections (top to bottom)

1. **Hero**
   - Headline: "Multi-Model AI Agents for the BFSI Back-Office"
   - Subhead: "Claude Opus reasons. Amazon Nova Act acts. Your team supervises."
   - One-line value prop + primary CTA ("Book a 15-minute call") + secondary CTA ("Read the
     proposal").
   - Background: navy with the same dot motif from slide 1 of the deck.

2. **The problem (one-liner stat strip)**
   - Three stat callouts (same as deck slide 2):
     - 20–40% of operating payroll is consumed by manual back-office workflows.
     - $15–$60 per case for a single KYC review.
     - 25–35% annual attrition in BFSI operations roles.

3. **How it works**
   - Three-step diagram: Claude Opus (Reasoner) → Orchestrator → Nova Act (Executor).
   - 2-sentence explanation per role.

4. **A typical day**
   - Visual stepped flow (matches deck slide 5):
     - 07:00 AM — Trigger
     - +0:04 — Plan (Opus)
     - +0:30 — Execute (Nova Act)
     - +2:20 — Match + post
     - +5:45 — Audit + escalate
   - Result strip: "47 transactions reconciled in 5:45 · $0.73 per run · 21× ROI."

5. **Why mid-market BFSI**
   - Three pillars: regulated, legacy ERPs, mid-market scale.
   - Brief story of how prior automation waves missed this customer.

6. **Cost-savings calculator (interactive)**
   - Three inputs: workflow type (dropdown), monthly volume, fully-loaded hourly cost.
   - Output: monthly $ saved, annual $ saved, payback period.
   - All in-page JavaScript; no backend.

7. **Security + compliance one-pager**
   - Six tiles: HIPAA-ready, AES-256 at rest, TLS 1.3 in transit, audit log immutability,
     SOC 2 in progress, VPC isolation.

8. **Lead capture form**
   - Fields: name, work email, company, role, "What back-office workflow are you trying to
     fix?" (free text).
   - Submit → POST to webhook URL stored in `LANDING_LEAD_WEBHOOK` env or fallback mailto:
     link.
   - Privacy: a one-line GDPR-compliant statement; link to privacy policy stub.

9. **Footer**
   - Founder name + contact.
   - Investor logos (placeholders for accelerator badges once accepted).
   - Year + © + privacy + terms (stubs).

## Calls to action

Primary CTA appears in: hero, after the "Day" section, in the footer.
Secondary CTA (read proposal): hero only.

## Technical constraints

- Single HTML file. No build step. Inline `<style>` and `<script>`.
- No external requests except a single Google Font load (optional; system font fallback).
- Mobile-responsive (tested down to 360px width).
- LCP target < 1.5s on a mid-tier mobile device.
- Accessibility: WCAG 2.2 AA.
- File size target: < 80 KB.

## Lead-capture wiring

- Form posts JSON to `LANDING_LEAD_WEBHOOK` if set in a `<script>` placeholder.
- Default: opens a mailto: with the form contents prefilled.
- We will swap in a Formspree, ConvertKit, or webhook endpoint once chosen.

## Future revisions (not in v1)

- Animated demo embed (replaces static "A typical day" section).
- Customer logo strip once we have signed pilots.
- Investor / accelerator badge strip once accepted.
- A/B test framework for headline copy.

---

## Companion page: `what_it_solves.html` (added v0.2.0)

A long-form explainer designed to convert sophisticated buyers who want the *why* before they
book a call. Linked from the landing nav ("What we solve") and from a sentence under the
problem section ("Read the deep dive →").

### Audience

Same as the landing page primary audience — CFOs, COOs, Heads of Operations at mid-market
BFSI firms — but specifically those who arrived via a content link, a press mention, or a
warm intro and want depth before scheduling a call.

### Sections

1. Page head + table of contents (8 anchor links).
2. The status quo — three composite scenarios (loan ops clerk, reconciliation analyst, KYC
   reviewer) with time/cost/error stats.
3. Anatomy of the pain — five workflow pain cards (invoice reconciliation, KYC, loan ops,
   account recon, regulatory filing) + a shared-signature card.
4. Why prior automation missed this customer — 4-row comparison table covering Big-4
   consultants, enterprise RPA, copilots, generalist agents.
5. The compounding damage — four damage cards (linear headcount, compliance debt, attrition,
   downturn risk).
6. How BackOfficePilot solves it — six solution cards (two brains, orchestrator, audit log,
   drives existing UIs, outcome-priced, leave-behind).
7. Before / After operating-model change — two-column comparison.
8. Per-workflow ROI math — four cards with concrete numbers per workflow type.
9. Industry breakdown — banks, insurance, asset mgmt, fintechs (year 1) + healthcare RCM
   (year 2).
10. CTA + pilot terms — two CTAs back to the landing page contact form.

### Brand consistency

Uses the same CSS custom-property palette, font stack, brand mark, and nav shell as the
landing page. Both files include the brand mark SVG inline so they can be deployed as static
HTML to any host (Vercel, S3+CloudFront, Netlify) without external assets.

### Cross-linking

- `what_it_solves.html` nav → links back to landing sections (How it works, Calculator,
  Security, Book a call).
- Landing nav → "What we solve" links to `what_it_solves.html`.
- Landing problem section → "Read the deep dive →" link.
- CTA at the bottom of `what_it_solves.html` → both "Book a call" (landing.html#contact)
  and "Back to the landing page" (landing.html).
