---
id: META-002
title: Naming Conventions for Specs, BDD, and Tests
type: spec
status: approved
owner: founder
depends-on: [META-001]
version: 0.1.0
last-updated: 2026-05-19
---

# Naming Conventions

This document is the authoritative source for how files, IDs, and references are named in this
repository. New contributors and Claude Code sessions read this before creating any spec file.

## File naming grammar

```
<TYPE>-<DOMAIN>-<NUMBER>-<kebab-slug>.md
```

| Slot | Allowed values | Example |
|---|---|---|
| `<TYPE>` | One of the registered type prefixes (see below). | `STORY` |
| `<DOMAIN>` | A 1–4 character domain code (see below). | `B3` |
| `<NUMBER>` | Zero-padded 3-digit number, unique within `<TYPE>-<DOMAIN>`. | `027` |
| `<kebab-slug>` | Lowercase, hyphenated, short (3–6 words). Descriptive. | `wire-agentcore-gateway-tool` |

Full example: `STORY-B3-027-wire-agentcore-gateway-tool.md`

The combination `<TYPE>-<DOMAIN>-<NUMBER>` is the **stable ID**. It survives renames of the
kebab-slug and is referenced everywhere (frontmatter, BDD file names, commit messages, the
traceability matrix).

## Type prefixes

| Prefix | Meaning | Lives in |
|---|---|---|
| `REQ` | Functional requirement (PRD-level). | `11-requirements/` (Stream A), `21-requirements/` (Stream B) |
| `NFR` | Non-functional requirement (security, performance, compliance, observability). | `40-non-functional/` |
| `EPIC` | A grouping of related stories. | `13-stories/`, `23-stories/` |
| `STORY` | An implementable unit of work with ACs. | `13-stories/`, `23-stories/` |
| `DESIGN` | A technical design document for a component or integration. | `12-design/`, `22-design/` |
| `ADR` | Architecture Decision Record. | `30-architecture/32-adrs/` |
| `SPEC` | A formal specification that does not fit another type (e.g. event catalog, API contract intro). | varies |
| `META` | Meta-spec about the process itself. | `00-meta/` |
| `BDD` | Gherkin feature file (not in `/specs`; lives in `/features` at the repo root). | `/features/<domain>/` |
| `TEST` | Test module (not in `/specs`; lives in `/tests`). | `/tests/<domain>/` |

A prefix is added only if a new category truly cannot fit any existing one. Adding a new prefix
requires an ADR.

## Domain codes

| Code | Domain | Used for |
|---|---|---|
| `PRD` | Product Requirements (BackOfficePilot platform-wide) | `REQ-PRD-*` |
| `A` | Stream A — BackOfficePilot product | `EPIC-A-*`, `STORY-A-*` |
| `A1`..`A5` | Stream A subsections (requirements, design, MVP stories, full-blown stories, landing) | `STORY-A3-*`, `STORY-A4-*`, etc. |
| `B` | Stream B — Mock invoice reconciliation use case | `EPIC-B-*`, `STORY-B-*` |
| `B1`..`B5` | Stream B subsections (requirements, design, MVP build, full-blown, runbook) | `STORY-B3-*`, etc. |
| `SEC` | Security NFRs | `NFR-SEC-*` |
| `PERF` | Performance NFRs | `NFR-PERF-*` |
| `OBS` | Observability NFRs | `NFR-OBS-*` |
| `COMP` | Compliance / regulatory NFRs | `NFR-COMP-*` |
| `COST` | Cost-control NFRs | `NFR-COST-*` |
| `RUN` | Reconciliation workflow domain (in `30-architecture/`) | `DESIGN-RUN-*` |
| `GW` | AgentCore Gateway domain | `DESIGN-GW-*` |
| `MEM` | AgentCore Memory domain | `DESIGN-MEM-*` |
| `REG` | AgentCore Registry domain | `DESIGN-REG-*` |
| `EXEC` | Executor layer (Nova Act + AgentCore Browser) | `DESIGN-EXEC-*` |
| `RSN` | Reasoner layer (Claude Opus) | `DESIGN-RSN-*` |

New domain codes are added in this table when introduced. Each addition must be justified in the
PR description (or ADR if it implies a new bounded context).

## Numbering rules

- Numbers are zero-padded to 3 digits: `001`, `017`, `204`.
- Numbers are sequential within `<TYPE>-<DOMAIN>` and never reused, even after deletion.
- Gaps are allowed; do not renumber to fill them.
- If a spec is superseded, the new spec takes the next available number; the old one keeps its
  number with `status: superseded`.

