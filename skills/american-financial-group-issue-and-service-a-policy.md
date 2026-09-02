---
name: issue-and-service-a-policy
description: Issue a policy or master-policy enrollment on Great American Carrier Services, then read, document, bill and cancel it.
api: Great American Carrier Services (Issuance, Policy, Forms, Billing, Document)
operations:
  - GET /api/customerNumber
  - GET /api/policyNumber
  - POST /api/issue
  - POST /api/enroll
  - POST /api/policyMod
  - POST /api/policy
  - POST /api/transactions
  - POST /api/certificate
  - POST /api/cancelDates
  - POST /api/cancel
  - POST /api/attach
  - POST /api/generatePDF
  - POST /api/createNewBillingAccount
  - POST /api/createPayments
generated: '2026-09-02'
method: generated
source: https://api-documentation.gaig.com/{issuance,policy,forms,billing,document}/index.html
---

# Issue and service a policy

Authenticate and discover exactly as in `quote-and-bind-new-business`: token from
`POST /oauth/token` (client_credentials), then `GET /api/endpoints` and `POST /api/docs` per API.

## Issue

1. **Identifiers.** `GET /api/customerNumber` and `GET /api/policyNumber` (Issuance) mint the numbers
   Great American expects. Do not invent them.
2. **Issue.** `POST /api/issue` (Issuance) for a straight-through policy transaction — supply all the data
   the transaction requires, per `/api/docs`. `POST /api/policyControlDate` sets the control date.
   `POST /api/emision` and `POST /api/impresion` are the Spanish-language equivalents of issue and print.
3. **Or enroll.** For a master policy: `POST /api/verifyMasterPolicy` (Policy) to confirm the master
   exists and is valid, then `POST /api/enroll` (Issuance). `POST /api/requestMasterPolicy` (Policy) asks
   underwriting for a new master policy type and generates an internal email — it is a request, not a
   provisioning call, so do not wait on it synchronously.

## Service

- **Read.** `POST /api/policy`, `POST /api/quote`, `POST /api/enrollment`, `POST /api/transactions` (Policy).
- **Evidence of coverage.** `POST /api/certificate` (Policy).
- **Forms.** `POST /api/attach` (Forms) with the `gaig-formdatatype` header —
  `CompleteFormList`, `SelectableFormList`, `PrintOrders` or `PrintOrdersOrderedByPrintTier`. Then
  `POST /api/generatePDF`. Forms accepts and returns XML as well as JSON; set both `Content-Type` and
  `Accept`.
- **Change.** `POST /api/policyMod` (Issuance) returns the policy mod and version to use on a subsequent
  transaction — call it BEFORE the change transaction, not after.
  `POST /api/enrollmentChange` and `POST /api/enrollmentCoverageChange` amend an enrollment.
- **Bill.** `POST /api/createNewBillingAccount`, `POST /api/getPaymentPlans`, `POST /api/billCharges` and
  `POST /api/createPayments` (Billing). `POST /api/getPoliciesForAccount` links account to policies.
- **Document.** Store artifacts through the Document API (`POST /api/*/documents`) or push them with
  `POST /api/ingest` (Ingestion) using the `gaig-data-type` header.

## Reversal — read this before you write

- **Cancel exists; the window is not published.** `POST /api/cancel` (Issuance) cancels an in-force policy
  or an existing enrollment. No document states a deadline for it. What IS available is
  `POST /api/cancelDates` (Policy) — "get valid cancellation date information" — so **call `/api/cancelDates`
  to learn the permitted dates for THIS policy rather than assuming any window.**
- **Payments have no published reversal.** `POST /api/createPayments` has no refund, void or reverse
  counterpart in the published surface. Treat it as one-way and confirm with the human first.
- **No idempotency key exists.** A retried `POST /api/issue` or `POST /api/createPayments` can duplicate.
  On a timeout, read back with `POST /api/policy` or `POST /api/getAccountDetails` before retrying.
