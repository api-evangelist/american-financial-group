---
name: report-a-claim
description: File and manage a first notice of loss on the Great American Carrier Services Claims API, with notification preferences and service providers.
api: Great American Carrier Services (Claims, Notification, Policy)
operations:
  - POST /api/fnol/create
  - PATCH /api/fnol/update
  - GET /api/fnol/submission
  - PUT /api/fnol/submit
  - POST /api/riskReport
  - POST /api/reportPayment
  - POST /api/paymentFeedback
  - POST /api/serviceProvider/create
  - POST /api/serviceProvider/search
  - POST /api/claims/optIn
  - POST /api/claims/optOut
  - POST /api/claims/messages
generated: '2026-09-02'
method: generated
source: https://api-documentation.gaig.com/{claims,notification,policy}/index.html
---

# Report a claim

Authenticate with `POST /oauth/token` (client_credentials), then `GET /api/endpoints` and `POST /api/docs`
before composing any body. The Claims reference page publishes no Environment Base URL table — only the
shared `{env}.api.gaig.com/oauth/token` endpoints — so **confirm the Claims base URL with Great American
API support rather than assuming it follows the `/claims` pattern of its siblings.**

## First notice of loss

1. **Confirm the policy.** `POST /api/policy` or `POST /api/verifyMasterPolicy` (Policy) so the loss is
   attached to a real, in-force contract.
2. **Open.** `POST /api/fnol/create` (Claims).
3. **Amend.** `PATCH /api/fnol/update` while the notice is still a draft. `GET /api/fnol/submission`
   reads back the current state.
4. **Submit.** `PUT /api/fnol/submit`. **This is the commit point and no withdraw operation is published** —
   there is no cancel, retract or void for a submitted FNOL. Confirm with the human before submitting; use
   `PATCH /api/fnol/update` for corrections, which is the only correction path Great American publishes.

## Around the claim

- **Risk report.** `POST /api/riskReport` submits a claims risk report.
- **Payments.** `POST /api/reportPayment` reports a claim payment; `POST /api/paymentFeedback` returns
  feedback on it.
- **Service providers.** `POST /api/serviceProvider/search` before `POST /api/serviceProvider/create` —
  create only when the search misses, since there is no delete operation for a service provider.
  `POST /api/serviceProvider/update` and the `/api/serviceProvider/address/{create,search,update}` family
  maintain them.
- **Notifications.** `POST /api/claims/optIn` and `POST /api/claims/optOut` (Notification) manage the
  claimant's notification preference — optOut is the documented reversal of optIn, and
  `POST /api/claims/reOpen` reverses `POST /api/claims/close`. `POST /api/claims/messages` and
  `POST /api/claims/notes` add message and note data; `POST /api/chat/start` and
  `POST /api/chat/transcript` handle chat.

## Rules that matter

- **PII.** An FNOL carries claimant personal data. Do not log request or response bodies, and do not echo
  them back into a transcript.
- **No idempotency key.** A retried `POST /api/fnol/create` can open a duplicate claim. Read back with
  `GET /api/fnol/submission` before retrying.
- **501 means "not entitled".** Claims serves many business divisions; if an endpoint answers 501 it is
  not enabled for your consumer, and `GET /api/endpoints` is the authority on what is.
- **Errors.** 400 → `errors[{category,code,message}]`; 401 → Apigee `fault` (refresh the token);
  500/501 → `{timestamp,status,error,message,path}`.
