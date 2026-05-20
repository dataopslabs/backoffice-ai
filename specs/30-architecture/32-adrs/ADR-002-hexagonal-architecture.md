---
id: ADR-002
title: Adopt hexagonal architecture (ports and adapters)
type: adr
status: approved
owner: founder
depends-on: [ADR-001]
version: 1.0.0
last-updated: 2026-05-19
---

# ADR-002: Adopt hexagonal architecture (ports and adapters)

## Context

BackOfficePilot integrates with several external systems whose APIs change on different
cadences: Bedrock (model versions), AgentCore (six services, each with its own GA path),
Nova Act, customer ERPs, banking portals, Slack/Teams. The domain logic — workflow planning,
exception classification, audit narration — must not be coupled to any of them.

Future-us will swap one of these (Bedrock → competing inference provider, Nova Act →
AgentCore Browser primary, NetSuite adapter → Sage adapter) at least once. The cost of those
swaps depends entirely on the architecture chosen now.

## Decision

Adopt **hexagonal architecture** (a.k.a. ports and adapters). The Python source tree is laid out
as:

```
src/
├── domain/             ← pure domain. Zero AWS imports. No HTTP, no DB.
├── application/        ← use cases and orchestration. Depends on domain only via ports.
├── adapters/           ← infrastructure. Implements ports. Talks to vendors.
│   ├── bedrock/
│   ├── agentcore/
│   ├── novaact/
│   ├── netsuite/
│   ├── slack/
│   └── ...
└── api/                ← FastAPI routers. Thin. Delegate to application.
```

Domain defines **ports** as abstract base classes (`ReasonerPort`, `ExecutorPort`,
`MemoryPort`, `EventBusPort`, ...). Adapters implement them. Dependency injection wires concrete
adapters at startup.

## Alternatives considered

- **Conventional layered architecture (presentation → business → data)**: simpler, but couples
  domain to infrastructure. Swap costs grow over time.
- **Onion architecture**: a stricter variant of hexagonal. Equivalent in practice for our scale;
  hexagonal terminology is more widely known.
- **Anemic services**: skip the discipline, push everything into FastAPI route handlers. Wins
  velocity now, loses everything else later.

## Consequences

**Positive**

- Domain tests are pure unit tests with no AWS, no network. Fast and deterministic.
- Adapters can be swapped without touching domain code. Vendor lock-in is structural, not coded.
- The pattern naturally accommodates multiple Executors (Nova Act + AgentCore Browser) behind
  one `ExecutorPort`.
- Aligns with DDD bounded contexts (`ADR` to be added if/when DDD is enforced more formally).

**Negative**

- More files and slightly more verbosity than a flat layout.
- Junior engineers may resist the ceremony. CLAUDE.md and code review must enforce.
- Risk of "anaemic domain" if developers push logic into application layer to avoid abstraction.
  Mitigation: code review and the rule "if you're testing it with mocks for AWS, the logic is in
  the wrong layer."

## Enforcement

A linter rule (custom `import-linter` config in `pyproject.toml`) fails CI if `domain/` or
`application/` imports anything from `adapters/` or any `boto3`/`anthropic` SDK package.
