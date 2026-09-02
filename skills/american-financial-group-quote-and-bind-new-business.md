---
name: quote-and-bind-new-business
description: Take a commercial risk from appetite check through pricing, submission and bind on the Great American Carrier Services APIs.
api: Great American Carrier Services (Shop, Risk Selection, Rating, Product, Submission)
operations:
  - POST /api/newBusiness/appetite
  - POST /api/checkEligibility
  - POST /api/products
  - POST /api/newBusiness/pricing
  - POST /api/pricingIndication
  - POST /api/rate
  - POST /api/quote
  - POST /api/newBusiness/application
  - POST /api/create
  - POST /api/bind
generated: '2026-09-02'
method: generated
source: https://api-documentation.gaig.com/{shop,risk-selection,rating,product,submission}/index.html
---

# Quote and bind new business

## Before anything

1. Get a token. `POST https://prod01.api.gaig.com/oauth/token` with
   `Authorization: Basic base64(clientId:clientSecret)`,
   `Content-Type: application/x-www-form-urlencoded`, body `grant_type=client_credentials`.
   The response carries `access_token`, `token_type` (Bearer) and `expires_in` (3599). Also read
   `api_product_list_json` — it names the API products you are entitled to, and it encodes the
   environment (`issuance-dev`, `rating-dev`).
2. `GET /api/endpoints` on each API you intend to use. It returns only the endpoints THIS consumer may
   call. Anything not in that list answers `501 Not Implemented`, and a 501 here means "not for you",
   not "broken".
3. `POST /api/docs` with `{"type":"request","endpoint":"/api/…","format":"json","scope":{"state":"OH"}}`
   for every endpoint you are about to call. **Do not assume a body shape.** The OpenAPI documents carry
   no schemas; the bodies vary by consumer, business division, product and state, and `/api/docs` is the
   only source of truth for them.

## The flow

1. **Appetite.** `POST /api/newBusiness/appetite` (Shop) or `POST /api/checkEligibility` (Risk Selection)
   to learn whether Great American writes this risk at all. Risk Selection is a stateful questionnaire —
   `POST /api/createAnswerSession`, then `POST /api/updateAnswerSession` as answers come in, then
   `POST /api/checkEligibility`. Stop here on a negative outcome; everything downstream is wasted.
2. **Product.** `POST /api/products`, `POST /api/classCodes` and `POST /api/availability` (Product) to
   resolve the product, class code and whether it is available in the state.
3. **Price.** `POST /api/pricingIndication` (Shop) for an indication, or `POST /api/rate` / `POST /api/quote`
   (Rating) for a full quote from locations, coverages, limits and deductibles. `POST /api/taxes` returns
   surplus lines and premium tax. All of this is read-only — nothing has been created yet.
4. **Application.** `POST /api/newBusiness/application` (Shop) to assemble the application, then
   `POST /api/newBusiness/documents` for the document set.
5. **Submit.** `POST /api/create` (Submission) to place the submission in the policy administration
   system. `POST /api/read` returns it; `POST /api/update`, `POST /api/assign` and `POST /api/setStatus`
   work it; `POST /api/search` finds it later.
6. **Bind.** `POST /api/bind` (Submission).

## Rules that matter

- **Bind is irreversible on this surface.** No unbind, void or rescind operation is published. Confirm
  the quote and the application with the human before calling `/api/bind`. The reversal path only appears
  after issuance, as `POST /api/cancel` on the Issuance API.
- **There is no idempotency key.** Nothing on this API accepts an `Idempotency-Key` or client reference,
  so a retried `POST /api/create` or `POST /api/bind` may create a duplicate. Do not retry a write blindly
  on a timeout — re-read with `POST /api/read` or `POST /api/search` first.
- **There is no documented rate limit and no 429** in any published status table. Pace yourself anyway;
  the gateway may enforce limits that are not published.
- **Errors.** `400` returns `{"errors":[{"category","code","message"}]}` — branch on `code`. `401` returns
  the Apigee `{"fault":{…}}` envelope, meaning your token expired: fetch a new one. `403` means your
  entitlement does not cover it. `500`/`501` return `{"timestamp","status","error","message","path"}`.
- **Shop orchestrates the rest.** If the same flow is available on Shop, prefer it — it is the workflow
  API that calls Submission, Rating, Product and Risk Selection for you. Note that the Shop reference
  document is the oldest of the 18 (last updated 2023-01-09), so verify shapes with `/api/docs`.
