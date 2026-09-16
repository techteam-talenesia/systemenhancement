# Paper.id Open API — Reference Notes

Compiled for the Talenesia engineering brief on integrating Paper.id programmatically (replacing manual import/export between Paper.id, Qontak CRM, sales spreadsheets, and accounting).

Primary source: https://open-api-paper-id.readme.io/reference/getting-started-1 and the pages linked from its documentation index (https://open-api-paper-id.readme.io/llms.txt). Compiled 2026-09-16.

Files in this folder:
- `authentication.md` — auth model, credentials, environments
- `partners.md` — customer/supplier resource (CRUD)
- `invoices.md` — Sales Invoice (A/R) and Purchase Invoice (A/P) endpoints
- `payments.md` — payment methods, balance, withdrawals
- `webhooks.md` — callback events for payment/invoice status changes (the key mechanism for auto-sync)
- `errors_and_responses.md` — response envelope(s) and HTTP status codes
- `reference_data.md` — tax/UoM/bank code lookups

## Auth model, in one paragraph

Paper.id's Open API uses a simple **static API key pair** — a `client_id` and `client_secret` generated from the dashboard (Settings → API Integration, requires an `owner`-role account) — sent as plain HTTP headers on every request. There is no OAuth flow, no token expiry/refresh, and no documented per-request signing for the REST API itself. Separate credentials exist for the staging environment (`https://open-api.stag-v2.paper.id`, no KYC required, real payments disabled — use the payment-simulation endpoint instead) and production (requires full KYC/KYB). Notably, **inbound webhook callbacks have no documented signature-verification mechanism** — this is flagged as a gap and a question to put directly to Paper.id before relying on callbacks for anything irreversible.

## Endpoint summary

| Method | Path | Purpose |
|---|---|---|
| GET | `/api/v2/partners` | List partners (customers/suppliers) |
| POST | `/api/v2/partners` | Create partner |
| GET | `/api/v2/partners/{partnerId}` | Get partner detail |
| PUT | `/api/v2/partners/{partnerId}` | Update partner |
| GET | `/api/v1/bank-account/{bankId}` | Retrieve a partner bank account |
| POST | `/api/v1/bank-account/partner` | Add a partner bank account |
| POST | `/api/v1/store-invoice` | Create Sales Invoice |
| GET | `/api/v1/sales-invoices/{invoice_id}` | Get Sales Invoice detail (incl. `payment_status`) |
| GET | `/api/v1/sales-invoices/all` | List Sales Invoices (filter by status/date) |
| POST | `/api/v1/sales-invoice/{invoice_id}/update` | Update Sales Invoice |
| POST | `/api/v1/sales-invoices/send-all/{invoice_id}/` | (Re)send Sales Invoice to customer |
| DELETE | `/api/v1/sales-invoice/{invoice_id}` | Delete Sales Invoice |
| POST | `/api/v1/sales-invoice/stamps` | Apply e-Meterai digital stamp duty |
| — | (Retrieve E-Meterai Balance) | Check remaining stamp quota — detail not fully captured |
| — | (Sales Invoice Add/Delete Attachment) | Manage files attached to a Sales Invoice — detail not fully captured |
| GET | `/api/v2/purchase-invoice` | List Purchase Invoices |
| GET | `/api/v2/purchase-invoice/{invoice_id}` | Get Purchase Invoice detail |
| PUT | `/api/v2/purchase-invoices/:id` | Update Purchase Invoice |
| — | (Delete Purchase Invoice) | Exists per docs index — detail not fetched |
| — | (Payment Request — create) | Direct payment request w/ fixed method — exact path not captured |
| POST | `/api/v1/payment/simulation` | **Staging only** — force a Payment Request to `PAID` |
| GET | `/api/v1/payment/balance` | Get digital payment balance (active + on-hold) |
| GET | `/api/v1/payment/balance/list` | List balance transactions |
| POST | `/api/v1/withdrawal/request` | Request withdrawal of balance to bank account |
| — | (OTP Verification Request) | Referenced for withdrawal/bank-account changes — detail not fetched |
| — | (Add Withdrawal Bank Account) | Referenced — detail not fetched |
| GET | `/api/v1/tax-list` | Lookup valid tax codes for invoice line items |
| GET | `/api/v1/uom-list` | Lookup valid unit-of-measurement codes |
| — | Bank Code Reference (static) | Supported banks list |
| — | Unit of Measurement Reference (static) | Default UoM list |

