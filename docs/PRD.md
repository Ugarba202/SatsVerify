# SatsVerify — Product Requirements Document v3

**Working name:** SatsVerify
**Event:** Bitcoin Buildathon 2026 — GDG DevFest Kaduna × Btrust
**Build day:** October 29, 2026
**Showcase:** October 31, 2026
**Track:** Greenfield Build
**Project type:** Open-source Bitcoin/Lightning infrastructure
**Status:** Proposed implementation scope following prior-art research
**Schedule:** October 11–29, 2026; 18 calendar days

## 1. Product Vision

Build an open-source, wallet-agnostic Lightning payment verification service that allows merchants and applications to check whether a specific Lightning invoice actually settled, using an HTTP API, a basic-phone interface, or a lightweight verifier application.

The system will combine a trusted payment observer with independently verifiable cryptographic evidence and reusable phone-channel infrastructure.

**Product promise:** Verify the payment, not the screenshot.

## 2. Problem Statement

Lightning wallets and merchant integrations can expose payment status, but developers building their own payment experiences must still implement invoice tracking, event processing, duplicate-request handling, failure states, merchant-facing confirmation, and channel-specific interfaces.

For merchants with limited access to smartphones or mobile data, the verification experience may be inaccessible or unnecessarily complicated.

Our research also identified direct competitors in feature-phone Lightning access, particularly Sabi Wallet and Machankura. Consequently, another consumer wallet would offer limited differentiation.

The project will investigate and address a narrower engineering problem:

**Can we provide reusable, well-tested infrastructure that lets an application or merchant query a Lightning payment's actual settlement state, obtain a receipt with clearly defined evidence, and access the result through low-bandwidth or basic-phone interfaces?**

The project will not claim that Lightning itself lacks settlement finality or that no existing verification API exists. The novelty claim must remain limited to the specific integration, evidence, and channel capabilities that our implementation and further prior-art research establish.

## 3. Target Users

### Primary users

- Small merchants accepting Lightning payments.
- Freelancers and service providers receiving Lightning invoices.
- Developers building Lightning-powered merchant applications.
- Operators who need reliable payment status without trusting screenshots or customer messages.

### Secondary users

- Existing Lightning wallets that want reusable payment-verification infrastructure.
- Developers implementing USSD or voice-based Bitcoin experiences.
- Auditors and technical reviewers who need to inspect payment evidence.

## 4. Product Principles

1. **Truthful status:** Only the configured authoritative payment observer can establish settlement for invoices it monitors.
2. **Evidence transparency:** Every receipt states what was observed, who issued the receipt, and what can be verified independently.
3. **No customer custody:** The service does not operate as a general-purpose consumer wallet or hold customer wallet balances.
4. **Channel independence:** The verification engine is independent of USSD, voice, Flutter, and HTTP.
5. **Idempotency:** Repeated requests must not cause unintended duplicate invoices or side effects.
6. **Minimal phone interaction:** The USSD flow must minimize the number and duration of paid sessions.
7. **Open-source reuse:** Core verification logic, protocol documentation, and testing tools must be available for other developers to use.
8. **Honest limitations:** The product must distinguish a cryptographically valid hash relationship from evidence that a real payment settled.

## 5. Scope

### P0 — Required for the showcase

- Lightning invoice creation through a supported node integration.
- Invoice registry and settlement-state machine.
- Authoritative settlement-event processing.
- Idempotent invoice creation and safe event processing.
- Merchant-facing payment-status API.
- Signed settlement receipts with a documented trust model.
- Independent CLI verification of receipt signatures and payment-hash relationships.
- USSD status-checking flow using a supported sandbox or demonstrable test setup.
- Basic Flutter verification interface, with local verification where applicable.
- Automated tests for pending, settled, expired, invalid, and duplicate cases.
- Threat model, architecture documentation, and reproducible demo instructions.

### P1 — If P0 is stable

- Deterministic voice IVR with DTMF input.
- Signed webhooks for integrators.
- A merchant-oriented receipt history.
- Hausa prompts.
- Additional node/provider adapters.
- QR or printable receipts.

### Explicit non-goals

- A consumer Lightning wallet.
- General-purpose balances, user private-key management, or customer custody.
- Naira-to-Bitcoin exchanges or swap execution.
- A general P2P marketplace.
- A new Lightning protocol implementation.
- Unrestricted speech recognition.
- A claim that every Lightning payment on the network can be verified by this service without access to an authoritative payment source.

## 6. Functional Requirements

### F1. Invoice issuance and observation

A registered integration or merchant requests an invoice for a specified amount.

The service:

