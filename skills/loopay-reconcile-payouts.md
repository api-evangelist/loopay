---
name: Reconcile Loopay payouts and movements
description: Look up payouts by caller-assigned external identifier and read account movements to reconcile a Loopay company product.
api: openapi/loopay-openapi-original.json
operations: [User.login, Payout.getSelfPayOutsByExternalIdentifierList, Payout.selfPayOuts, Movements.selfPayOuts]
---

# Reconcile Loopay payouts and movements

Use this to reconcile disbursements against your own ledger.

## Auth
1. `User.login` (`POST /login`) → Bearer token on all calls.

## Reconcile by your reference
2. `Payout.getSelfPayOutsByExternalIdentifierList` (`POST /payouts/byExternalIdentifierList`) with a `SelfPayOutsByExternalIdentifierInput` list of the `externalIdentifier`s you assigned at creation. This is the safe way to confirm whether a payout landed after a create timeout — there is no idempotency key, so the external identifier is the dedup handle.

## List and window
3. `Payout.selfPayOuts` (`GET /payouts`) — list payouts; page with `page` + `recordsPerPage`, filter with `startDate` + `endDate`.
4. `Movements.selfPayOuts` (`GET /movements/{companyProductId}`) — read ledger movements for a company product to match against your books.

## Rules
- Pagination is page-number based (`page`, `recordsPerPage`); iterate until fewer than `recordsPerPage` rows return.
- Always reconcile by external identifier before re-issuing a payout you are unsure about.