## Frontmatter schema

Every `.md` file under `/specs/` (except this `README`-equivalent set) starts with YAML frontmatter:

```yaml
---
id: STORY-B3-027
title: Wire AgentCore Gateway tool for NetSuite export
type: story
status: draft
owner: founder
depends-on: [DESIGN-A2-003, REQ-PRD-018]
covers-req: [REQ-PRD-018, REQ-PRD-019]
bdd: [BDD-B3-027.feature]
tests:
  - tests/recon/test_gateway_netsuite.py::test_export_invoices_succeeds
  - tests/recon/test_gateway_netsuite.py::test_expired_credentials_trigger_rotation
version: 0.1.0
last-updated: 2026-05-19
revision-history:
  - { version: 0.1.0, date: 2026-05-19, author: founder, note: "Initial draft." }
---
```

### Field reference

| Field | Required on | Notes |
|---|---|---|
| `id` | all | Immutable. Matches the filename's `<TYPE>-<DOMAIN>-<NUMBER>`. |
| `title` | all | Short human-readable title. May change without bumping version. |
| `type` | all | One of: `req`, `nfr`, `epic`, `story`, `design`, `adr`, `spec`, `meta`. Lowercase. |
| `status` | all | `draft` → `approved` → `in-progress` → `done`, plus `superseded`. |
| `owner` | all | GitHub handle or role. |
| `depends-on` | all | List of spec IDs this spec depends on. Empty list `[]` allowed. |
| `covers-req` | stories, designs | List of REQ/NFR IDs this spec implements or addresses. |
| `bdd` | stories | List of feature file names. Empty until the BDD is authored. |
| `tests` | stories | List of test addresses (path::function). |
| `version` | all | semver. See `01-spec-driven-process.md` for bump rules. |
| `last-updated` | all | ISO date. |
| `revision-history` | optional | Append a row on every published change. |
| `supersedes` | only on replacement specs | The ID of the old spec being replaced. |
| `superseded-by` | only on superseded specs | The ID of the new spec. |

## Examples — good and bad

**Good**

```
REQ-PRD-014-audit-log-retention.md
NFR-SEC-002-vpc-isolation-prod-uat.md
DESIGN-GW-003-tool-registration-flow.md
STORY-B3-027-wire-agentcore-gateway-tool.md
ADR-007-pytest-bdd-over-behave.md
```

**Bad — fix before committing**

| Filename | Problem |
|---|---|
| `story-b3-27-tool.md` | Type prefix lowercase; number not padded; slug too vague. |
| `STORY-B-027-Wire_Gateway.md` | Domain missing subsection; slug has underscores and capitals. |
| `STORY-B3-0027-wire-agentcore-gateway-tool-for-netsuite-export-with-fallback.md` | Number over-padded; slug too long (>6 words). |
| `STORY-B3-027.md` | Missing slug. |

## Cross-references in prose

Within markdown body text, reference other specs by ID in inline code:

```
This story implements `REQ-PRD-018` and depends on the design in `DESIGN-A2-003`.
```

Do **not** use markdown links to file paths — paths change, IDs do not. The traceability matrix
maintains the path mapping.

## Naming downstream artifacts

| Artifact | Pattern | Example |
|---|---|---|
| BDD feature file | `BDD-<DOMAIN>-<NUMBER>.feature` | `BDD-B3-027.feature` |
| Test module | `test_<kebab-slug>.py` | `test_gateway_netsuite.py` |
| ADR | `ADR-<NUMBER>-<kebab-slug>.md` | `ADR-007-pytest-bdd-over-behave.md` |
| Workflow definition | `<workflow-name>.yaml` | `daily_reconciliation.yaml` |
| Event class | `PascalCase` ending in past-tense verb | `RunCompleted`, `ExceptionRaised` |
| Domain port (Python ABC) | `PascalCase` ending in `Port` | `ReasonerPort`, `ExecutorPort` |
| Adapter implementation | `PascalCase` ending in vendor name | `BedrockReasonerAdapter`, `NovaActExecutorAdapter` |

## What this enables

- A Claude Code session can resolve any `<ID>` to a file path with a single grep.
- The traceability matrix is generated mechanically by parsing frontmatter.
- BDD and test files have a one-to-one mapping to stories — no orphan tests, no orphan specs.
- New domains plug in without renaming anything that already exists.
