---
id: NFR-SEC-004
title: Encryption in transit
type: nfr
status: approved
owner: founder
depends-on: []
covers-req: [REQ-PRD-001]
version: 0.1.0
last-updated: 2026-05-19
---

# NFR-SEC-004 — Encryption in transit

## Statement

All network traffic **shall** use TLS 1.3 (TLS 1.2 acceptable for client compatibility on the
dashboard). No service-to-service communication **shall** use plaintext. mTLS **shall** be used
where supported (currently: AgentCore Gateway-to-tool calls when wrapping internal tools).

## Verification

- API Gateway TLS policy `TLS_1_3` enforced.
- CloudFront / ALB TLS policy `ELBSecurityPolicy-TLS13-1-2-2021-06`.
- AWS Config rule `alb-http-to-https-redirection-check`.
- Custom check in CDK synth tests asserts no `HTTP` listeners exist.

## Internal calls

- VPC endpoints used for AWS service calls so traffic stays on the AWS backbone.
- Bedrock, AgentCore service calls authenticated by IAM SigV4 over TLS 1.3.

## Browser-facing

- Strict-Transport-Security header (max-age ≥ 1 year, includeSubDomains, preload).
- Content-Security-Policy on dashboard responses.
