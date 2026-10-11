# SatsVerify — Engineering Roadmap

**Target:** Build day October 29, 2026; showcase October 31, 2026.

This roadmap follows the P0/P1 scope and acceptance criteria in [PRD.md](PRD.md). Dates are targets, not permission to skip release gates. Voice IVR remains optional.

## E0 — Repository foundation and governance

**Deliverables**
- PRD committed.
- README, architecture, threat model, and roadmap.
- Contribution, security-reporting, and community guidelines.
- Minimal CI and pull-request workflow.

**Exit criteria**
- Documents agree on project scope and trust assumptions.
- Existing files are preserved.
- CI checks the repository's initial quality gates.

## E1 — Lightning integration feasibility

**Priority:** First engineering gate.

**Deliverables**
- Short comparison of viable node/SDK integrations.
- One selected integration and documented rationale.
- Repeatable test environment.
- Invoice creation and authoritative settlement observation.
- Notes on event semantics, retries, expiry, and reconnection.

**Exit criteria**
- A reproducible test demonstrates an invoice moving from unpaid to settled based on the selected payment source.
- Test/simulated settlement is labelled accurately.

Do not build broad API or channel features before this gate passes.

## E2 — Verification domain

**Deliverables**
- Invoice and merchant domain types.
- Explicit state machine and permitted transitions.
- Event validation, deduplication, and audit events.
- Tests for valid, stale, duplicate, expired, and invalid transitions.

**Exit criteria**
- Invalid transitions are rejected and duplicate events do not cause unintended side effects.

## E3 — Persistence and merchant API

**Deliverables**
- PostgreSQL schema and migrations.
- Invoice registry and settlement metadata.
- Authenticated or appropriately access-controlled status endpoint.
- Merchant isolation and idempotency for mutations.
- Consistent error responses and bounded requests.

**Exit criteria**
- Status reads return the correct state and unauthorized merchants cannot read each other's invoices.

## E4 — Signed settlement receipts

**Deliverables**
- Versioned canonical receipt format.
- Documented asymmetric signing algorithm and key format.
- Protected signing-key configuration.
- Key rotation/revocation procedure.
- Tests for valid, altered, invalid, and expired receipts.

**Exit criteria**
- Receipt authenticity is reported separately from the issuer's settlement assertion.

## E5 — Independent CLI verifier

**Deliverables**
- Receipt parsing and protocol-version validation.
- Signature verification against an explicitly trusted key.
- Optional preimage/hash check.
- Clear machine-readable and human-readable results.

**Exit criteria**
- The CLI rejects invalid data and never claims more than the checks establish.

## E6 — USSD adapter

**Deliverables**
- Compatible provider sandbox or labelled local demonstration adapter.
- Status lookup for an existing invoice.
- Server-side session correlation, authorization, and rate limiting.
- Documented session behavior and cost assumptions.

**Exit criteria**
- The phone interface returns the same authoritative state as the API without treating session IDs as identity proof.

## E7 — Flutter verifier

**Deliverables**
- Receipt/reference input or supported scan flow.
- Payment-state and issuer display.
- Local signature and hash checks where data and trusted keys are available.
- Clear explanation of online-only checks and offline limitations.

**Exit criteria**
- Supported offline checks work with previously obtained receipt data and trusted keys.

## E8 — Reliability and security

**Deliverables**
- Automated critical-path tests.
- Secret-handling and data-disclosure review.
- Rate limits and request bounds.
- Setup instructions and repeatable test commands.
- Known limitations and failure-mode documentation.

**Exit criteria**
- Critical security and duplicate-processing cases pass and are demonstrable.

## E9 — Voice IVR (stretch)

Start only if E1–E8 P0 work is stable. Use deterministic prompts and DTMF; reuse the same verification API.

## E10 — Showcase preparation

**Deliverables**
- External-wallet end-to-end path where the selected environment permits it.
- Repeatable regtest fallback labelled as a test.
- Demo completed in under three minutes.
- At least five rehearsals.
- Clear prior-art comparison and honest trust-model explanation.

## Scope-control rule

When schedule pressure rises, reduce optional scope in this order: voice IVR, extra node adapters, receipt history, Hausa prompts, and nonessential UI polish. Preserve the authoritative settlement path, receipt verification, access control, and critical tests.