**Webhooks (push, not pulled via the table above)** — registered via dashboard, not an API call:

| Callback | Fires on |
|---|---|
| Payment In | Inbound (A/R) payment success/failure |
| Payment Out | Outbound (A/P) payment success/failure |
| Invoice Paid | Sales Invoice status → `PAID` |
| Disbursement | Withdrawal/payout funds actually disbursed to bank |

Both a payment callback URL **and** an invoice callback URL must be registered if integrating via the Sales Invoice API (registering only one leaves a coverage gap). Payment callback payload includes `ref_id`, `external_id`, `payment_date`, `payment_info.{method, channel, amount, paid_amount, paid_at, status, updated}`, and `additional_info.invoices[].{uuid, number}` as the join key back to the invoice. See `webhooks.md` for full payload and the signature-verification gap.

## Integration implications for Talenesia

This is the part that matters most for the brief — what replaces today's manual import/export:

1. **Real-time payment-status sync (the biggest win).** Today, someone presumably checks the Paper.id dashboard and manually updates the sales spreadsheet / Qontak / accounting when a student pays. Registering the **Invoice Paid** and **Payment In** webhook callbacks against a small internal receiving service eliminates that manual step entirely: the moment a student pays, Paper.id pushes a callback containing the invoice UUID/number and payment amount/timestamp, which an internal handler can use to (a) flip the enrollment/order record to "paid" in the sales spreadsheet's replacement system, (b) update the deal/contact stage in Qontak via Qontak's own API, and (c) post the transaction into accounting — all within seconds instead of at whatever cadence manual exports happen today.
2. **Defensive reconciliation, not blind trust of webhooks.** Because Paper.id doesn't document webhook signature verification, the handler should treat a callback as a trigger to re-confirm via `GET /sales-invoices/{invoice_id}` (which returns the authoritative `payment_status`) before writing anything irreversible (e.g., before auto-granting course access). This is a "verify, don't trust" pattern, not a blocker to automation.
3. **No more manual invoice creation.** `POST /store-invoice` means invoices can be generated automatically the moment a CRM deal is marked won or an enrollment form is submitted — pre-filled with the customer's Partner record — instead of staff manually re-keying invoice details into Paper.id's UI.
4. **Customer/partner data as a single sync target.** The Partner API (`partners.md`) gives a natural two-way sync point with Qontak: push new/updated CRM contacts into Paper.id as Partners using a stable `number` (e.g., the Qontak contact ID) so repeated syncs update rather than duplicate.
5. **Bank reconciliation and cash-out automation.** The Digital Payment Balance and Withdrawal APIs mean even the "move settled Paper.id balance into Talenesia's real bank account" step — plus the bookkeeping of matching balance transactions to invoices — could be automated or at least monitored, rather than checked manually in the dashboard, closing the loop with the **Disbursement callback**.
6. **Data-quality guardrails for automated invoice creation.** Use the Tax List / UoM List lookups to validate `tax_id`/`uom_code` before submitting invoices programmatically, avoiding silent validation failures that a human would otherwise catch by eyeballing the dashboard form.

## Gaps / pages not (fully) captured

These were either not linked from the docs index, 404'd, or only yielded an overview rather than full field-level detail in this pass — worth a direct follow-up with Paper.id's integration team before finalizing the brief's technical appendix:

- Exact production base URL (only the staging host, `open-api.stag-v2.paper.id`, was shown in examples).
- Exact HTTP method/path for creating a **direct Payment Request** (as distinct from the Sales Invoice `store-invoice` flow) — the docs describe the concept and the payment-method table but not the concrete endpoint signature.
- Webhook **registration steps** (which dashboard screen/form) and **retry policy in production** (staging explicitly has no retry).
- Webhook **payload schemas for Payment Out, Invoice Paid, and Disbursement** specifically (only the general Payment callback payload — which covers Payment In/Out — was fully retrievable).
- Whether any **webhook signature/HMAC verification** exists but is simply undocumented in the public reference — this should be asked directly, since building a production receiver without it is a real security consideration.
- Full field schemas for: `GET /api/v1/tax-list`, `GET /api/v1/uom-list`, `GET /api/v1/bank-account/{bankId}`, `POST /api/v1/bank-account/partner`, OTP Verification Request, Add Withdrawal Bank Account, Sales Invoice attachment endpoints, Retrieve E-Meterai Balance, Delete Purchase Invoice.
- `changelog` and `service-status-1` pages exist in the docs index but were intentionally skipped as out of scope for a technical reference.