- Creates the invoice through the configured Lightning node.
- Records the invoice reference, payment hash, expected amount, creation time, and expiry.
- Associates the invoice with the merchant or integration that requested it.
- Subscribes to or polls the authoritative payment source using a documented integration.
- Processes events idempotently.
- Maintains an auditable record of state changes.

The MVP state model is:

`OPEN → IN_FLIGHT → SETTLED`

Alternative terminal states include `FAILED` and `EXPIRED`, with transitions defined according to the chosen node's actual event semantics.

The implementation must not infer that a payment settled merely because a payer claims to have paid.

### F2. Payment-status API

Provide an authenticated or appropriately access-controlled endpoint that returns a documented status for a known invoice.

A response should include:

- Invoice reference.
- Merchant or integration scope.
- Expected amount and currency unit.
- Current payment state.
- Settlement timestamp when established by the payment source.
- Evidence type and receipt reference.
- Receipt signature or verification instructions where available.

The API must not expose one merchant's invoice details to an unrelated merchant.

Repeated reads must be safe. Mutating operations must use idempotency keys.

### F3. Settlement receipt and attestation

When the authoritative observer establishes settlement, the service produces a structured receipt.

The payload may contain:

- Protocol version.
- Invoice reference and payment hash.
- Amount and unit.
- Settlement timestamp and its source.
- Issuing observer identity.
- Merchant or integration identifier.
- Unique receipt identifier.
- Evidence and signature metadata.

The service signs the canonicalized payload using an asymmetric signing key. The public verification key and signature format are documented.

The signature establishes that the identified issuer signed the receipt. It does not, by itself, prove that the issuer observed a genuine settlement.

Where the integration exposes the preimage, the receipt may include it subject to the project's disclosure policy. The verifier checks:

`SHA256(preimage) == payment_hash`

A successful hash check establishes the preimage relationship. Establishing that the invoice settled for the claimed amount still depends on trustworthy observation and the invoice's authenticated terms.

The service must not publish the preimage of an invoice issued by another system unless it is authorized to disclose it. Access to sensitive payment data must be restricted.

### F4. Independent verifier CLI

Provide a small open-source command-line tool that can:

- Validate receipt structure and protocol version.
- Verify the issuer's signature against a trusted public key.
- Check the preimage-to-payment-hash relationship when a preimage is provided.
- Validate that the receipt has not been altered.
- Report which checks passed and which remain unproven.
- Clearly distinguish cryptographic consistency from independently established settlement.

Example output:

`Receipt signature: VALID`

`Preimage/hash relationship: VALID`

`Settlement observation: ASSERTED BY TRUSTED ISSUER`

`Conclusion: RECEIPT VALID UNDER THE CONFIGURED TRUST MODEL`

A verifier without a trusted issuer key or authenticated settlement source must not claim universal proof of payment.

### F5. USSD adapter

Provide a phone interface through a compatible USSD gateway.

Initial menu:

1. Create a test sale.
2. Check an existing invoice.
3. Retrieve the latest receipt reference.

Requirements:

- Keep session state on the server.
- Treat session identifiers as temporary correlation values, not proof of user identity.
- Retrieve invoice state by reference when a session resumes.
- Never re-execute a mutating action solely because a session restarts.
- Apply rate limits and access controls.
- Avoid revealing sensitive payment data to an unauthenticated caller.
- Minimize menu steps and document the measured session count and cost assumptions.

For the showcase, the target is a maximum of three USSD sessions per completed sale where the provider and flow permit it. This is a design target, not a guaranteed telecom billing outcome.

The service must not assume that a full Lightning invoice can fit in one ordinary SMS. Use a supported delivery mechanism or shareable payment reference, and document any demo-specific limitations.

### F6. Voice adapter

If time permits, add a deterministic IVR interface using recorded or synthesized prompts and DTMF inputs.

The voice adapter uses the same verification API as USSD. It must not implement a separate source of truth.

The flow supports status lookup, receipt reference playback, and repeat prompts. Open-ended speech recognition is out of scope.

### F7. Flutter verifier

Provide a minimal Flutter application for smartphone users and judges.

Initial screens:

- Enter or scan a supported invoice or receipt reference.
- View payment state.
- Inspect the receipt and issuer.
- Check the receipt signature using a trusted public key.
- Check the preimage/hash relationship when the required data is available.
- Explain which checks require an online source and which can run offline.

Offline verification is limited to checks possible from the data and trust anchors already stored on the device. An offline app cannot independently discover a new settlement or establish that a receipt issuer is honest.

### F8. Audit and event handling

Maintain an append-only application event log for:

- Invoice creation.
- Observed payment-state changes.
- Receipt issuance.
- Verification requests.
- Authorization failures and rate-limit events.

