# SatsVerify — Threat Model

**Status:** Initial threat model. Update it as implementation and integration decisions become concrete.

## 1. Security objectives

- Report invoice status based on the configured authoritative payment source.
- Prevent unauthorized access to merchant payment records.
- Make receipt authenticity and evidence limitations understandable.
- Prevent duplicate operations and unsafe event replay.
- Protect signing keys, credentials, payment preimages, and sensitive merchant data.
- Make failures visible rather than silently reporting success.

## 2. Assets to protect

- Invoice identifiers, payment hashes, amounts, and settlement timestamps.
- Merchant and integration credentials.
- Receipt-signing private keys and trusted public-key metadata.
- Payment preimages, where collected.
- Invoice state and event history.
- Idempotency records and audit events.
- USSD session data and provider credentials.

## 3. Trust assumptions

- The configured Lightning node or provider is authoritative for the invoices it monitors, subject to its documented behavior and security.
- The receipt-signing key is controlled by the identified issuer and has not been compromised.
- Verifiers obtain trusted public keys through an authenticated distribution or configuration process.
- The application, database, host, and deployment secrets are not assumed immune to compromise.
- Telecom caller ID and USSD session identifiers alone do not establish a caller's identity.

A signed receipt is an authenticated statement by an issuer, not independent proof that the statement is truthful.

## 4. Threats and mitigations

| Threat | Initial mitigation | Required verification |
|---|---|---|
| Fabricated payment screenshot | Never use payer screenshots as a settlement event | Unpaid invoice remains unpaid |
| Forged or altered receipt | Canonical payload and asymmetric signature verification | Altered payload and invalid signature fail |
| Incorrect preimage | Verify `SHA256(preimage) == payment_hash` | Wrong preimage fails |
| False settlement event | Validate against the selected authoritative integration and its semantics | Invalid or stale events cannot settle an invoice |
| Duplicate invoice creation | Idempotency keys and stored operation results | Retried requests do not create unintended invoices |
| Duplicate event delivery | Event deduplication and safe state transitions | Replayed events have no unintended side effects |
| Cross-merchant access | Enforce merchant scope on every relevant query and mutation | Merchant A cannot read Merchant B's invoice |
| Stale receipt or replay | Explicit timestamp, expiry, and operation-specific reuse policy | Expired or disallowed reuse is rejected |
| Signing-key compromise | Restricted key access, rotation, and revocation procedure | Document and test key rotation/revocation |
| Preimage disclosure | Minimize collection and restrict access and logging | Ordinary logs contain no preimages |
| Credential leakage | Runtime secret configuration; never commit secrets | Repository and CI checks detect accidental secrets where configured |
| Denial of service | Rate limits, bounded payloads, timeouts, and resource controls | Excessive requests are constrained |
| USSD session abuse | Server-side state, authorization, rate limits, minimal disclosure | Session IDs alone cannot authorize sensitive access |

## 5. Receipt verification requirements

The verifier must report separate results for:

1. Receipt structure and protocol version.
2. Signature validity against a specifically trusted key.
3. Preimage/hash relationship, if the necessary data is supplied.
4. Freshness and expiry policy.
5. Settlement observation asserted by the identified issuer.

It must not label a receipt as universally proven payment merely because its signature is valid.

## 6. Logging and data handling

Do not write private keys, access tokens, PINs, or payment preimages to ordinary application logs. Restrict audit-log access and avoid describing logs as tamper-proof unless a tested integrity mechanism supports that claim.

Document retention periods and remove unnecessary personal or payment data from USSD sessions and application records.

## 7. Residual risks

- A compromised or dishonest authoritative observer can issue incorrect settlement statements.
- A compromised signing key can authenticate fraudulent receipts until trust is revoked.
- Offline verification cannot establish a new payment state.
- Telecom delivery, caller identity, provider availability, and session billing depend on external systems.
- Legal and operational obligations depend on the actual deployment and must be reviewed before production use.

## 8. Release gate

Before a public demonstration, test invalid signatures, altered payloads, incorrect preimages, expired invoices, duplicate requests, duplicate/stale events, unauthorized merchant access, and sensitive-data logging. Document any untested control explicitly.
