# Payments, Balance & Withdrawals

Source: https://open-api-paper-id.readme.io/reference/receive-payments, /reference/payment-api, /reference/payment-simulation-staging-only, /reference/digital-payment-balance, /reference/retrieve-digital-payment-balance, /reference/retrieve-list-of-balance-transactions, /reference/withdrawal-transaction, /reference/create-withdrawal-request, /reference/invoicing-payments

All endpoints require `client_id` / `client_secret` headers. Base URL (staging): `https://open-api.stag-v2.paper.id/api/v1`.

## Two ways to collect a payment

1. **Invoice-based** (typical for Talenesia): create a Sales Invoice (see `invoices.md`); the customer gets a payment link (`payper_url`/`payment_link`) and freely picks any supported payment method.
2. **Direct Payment Request**: merchant creates a Payment Request with the invoice data *and* a fixed payment method embedded — the customer cannot change the method. Used when you want a custom checkout UI rather than Paper.id's hosted payment page.

Either way: "Paper.id sends a payment notification to your partner for every payment made, irrespective of the payment method" — this is not optional/configurable per the docs.

## Supported payment methods (Payment API)

| Method | Channels |
|---|---|
| `bank_transfer` | BRI, Mandiri, BNI, Permata |
| `credit_card` | Visa, Mastercard, JCB |
| `ewallet` | OVO |
| `mitra_pembayaran_digital` | Tokopedia, Blibli, Shopee |
| `qris` | QRIS |

Behavioral notes from the docs: a paid Payment Request cannot be paid again; once paid, "a callback of success will be triggered to the callback URL that has been registered" (see `webhooks.md`). Exact HTTP method/path for creating a Payment Request (as opposed to a Sales Invoice) was not captured in this pass beyond the general `/payment/...` namespace used by balance/simulation endpoints — flagged as a gap.

## Payment simulation (staging only)

`POST /api/v1/payment/simulation` — forces a Payment Request to a given status (e.g. `PAID`) in staging, since real settlement is disabled there.

Body: `{ "change_status_to": "PAID", "ref_id": "REF-XXXX" }`

Use this to test the full webhook flow end-to-end before going live.

## Digital Payment Balance

Paper.id holds collected funds in a Paper.id-managed balance (via virtual accounts etc.) before you withdraw to your own bank. Two balance types:
- **Active balance** — withdrawable now
- **Holding balance** — temporarily reserved, not yet withdrawable

### Get balance
`GET /payment/balance`
```json
{ "currency": "IDR", "balance": 56005, "on_hold_amount": 0, "created_at": "...", "modified_at": "..." }
```

### List balance transactions
`GET /payment/balance/list` — query `limit`, `offset`. (Response field-level schema wasn't documented beyond 200/400 status; described purpose: "check all the transactions of digital payment transactions from customer and withdrawal balance.")

## Withdrawals (moving Paper.id balance → Talenesia's bank account)

### Create withdrawal request
`POST /withdrawal/request`

Body: `{ "transaction_detail": { "ref_id": "string", "total_amount": <int> } }`

Response:
```json
{ "data": { "ref_id": "00000001", "transaction_status": "Disbursed Requested" }, "message": "Successful withdrawal request", "status_code": 200 }
```

Withdrawals can also be triggered manually from the dashboard as an alternative to the API. Related endpoints referenced in the docs index but not fetched in depth: `get_api-v2-verification-request` (OTP verification, likely required to authorize a withdrawal or bank-account change) and `post_api-v2-withdrawal-bank-accounts` (register a destination bank account for withdrawals) — see README gaps.

## Integration relevance

- Balance + balance-transactions endpoints are the natural source for **automated bank reconciliation** — pull them on a schedule (or after each payment webhook) and write into the accounting system instead of manually reading the Paper.id dashboard.
- The withdrawal API means even the "move settled funds to our real bank account" step, which is presumably done manually today, could be automated or at least monitored programmatically (e.g., auto-withdraw once active balance crosses a threshold).