Do not log PINs, private keys, access tokens, or preimages in ordinary application logs.

For an 18-day MVP, an append-only application log with restricted write access and integrity checks is acceptable. Do not claim tamper-proof storage unless a suitable integrity mechanism has actually been implemented and tested.

## 7. Security and Trust Model

### Threats

- Forged screenshots and fabricated payment messages.
- Fake or altered receipts.
- Duplicate API calls and repeated USSD commands.
- Replayed or stale responses.
- Unauthorized access to merchant invoices.
- Incorrect mapping between a receipt and an invoice.
- Compromised payment-observer credentials.
- Malicious or incorrect settlement events.
- Disclosure of payment preimages or merchant data.
- Denial of service against verification endpoints.

### Required controls

- Asymmetric receipt signatures with documented key management.
- Authenticated merchant and integration access.
- Per-merchant invoice isolation.
- Idempotency for mutating requests.
- Replay-safe event processing.
- Rate limiting and bounded request sizes.
- TLS for network transport.
- Secret management outside source code.
- Minimal retention of sensitive information.
- Tests for invalid signatures, altered receipts, invalid preimages, stale events, expired invoices, and duplicate requests.
- A documented process for rotating or revoking signing keys.

A nonce may help correlate requests and responses, but a nonce alone does not establish freshness or prevent replay. Verification clients must apply an explicit timestamp, expiry, or one-time-use policy appropriate to the operation.

### Custody statement

The service does not manage customer wallet balances or private keys. It may operate a Lightning node for invoices it issues as part of the merchant integration. The team must document who controls that node's funds, how test funds are handled, and what operational and legal obligations apply.

The absence of customer custody does not automatically establish a regulatory exemption. Obtain appropriate legal guidance before operating a production service in Nigeria.

## 8. High-Level Architecture

**Channel adapters**

- USSD gateway adapter.
- Optional deterministic IVR adapter.
- REST clients and webhooks.
- Flutter verifier.

**Verification service**

- Rust and Axum API.
- Invoice registry and state machine.
- Merchant/integration authorization.
- Idempotency and event-deduplication middleware.
- Receipt generation and asymmetric signing.
- Public-key-based receipt verification.
- Audit event processing.

**Persistence**

- PostgreSQL for invoice metadata, state transitions, merchant scope, idempotency records, and receipt metadata.
- Protected signing-key storage.
- Restricted audit-log access.

**Lightning integration**

- A clearly defined adapter for the selected node or SDK.
- Authoritative invoice creation and settlement-event observation.
- A documented test configuration and repeatable regtest environment.

The team should choose one Lightning implementation for the MVP after a short integration spike. The adapter boundary should permit future implementations without delaying the core.

## 9. Acceptance Criteria

1. A merchant creates an invoice through the configured Lightning integration.
2. An external supported Lightning wallet pays the invoice.
3. The service observes settlement and updates the correct invoice.
4. A status query returns the correct result without trusting payer-supplied screenshots or messages.
5. A forged receipt or altered payload fails signature verification.
6. A wrong preimage fails the hash check.
7. A valid receipt passes its defined verification checks.
8. The CLI distinguishes receipt authenticity from independently proven settlement.
9. Expired or unpaid invoices are never reported as settled.
10. Duplicate mutating requests do not create unintended duplicate invoices.
11. An invoice belonging to one merchant cannot be read by an unauthorized merchant.
12. The USSD demonstration can retrieve the status of an existing invoice through the configured test environment.
13. The Flutter verifier can perform its supported offline checks using previously obtained receipt data and trusted keys.
14. The repository contains automated tests, setup instructions, the threat model, and a repeatable demo.
15. The end-to-end demonstration can be completed in under three minutes.

## 10. Three-Minute Demonstration

**0:00–0:30 — Problem**

Show a fabricated payment screenshot. Explain that a screenshot is not evidence of settlement.

**0:30–1:15 — Real payment**

Create an invoice through the merchant interface, pay it using an external Lightning wallet, and show the authoritative observer registering settlement.

**1:15–1:50 — Phone-channel verification**

Use the supported USSD test interface to retrieve the payment status. Show that the fabricated screenshot alone cannot change the invoice state.

**1:50–2:30 — Independent verification**

Run the CLI against the issued receipt. Demonstrate signature verification and, where available, the preimage/hash check. Explain precisely what each result establishes.

**2:30–3:00 — Open-source infrastructure**

Show the architecture, test results, and adapter boundary. Explain how another wallet or merchant application could integrate the verification service.

A testnet or regtest demonstration must be labelled accurately. Do not present simulated settlement as a real production payment.

