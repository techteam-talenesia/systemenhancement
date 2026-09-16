# Partners (Customers / Suppliers / Contacts)

Source: https://open-api-paper-id.readme.io/reference/managing-partners, /reference/partner-1, /reference/get_api-v2-partners, /reference/post_api-v2-partners, /reference/get_api-v2-partners-partnerid, /reference/put_api-v2-partners-partnerid, /reference/partner-bank-account

## What a "Partner" is

A single unified object represents both **buyers (customers)** and **suppliers/vendors**. There is no separate "customer" vs "contact" resource — it's all "Partner", disambiguated by a `type` field (`client`/`supplier`/`both`). This is the natural mapping target for Talenesia's students/companies in Qontak CRM and the sales spreadsheet.

Key rules called out in the docs:
- `number` (partner code) + `name` should be treated as a stable pair. If you send an existing `number` with a different `name`, the API errors out (protects against accidental overwrite of the wrong partner).
- Matching `number` + `name` on a create/reference call **updates** the existing partner rather than creating a duplicate — useful for idempotent sync from an external CRM (e.g., using the Qontak contact ID as the Paper.id partner `number`).
- Partners can also be auto-created implicitly by referencing a `customer` object inline inside an invoice creation call (see `invoices.md`), not just via the dedicated Partner endpoints below.

## Endpoints

All require `client_id` / `client_secret` headers.

### List partners
`GET /api/v2/partners`

Query params (all optional): `limit` (default 10, max 100), `offset`, `type`, `name`, `number`, `mobile`, `phone`, `email`.

Response: `{ message, pagination: {limit, offset, total_record, page, total_page}, data: [Partner...] }`

### Create partner
`POST /api/v2/partners`

Request body:

| Field | Type | Required |
|---|---|---|
| `name` | string | yes |
| `number` | string | yes — unique per company |
| `type` | string enum (`Supplier`/`supplier`, `Client`/`client`, `Both`/`both`) | yes |
| `phone` | string, E.164 | yes |
| `email` | string | no |
| `mobile_phone` | string, E.164 | no |
| `business_type` | enum: `pt`, `cv`, `perorangan`, `lainnya` | no |
| `address` | object (`address_line_1`, `address_line_2`, `state`, `city`, `postal_code`, `country`) | no |
| `notes` | string | no |
| `virtual_account` | string | no |
| `contacts` | array of `{name, email, phone, position}` (all required per item) | no |
| `bank_accounts` | array of `{bank_code, bank_account_number}` (required per item) | no |
| `custom_type` | string | no |

Response (201): partner object with `id`, plus all submitted fields, `created_at`, `updated_at`.

### Get partner
`GET /api/v2/partners/{partnerId}`

Response includes everything above plus:
- `counter_parties` (object)
- `payment_method` — booleans per channel: `credit_card`, `bank_transfer`, `ewallet`, `mitra_pembayaran_digital`, `qris` (which payment methods this partner has actually used/enabled)

404 if not found.

### Update partner
`PUT /api/v2/partners/{partnerId}`

Same required/optional fields as create (`name`, `number`, `type`, `phone` required). Returns 201 with the updated object.

## Partner Bank Account

Separate sub-resource for managing a partner's bank account details (used e.g. for supplier disbursement / payout scenarios):
- `GET /api/v1/bank-account/{bankId}` — retrieve a bank account
- `POST /api/v1/bank-account/partner` — add a bank account to a partner

(Only overview-level detail was retrievable for these two; full field lists were not captured — see README gaps.)

## Integration relevance

This is the resource to sync **bidirectionally** with Qontak CRM contacts: push new/updated CRM contacts into Paper.id as Partners (`POST`/`PUT /api/v2/partners`), and optionally pull Paper.id partner `payment_method` history back into CRM to see which channels a customer actually uses.
