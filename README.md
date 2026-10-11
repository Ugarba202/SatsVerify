# SatsVerify

**Verify the payment, not the screenshot.**

SatsVerify is a proposed open-source, wallet-agnostic Lightning payment-verification service. It aims to help merchants and applications query the settlement state of invoices monitored by a configured authoritative payment source, inspect signed receipts, and access verification through an API and a basic-phone interface.

> **Project status:** Early development. The product specification is documented, but the implementation and integrations are not yet complete. Do not use this project to confirm production payments.

## The problem

Applications integrating Lightning payments must handle invoice tracking, settlement events, duplicate requests, payment states, and reliable merchant confirmation. Merchants with limited access to smartphones or mobile data may also need a simpler verification interface.

SatsVerify explores reusable infrastructure for these needs rather than building another consumer wallet.

## Planned capabilities

- Lightning invoice creation and authoritative settlement observation.
- Merchant-scoped payment-status API.
- Explicit invoice state transitions and idempotent operations.
- Asymmetrically signed settlement receipts with documented trust assumptions.
- Independent CLI checks for receipt signatures and preimage/hash relationships.
- USSD status lookup through a compatible gateway or clearly labelled test environment.
- Minimal Flutter receipt-verification interface.
- Automated tests, a threat model, and reproducible demo instructions.

These are planned capabilities, not claims that every feature already works.

## Trust model

SatsVerify distinguishes between:

1. **Issuer authenticity:** a valid signature shows that the holder of a signing key signed a receipt.
2. **Preimage/hash consistency:** where supplied, the verifier checks `SHA256(preimage) == payment_hash`.
3. **Settlement observation:** the configured authoritative payment source reports settlement for an invoice it monitors.

A valid signature or hash relationship alone does not prove that a payment settled for the claimed amount. The verifier must report what it checked and what remains dependent on trust in the configured observer.

## Planned technology

- **Backend:** Rust and Axum.
- **Persistence:** PostgreSQL.
- **Phone interface:** USSD adapter.
- **Verifier app:** Flutter.
- **Lightning integration:** to be selected after a feasibility spike.

The Lightning node or SDK is intentionally undecided until invoice creation and settlement observation have been demonstrated in a repeatable test environment.

## Repository guide

- [Product requirements](docs/PRD.md)
- [Architecture](docs/ARCHITECTURE.md)
- [Threat model](docs/THREAT_MODEL.md)
- [Roadmap](docs/ROADMAP.md)
- [Contributing](CONTRIBUTING.md)
- [Security reporting](SECURITY.md)

## Development status

The immediate engineering milestone is to prove that the selected Lightning integration can create an invoice and observe its settlement reliably. Broader API and channel work depends on that result.

Setup instructions will be added after the initial integration and development environment have been selected and tested.

## Scope boundaries

SatsVerify is not intended to be a consumer wallet, a customer-balance service, a swap platform, or a new Lightning protocol implementation. It cannot independently verify arbitrary network payments without access to an authoritative source for the relevant invoice.

## Open source

SatsVerify is distributed under the [MIT License](LICENSE).

Contributions are welcome under the process described in [CONTRIBUTING.md](CONTRIBUTING.md). This project is experimental and is not ready for production payment decisions.
