# Webhooks / Callbacks

Source: https://open-api-paper-id.readme.io/reference/handling-api-callbacks, /reference/payment-callbacks

This is the most important area for Talenesia's integration: it's what would let payment/invoice status changes push automatically into internal systems instead of staff manually checking Paper.id and re-keying data.

## Callback types

The docs describe **four** callback events:

| Callback | Fires when |
|---|---|
| **Payment In** | An inbound (A/R) payment transaction succeeds or fails |
| **Payment Out** | An outbound (A/P / payout) payment transaction succeeds or fails |
| **Invoice Paid** | A Sales Invoice's status changes to `PAID` |
| **Disbursement** | Funds are disbursed to a destination bank account (withdrawal completes) |

Important scoping note directly from the docs: the Payment callback **"is catered for a successful/failed payment only, and not catering the disbursement process"** — disbursement completion is a separate callback (Disbursement Callback), not folded into the payment callback.

Also explicitly noted: **"You need to register your URL to both payment and invoice callback, if you use Sales Invoice API"** — i.e., if Talenesia integrates via Sales Invoices (the expected path), you must register two separate callback URLs (payment callback + invoice callback), not just one, to get full status coverage.

## Registration

Callback URLs are registered **via the Paper.id dashboard**, not via an API call, per the docs (the exact settings-page navigation wasn't captured in this pass — likely under the same "API Integration" settings area used for API keys; confirm during implementation).

## Payload — Payment callback

```json
{
  "ref_id": "<your client-generated payment/transaction reference>",
  "external_id": "<Paper.id's payment reference, shown in their dashboard>",
  "payment_date": "YYYY-MM-DD",
  "message": "<status message>",
  "payment_info": {
    "method": "bank_transfer | credit_card | ewallet | qris | mitra_pembayaran_digital",
    "channel": "<specific brand/channel, e.g. BRI, OVO, QRIS>",
    "amount": 0,
    "paid_amount": 0,
    "paid_at": "<timestamp>",
    "status": "PAID",
    "updated": "<timestamp>",
    "source": "paper-chain"
  },
  "additional_info": {
    "invoices": [ { "uuid": "...", "number": "..." } ]
  }
}
```

`additional_info.invoices` is the join key back to a specific Sales Invoice (`uuid`/`number`) — this is what you'd match against your internal order/enrollment record to mark a student's payment as received.

## Invoice Paid / Payment In / Payment Out / Disbursement — payload detail

Only the generic Payment callback payload above was retrievable in full. The docs index lists Invoice Paid, Payment In, Payment Out, and Disbursement as distinct callback types under "Handling API Callbacks," but per-event payload schemas beyond the Payment callback shown above were not present in the fetched content. **Gap — recommend requesting a live webhook sample or a Paper.id integration contact walkthrough before building the receiving endpoint**, since exact field names may differ slightly per event type.

## Authenticity / signature verification — GAP

**No HMAC signing, shared secret, or signature-header scheme is documented anywhere in the fetched pages.** Neither `handling-api-callbacks` nor `payment-callbacks` mentions verifying that an inbound request genuinely originated from Paper.id (e.g., no `X-Paper-Signature` header, no documented shared secret to compute an HMAC against). This is a real gap to flag in the engineering brief: a production webhook receiver should, at minimum:
- Only accept callbacks from Paper.id's known outbound IP range if published (not found in docs — ask Paper.id support),
- Treat the payload as advisory and always re-confirm status via a `GET` call (e.g., `GET /sales-invoices/{invoice_id}`) before taking an irreversible action (e.g., marking a paid enrollment as confirmed),
- Ask the Paper.id account/integration team directly whether a signing mechanism exists but is undocumented in the public reference.

## Retry policy

Not documented for production. In **staging**, the docs explicitly state callback retries are **not available** — if a staging callback delivery fails, the only way to test again is to create a new invoice/payment. Production retry behavior (attempts, backoff, timeout) was not found in the fetched pages — another item to confirm directly with Paper.id.

## Expected acknowledgment response

Not documented in the fetched pages (typical practice for these systems is a `200 OK` from the receiver within a short timeout, but Paper.id's own requirement — timeout window, expected body — was not stated). Flag as a build-time question.

## Integration relevance (the core of the brief)

This is the mechanism that replaces manual export/import for payment status:

1. Register both the **payment callback** and **invoice callback** URLs against a small internal receiving endpoint (e.g., a lightweight webhook handler service).
2. On receipt of a Payment/Invoice-Paid callback, look up the invoice by `additional_info.invoices[].uuid` (or `number`), confirm status via a `GET` call (defensive re-check given no signature verification), then:
   - Update the sales spreadsheet / internal order record automatically,
   - Push a status update into Qontak CRM (e.g., "payment received" stage change) via Qontak's own API,
   - Push the transaction into the accounting system instead of manual CSV export/import.
3. Use the **Disbursement callback** similarly to auto-confirm when settled funds actually land in Talenesia's bank account, closing the reconciliation loop without staff checking the dashboard.
