---
id: REQ-PRD-013
title: Policy bundle versioning
type: req
status: approved
owner: founder
depends-on: [DESIGN-DATA-001]
covers-req: []
version: 0.1.0
last-updated: 2026-05-19
---

# REQ-PRD-013 — Policy bundle versioning

## Statement

The system **shall** maintain a versioned history of every customer's policy bundle, **shall**
associate each run with the policy bundle version active at run-start, and **shall** allow
authorized users to upload a new bundle without disrupting in-flight runs.

## Acceptance criteria

AC-1: Given a new policy bundle upload,
      when an authorized compliance officer submits it,
      then a new bundle version is created (semver-bumped) and the previous version is retained.

AC-2: Given an in-flight run started against version `1.4.2`,
      when version `1.5.0` is uploaded mid-run,
      then the run continues against `1.4.2`; subsequent runs use `1.5.0`.

AC-3: Given a published bundle version,
      when an audit query asks "which version was applied to run X",
      then the API returns the version, the file content hash, and a signed URL to the bundle.

AC-4: Given a rollback request,
      when an authorized user pins a prior bundle version as active,
      then subsequent runs use the pinned version; rollback is logged as an audit event.

## Implementation notes

- Policy bundle stored in versioned S3 bucket; metadata in DDB.
- Bundle hash recorded in the `Run` record for tamper-evidence linkage.
- Schema: `30-architecture/34-api-contract/schemas/policy_bundle.schema.json`.
