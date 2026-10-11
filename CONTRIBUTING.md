# Contributing to SatsVerify

Thank you for considering a contribution.

SatsVerify is an experimental open-source project for wallet-agnostic Lightning payment verification. Contributions should preserve the documented trust model and must not imply that a signed statement alone proves payment settlement.

## Before starting

1. Read [README.md](README.md) and [docs/PRD.md](docs/PRD.md).
2. Check existing issues and pull requests before starting overlapping work.
3. For substantial changes, open an issue to agree on the problem and acceptance criteria.
4. Do not expand the product scope without discussing the trade-off.

## Workflow

1. Start from an up-to-date `main` branch.
2. Create a focused branch, for example `feat/receipt-verifier` or `docs/threat-model`.
3. Keep each change small enough to review.
4. Add or update tests for behavior changes.
5. Run the relevant tests and checks before opening a pull request.
6. Open a PR describing the problem, approach, testing performed, security implications, and known limitations.

Do not commit secrets, access tokens, private keys, payment preimages, or real customer data.

## Commit messages

Use concise messages that describe the change, such as:

- `docs: add architecture overview`
- `feat: add invoice state model`
- `test: cover duplicate settlement events`
- `fix: reject expired receipts`

## Engineering expectations

- Preserve existing behavior unless the change intentionally updates it.
- Validate external input and return explicit errors.
- Keep channel adapters separate from payment-state authority.
- Process repeated requests and events safely.
- Document trust assumptions and limitations.
- Never mark an invoice settled from a payer-supplied screenshot or claim.
- Do not claim offline verification establishes new settlement information.

## Pull requests

A PR should include:

- A clear problem statement and proposed change.
- Links to related issues or requirements.
- Tests and commands run, with results.
- Documentation updates when behavior or architecture changes.
- Security, privacy, migration, or compatibility considerations.

Maintainers may request changes or defer work that conflicts with the PRD, security model, or buildathon priorities.

## License

By contributing, you agree that your contributions are provided under the project's MIT License.
