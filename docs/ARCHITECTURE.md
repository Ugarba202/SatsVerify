# SatsVerify — Architecture

**Status:** Proposed architecture; implementation decisions remain subject to feasibility testing.

## 1. Design goals

- Keep payment observation separate from user-facing channels.
- Use one authoritative source for the settlement state of invoices the service monitors.
- Make receipt evidence and trust assumptions explicit.
- Prevent cross-merchant data access and unintended duplicate operations.
- Make the core testable without requiring a live telecom provider.
- Prefer the smallest viable implementation for the buildathon.

## 2. Logical components

### Channel adapters

- REST clients and merchant integrations.
- USSD gateway adapter.
- Minimal Flutter verifier.
- Optional deterministic voice IVR, only after P0 is stable.

Adapters call the same verification service. They must not maintain separate sources of payment truth.

### Verification service

The proposed backend uses Rust and Axum and is responsible for:

- Merchant and integration authorization.
- Invoice issuance requests.
- Invoice registry and state machine.
- Idempotency and event deduplication.
- Settlement observation from the selected Lightning integration.
- Payment-status responses.
- Canonical receipt generation and asymmetric signing.
- Audit events and bounded, rate-limited requests.

### Persistence

PostgreSQL is the proposed store for invoice metadata, merchant scope, state transitions, idempotency records, and receipt metadata.

Signing keys must be handled separately from ordinary application data and must never be committed to the repository.

### Lightning integration boundary

A dedicated adapter isolates node- or SDK-specific invoice creation and settlement events from the domain model.

The exact node or SDK is **not yet selected**. The initial feasibility spike must establish:

1. Repeatable invoice creation.
2. Access to invoice identifiers and payment hashes.
3. Authoritative settlement observation.
4. Event or polling semantics, including duplicates and reconnect behavior.
5. A reproducible test environment.

Do not implement provider-specific behavior until this spike informs the interface.

## 3. Proposed settlement flow

1. An authorized merchant requests an invoice with an idempotency key.
2. The service requests invoice creation from the configured Lightning integration.
3. The service stores the invoice reference, payment hash, amount, expiry, merchant scope, and initial state.
4. The observer receives or polls payment state from the configured source.
5. The service validates the event, deduplicates it, and applies a permitted state transition.
6. Once settlement is authoritatively observed, the service records the transition and prepares a receipt.
7. The merchant API returns the current state and receipt reference.
8. The CLI or Flutter verifier checks the receipt signature and any supplied preimage/hash relationship, while reporting the limits of those checks.
9. The USSD adapter retrieves the same server-side status through a constrained, access-controlled flow.

A payer-provided screenshot or message must never transition an invoice to settled.

## 4. Invoice state model

The PRD proposes the primary path:

`OPEN → IN_FLIGHT → SETTLED`

`FAILED` and `EXPIRED` are alternative terminal states, with exact transitions defined according to the selected integration's documented semantics.

The implementation must define permitted transitions, handle late or stale events, and prevent a duplicate event from causing unintended side effects.

## 5. Receipt and verification boundary

The receipt contains a versioned, canonical payload and an asymmetric signature. The payload identifies the issuer and describes the evidence available.

The verifier checks:

- Payload structure and supported protocol version.
- Signature against a trusted public key.
- Preimage/hash consistency if the preimage is provided.
- Timestamp, expiry, and other freshness policy defined by the protocol.

A valid signature authenticates the issuer's signed statement; it does not prove the issuer is honest. A preimage/hash match does not by itself prove settlement or the amount paid. Offline verification cannot discover new settlement information.

The canonical encoding, signature algorithm, key rotation procedure, and revocation policy must be explicitly specified before implementation is considered complete.

## 6. Security boundaries

- Every invoice and status query is scoped to the authorized merchant or integration.
- Mutating requests use idempotency keys.
- Observer events are validated and processed idempotently.
- Sensitive values are excluded from ordinary logs.
- Secrets are supplied through an appropriate runtime secret mechanism, not source files.
- USSD session identifiers are correlation values, not proof of identity.
- Network APIs use TLS in deployed environments.
- Rate limits and request-size limits apply to externally accessible endpoints.

See [THREAT_MODEL.md](THREAT_MODEL.md).

## 7. Architecture decision record

| Decision | Current position |
|---|---|
| Backend | Rust with Axum proposed |
| Persistence | PostgreSQL proposed |
| Lightning node/SDK | Undecided; feasibility spike required |
| Receipt signing format | Must be selected and documented before implementation |
| USSD provider | Depends on accessible provider or sandbox |
| Flutter verifier | Minimal P0 interface |
| Voice IVR | P1/stretch goal |

## 8. Architecture acceptance gate

Before proceeding to broad feature development, demonstrate invoice creation and authoritative settlement observation in a repeatable test environment. Document the selected integration, test setup, event semantics, and known limitations.
