---
id: STORY-B3-001
title: Mock Acme bank treasury portal
type: story
status: approved
owner: founder
depends-on: [REQ-B1-002]
covers-req: [REQ-B1-002]
bdd: [BDD-B3-001.feature]
tests: [tests/mock_bank/test_login.py, tests/mock_bank/test_export.py]
version: 0.1.0
last-updated: 2026-05-19
---

# STORY-B3-001 — Mock Acme bank portal

## Story

As the founder, I need a small Flask app that simulates the Acme bank treasury portal —
login (username + password + TOTP), navigate to "Daily Transactions" page, export CSV — so
that Nova Act has something concrete to drive in UAT.

## Acceptance criteria

AC-1: Given the app is deployed on Vercel at `https://acme-bank-mock.bop-uat.com`,
      when an HTTP client visits `/`,
      then a login form renders (username + password fields).

AC-2: Given correct credentials + a valid TOTP,
      when the form is submitted,
      then the user is redirected to the dashboard; an HTTP-only session cookie is set.

AC-3: Given a session cookie,
      when the user visits `/transactions?date=YYYY-MM-DD`,
      then a list view renders with a "Download CSV" link.

AC-4: Given the CSV link is clicked,
      when the request is processed,
      then a CSV file is downloaded with the day's mock data.

AC-5: Given Nova Act executes the workflow,
      when it scripts the above steps,
      then it succeeds without manual intervention.

## Technical notes

- Flask + Jinja templates; gunicorn under Vercel Python runtime.
- TOTP via `pyotp`; secret stored in `cust/acme/bank/credentials` (UAT only).
- Mock data sourced from `STORY-B3-005` generator on startup.

## Definition of done

- [ ] Deployed and reachable from the UAT VPC NAT egress IPs.
- [ ] Nova Act test playbook passes against it.
- [ ] Source code in `mock-services/acme-bank/`.
