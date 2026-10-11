# Security Policy

## Project status

SatsVerify is experimental and is not ready for production payment decisions. Do not send real customer data, private keys, access tokens, or payment preimages in issue reports.

## Reporting a vulnerability

Please do not publish exploitable security details in a public issue.

Until a dedicated private reporting channel is configured, contact the repository maintainer privately through a verified contact method associated with the GitHub account `Ugarba202`. Include:

- A concise description of the issue and its impact.
- Affected commit, component, or version.
- Minimal reproduction steps or a proof of concept.
- Any suggested mitigation.

Do not include real credentials or sensitive payment data. If the repository's GitHub Security Advisories private reporting feature is enabled, use it.

## Areas of particular concern

- Unauthorized cross-merchant invoice access.
- Incorrect settlement-state transitions.
- Forged, altered, stale, or replayed receipts and events.
- Signing-key exposure or invalid trust-key handling.
- Idempotency failures that create unintended invoices.
- Disclosure of payment preimages, credentials, or personal data.
- USSD session authorization and rate-limit bypasses.

## Response

The maintainers will assess reports, attempt to reproduce the issue, determine its impact, and coordinate a fix and disclosure. No response-time guarantee is made while the project is maintained by a small buildathon team.

Please do not assume a report is resolved until a maintainer confirms the fix.