## 11. Eighteen-Day Execution Plan

### Phase 1 — Foundation and feasibility: October 11–14

- Inspect existing merchant-verification projects and the Sabi Wallet repository.
- Confirm the narrow differentiation and identify potential integration partners.
- Select one Lightning implementation.
- Prove invoice creation and settlement-event observation.
- Define the receipt format and threat model.
- Create the repository, CI, and regtest environment.

**Exit criterion:** A reproducible test demonstrates an invoice moving from unpaid to settled based on the selected payment source.

### Phase 2 — Verification engine: October 15–19

- Implement invoice metadata and state transitions.
- Add idempotency and event deduplication.
- Implement authenticated status queries.
- Implement asymmetric receipt signing.
- Build the CLI verifier.
- Test expired, invalid, duplicated, and altered events or receipts.

**Exit criterion:** The core verification engine and CLI pass the critical test suite.

### Phase 3 — Phone access and integration: October 20–24

- Attend the relevant Bitcoin and Lightning masterclasses.
- Implement the USSD adapter against the available provider environment.
- Complete the merchant status lookup.
- Validate session-expiry behavior and access controls.
- Add the minimal Flutter verifier.
- Attempt the voice integration only after the core and USSD path are stable.

**Exit criterion:** The phone interface can retrieve the same authoritative status as the API.

### Phase 4 — Hardening and showcase preparation: October 25–28

- Test external-wallet payment flow.
- Test stale, duplicated, expired, and invalid states.
- Review signing-key handling and data disclosure.
- Complete README, architecture, threat model, and setup instructions.
- Rehearse the demo at least five times.
- Prepare a labelled fallback recording if the external network or provider fails.

**Exit criterion:** The team can demonstrate the complete flow and explain the trust model without making unsupported claims.

### October 29 — Build day

- Integrate the final components.
- Fix critical defects.
- Freeze feature scope.
- Run the full test suite.
- Package and rehearse the final demo.

### October 31 — Showcase

Demonstrate the verified end-to-end flow, independent checks, and reusable open-source architecture.

## 12. Risks and Mitigations

| Risk                                                     | Mitigation                                                                                         |
| -------------------------------------------------------- | -------------------------------------------------------------------------------------------------- |
| Existing products already expose payment status          | Verify the exact gap against existing APIs, POS tools, and repositories before claiming novelty.   |
| The node integration takes too long                      | Select one integration early and make the settlement-event spike the first milestone.              |
| The signature is mistaken for proof of settlement        | Separate issuer authenticity from settlement observation in the receipt format and CLI.            |
| USSD provider access is delayed                          | Build against a sandbox or local adapter and label the demonstration environment accurately.       |
| Voice integration consumes the schedule                  | Keep voice as a stretch goal until P0 passes.                                                      |
| Preimage disclosure creates privacy or operational risks | Restrict disclosure, document invoice ownership, and avoid publishing sensitive values by default. |
| A demo payment fails due to network conditions           | Maintain a repeatable regtest fallback and clearly distinguish it from the live path.              |
| Legal obligations are misunderstood                      | Avoid definitive exemption claims and seek qualified guidance before production deployment.        |

## 13. Prior-Art Validation

Before submission, compare the implementation with:

- Sabi Wallet's current source code and integration model.
- Machankura's phone-channel payment and status flows.
- BTCPay Server and other Lightning merchant integrations.
- Existing node invoice-status APIs and event streams.
- Existing payment webhook and receipt-signing approaches.
- Existing open-source USSD, IVR, and payment-orchestration components.

Document what exists, what is missing, what we actually implement, and why an independent developer would reuse it.

The project should not claim that no settlement-verification API exists. Its contribution must be demonstrated through the combination of defined evidence semantics, independent receipt verification, robust event handling, and a reusable phone-channel adapter—or narrowed further if prior art reveals that this combination already exists.

## 14. Definition of Done

The project is ready for judging when it has:

- A working payment-observation and verification service.
- A signed receipt format with explicit trust assumptions.
- A CLI that independently checks the supported cryptographic properties.
- A working phone-channel demonstration.
- Automated tests for critical failure and security cases.
- A documented integration boundary for future wallets and merchant systems.
- An open-source repository with setup instructions, known limitations, and contribution guidance.
- A clear explanation of the exact gap addressed and how the project complements existing Bitcoin infrastructure.

## Product Principle

**Make payment verification simple for merchants and rigorous for developers.**

We are not building another wallet. We are building a reusable infrastructure component that helps applications report Lightning payment outcomes accurately, makes evidence inspectable, and extends access to users who cannot rely on a conventional smartphone interface.
