# Invoices — Sales (A/R) and Purchase (A/P)

Source: https://open-api-paper-id.readme.io/reference/sales-invoices, /reference/create-sales-invoice, /reference/retrieve-single-sales-invoice-details, /reference/retrieve-list-of-all-sales-invoices, /reference/update-sales-invoice, /reference/send-sales-invoice, /reference/delete-sales-invoice, /reference/stamp-sales-invoice, /reference/creating-sending-invoices, /reference/purchase-invoices, /reference/list-of-all-purchase-invoice, /reference/retrive-single-purchase-invoice-1, /reference/update-purchase-invoice

All endpoints require `client_id` / `client_secret` headers. Base URL (staging): `https://open-api.stag-v2.paper.id/api/v1` (v2 for some purchase-invoice endpoints — see below).

## Sales Invoices (Accounts Receivable — Talenesia billing students/clients)

The Sales Invoice API is Paper.id's description of a "robust platform...for A/R interactions" — this is the primary resource for Talenesia issuing bootcamp/course invoices.

### Create
`POST /store-invoice`

Request body:

| Field | Type | Required | Notes |
|---|---|---|---|
| `invoice_date` | string | yes | `DD-MM-YYYY` |
| `due_date` | string | yes | `DD-MM-YYYY` |
| `number` | string | yes | must be unique |
| `customer` | object `{id, name, phone required; email optional}` | yes | auto-creates/updates a Partner |
| `items` | array | yes | see below |
| `send` | object `{email, whatsapp, sms}` booleans | yes | delivery channels, default true each |
| `signature_text_header` / `signature_text_footer` | string | no | |
| `terms_condition` | string | no | |
| `notes` | string | no | |
| `additional_info` | object | no | free metadata |
| `additional_discount` + `additional_discount_type` (`percentage`/`amount`) | number/string | no | |
| `additional_fee` | array of `{delivery_fee: number}` | no | |

Item object: `name`, `description`, `quantity` (int), `price` (float) required; `discount`, `discount_type` (`amount`/`percentage`), `tax_id`, `uom_code`, `additional_info` optional.

Response (201):
```json
{
  "status_code": 201,
  "data": {
    "id": "aad61dca-...",
    "number": "INV/08/08/0021",
    "payper_url": "get.paper.id/21d1rdK",
    "pdf_url": "https://storage.googleapis.com/...",
    "pdf_url_short": "get.paper.id/W7vfptM",
    "status_send": {"email": true, "whatsapp": true, "sms": true}
  }
}
```
Error (400) example: `{"error": {"status_code": 400, "message": "number sudah dipakai."}}` (invoice number already used).

### Retrieve single invoice
`GET /sales-invoices/{invoice_id}`

Response highlights:
- `data.status.payment_status`: **`unpaid` | `partially paid` | `paid` | `overdue`** — this is the field to poll/sync if not relying on webhooks.
- `data.status.acceptance_status`: `not accepted` | `accepted`
- `data.payment_link`, `data.pdf_link`
- `data.customer` (id, company_name, company_email, company_phone, contacts)
- `data.items`, `data.attachments`, `additional_fee.delivery_fee`, `total`, `created_at`, `modified_at`

### List invoices
`GET /sales-invoices/all`

Query: `status` (`paid`|`partially_paid`), `limit`, `offset`, `start_date`, `end_date`, `order_by` (`asc`/`desc`).

Response: `{invoices: [{uuid, number, invoice_date, status (int code), links[]}], pagination: {limit, offset, total_records}}`. Note the list response returns a numeric `status` code rather than the string values seen on the single-invoice endpoint — the docs don't map the codes, so the detail endpoint is the more reliable source for status text.

### Update
`POST /sales-invoice/{invoice_id}/update` — same body shape as create (`invoice_date`, `due_date`, `number`, `customer`, `totals`, `items`, `send` required). Best-practice note from the docs: **if you update an already-sent invoice, re-send it** to avoid the customer paying against stale details.

### Send / resend
`POST /sales-invoices/send-all/{invoice_id}/` — body lets you target specific recipients: `email: {to, cc}`, `whatsapp: {number: []}`, `sms: {number: []}`. Response echoes per-channel delivery status.

### Delete
`DELETE /sales-invoice/{invoice_id}` → `{"status_code":200,"code":"SI-DLT-200","message":"OK"}`; 404 `ERR-CMN-000` if not found.

### E-Meterai (digital stamp duty)
`POST /sales-invoice/stamps` — applies Indonesian legal e-stamp to an invoice PDF and can (re)send it. Body: `invoice_number` + `send` required; optional stamp placement coordinates (`vis_llx/lly/urx/ury`, `vis_signature_page`), `retry_flag`. Response includes `pdf_link`, `sn_number` (stamp serial), remaining `quota`. A related endpoint, **Retrieve E-Meterai Balance**, exists to check remaining stamp quota but detail wasn't captured — see README gaps.

### Attachments
Two endpoints exist — **Add Attachment** and **Delete Attachment** on a Sales Invoice (upload/remove a file linked to an invoice, e.g. supporting documents). Full field-level detail wasn't captured in this pass.

## Purchase Invoices (Accounts Payable — bills Talenesia receives, e.g. from vendors)

Base path differs: `https://open-api.stag-v2.paper.id/api/v2/purchase-invoice`.

- **List**: `GET /purchase-invoice` — query: `limit`, `offset`, `order` (asc/desc), `sort` (default `created_at`), `start_invoice_date`/`end_invoice_date`, `start_due_date`/`end_due_date`. Response: `{code, message, data: [{uuid, invoice_number, status, dates, totals, items[], partner}]}`.
- **Retrieve single**: `GET /purchase-invoice/{invoice_id}` → includes `status` (e.g. `"DRAFT"`), `subtotal`, `grand_total`, `tax_inclusive`/`tax_exclusive`, `items`, `partner`, and Indonesian tax-invoice fields (`tax_invoice` with NPWP/faktur data). 404 → `ERR-INV-001`.
- **Update**: `PUT /purchase-invoices/:id` — body: `partner_id` (must be an existing partner) + `invoice_number` required, `items[]` required (`product_name`, `quantity`, `price` required per line), plus optional dates/notes/terms.
- **Delete**: endpoint exists (`delete-purchase-invoice` in docs index) but detail wasn't fetched in this pass.

Purchase invoices are lower priority for Talenesia's initial integration (Talenesia is mainly issuing invoices, not receiving many), but relevant if vendor/expense tracking is in scope later.

## Integration relevance

- `POST /store-invoice` replaces manual invoice creation in Paper.id's dashboard — this can be triggered directly from a sales/CRM event (e.g., a Qontak deal marked "won").
- `GET /sales-invoices/{invoice_id}` (or the list endpoint) is the pull-based way to sync status if webhooks are not used; **`payment_status`** is the field to write into the sales spreadsheet / accounting system.
- Webhooks (see `webhooks.md`) are the push-based, real-time alternative — far more useful for "auto-sync instead of manual export/import."
