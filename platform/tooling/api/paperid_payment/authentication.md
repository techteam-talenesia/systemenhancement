# Authentication & Environments

Source: https://open-api-paper-id.readme.io/reference/get-your-api-keys, https://open-api-paper-id.readme.io/reference/the-staging-environment, https://open-api-paper-id.readme.io/reference/your-first-api-call-create-invoice

## Auth model

Paper.id's Open API uses a **static API key pair**, not OAuth and not per-request signing:

- `client_id` — sent as an HTTP header on every request
- `client_secret` — sent as an HTTP header on every request

Both are required headers on essentially every endpoint documented (no bearer token exchange, no OAuth flow, no token refresh step was found anywhere in the docs). Example from the "first API call" walkthrough:

```
POST {{BASE_URL}}/api/v1/store-invoice
client_id: <your client id>
client_secret: <your client secret>
Content-Type: application/json
```

**Gap:** the docs do not show a signed-request / HMAC-per-call scheme for the REST API. (Webhooks/callbacks are a separate story — see `webhooks.md` — and notably do **not** document any signature verification either, which is a real security gap to flag in the brief.)

## Obtaining credentials

1. Log into the Paper.id dashboard with an account that has the **`owner`** role (required — other roles cannot reach this page).
2. Open Settings (cog icon, top right) → **API Integration**.
3. Click **Generate API Key** to get the `client_id` / `client_secret` pair.
4. Optionally add the integrating server's IP to an **IP allow-list** for extra security.

Credentials are generated separately for the staging dashboard and the production dashboard — the docs describe getting a "test account for integration purposes" in staging first, then a "real account" in production once integration testing is complete.

## Environments

| Environment | Notes |
|---|---|
| **Staging** | Base URL seen in examples: `https://open-api.stag-v2.paper.id`. Architecturally "1:1 compared with production, with lower server specification." No KYC/KYB or company-document verification required to get a staging account. **Never attempt a real payment in staging** — instead use the payment simulation endpoint (`payment-simulation-staging-only`, see `payments.md`) to force a request to `PAID`. Callback **retries are not available** in staging — if a callback fails to deliver, you must create a new invoice/payment to test again. Staging is described as being used heavily for feature testing and "may not always operate smoothly or consistently." |
| **Production** | Requires full KYC/KYB and company documentation before go-live. Base URL pattern implied to be the production equivalent of the staging host (exact production hostname not shown in the fetched pages — confirm with Paper.id account manager before building the integration). |

## Practical implication for Talenesia

Because auth is a simple static header pair (no OAuth dance, no token expiry to manage), a backend integration is low-friction to build: store `client_id`/`client_secret` as secrets in whatever service talks to Paper.id (e.g., a small integration service or serverless function), and call the REST API directly. The bigger design question is around **webhook authenticity** (see `webhooks.md`), since no signing secret is documented for verifying that an inbound callback really came from Paper.id.
