---
name: Create and track a Loopay payout
description: Authenticate, resolve reference data, create a self pay-out, and poll its status using the Loopay API.
api: openapi/loopay-openapi-original.json
operations: [User.login, Bank.bankBases, Currency.currencies, DocumentType.documentTypeBases, Payout.createSelfPayOut, Payout.selfPayOut]
---

# Create and track a Loopay payout

Use this to disburse a single payment (self pay-out) through Loopay.

## Auth
1. Mint a bearer token with `User.login` (`POST /login`). Send it as a Bearer token on every subsequent request. Loopay's OpenAPI omits a machine-readable `securityScheme`; auth is token-based per the integration guide.

## Resolve reference data
2. `Bank.bankBases` (`GET /banks`) — resolve the destination `bankBaseId`.
3. `Currency.currencies` (`GET /currencies`) — resolve `currencyId`.
4. `DocumentType.documentTypeBases` (`GET /documentTypes`) — resolve the beneficiary `documentTypeId`.

## Create the payout
5. `Payout.createSelfPayOut` (`POST /payout/create`) with a `CreateSelfPayOutInput` body: `companyProductId`, `companyId`, `bankBaseId`, `currencyId`, `documentTypeId`, amount, and beneficiary fields. Set a caller-owned `externalIdentifier` so you can reconcile later — Loopay has no idempotency-key header, so this reference is your dedup handle. Expect `200` or `202` (accepted for async processing).

## Track
6. Poll `Payout.selfPayOut` (`GET /payout/{id}`) with the returned id until it reaches a terminal state.

## Rules
- There is no idempotency contract; do not blindly retry `createSelfPayOut`. On timeout, look up by external identifier before re-sending (see the reconcile skill).
- Only `200`/`202` are documented; treat any non-2xx as an undocumented error and surface the raw body.
