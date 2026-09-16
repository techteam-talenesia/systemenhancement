# Reference / Utility Data

Source: https://open-api-paper-id.readme.io/reference/paper-utilities-1 (overview only — sub-pages not fetched in depth this pass)

Paper.id exposes a small set of "utility" / lookup endpoints that support the core invoice and payment calls (e.g., populating dropdowns or validating codes before submitting an invoice):

| Resource | Purpose | Endpoint (from docs index) |
|---|---|---|
| Tax List | Lookup valid `tax_id` values to attach to invoice line items | `GET /api/v1/tax-list` (path pattern per docs index; full param/response detail not captured) |
| Unit of Measurement (UoM) List | Lookup valid `uom_code` values for invoice line items | `GET /api/v1/uom-list` (same caveat) |
| Bank Code Reference | List of banks supported for bank-transfer payments and bank-account fields (`bank_code`) | Static reference page — https://open-api-paper-id.readme.io/reference/bank-code-reference |
| Unit of Measurement Reference | Full static list of default UoMs | Static reference page — https://open-api-paper-id.readme.io/reference/unit-of-measurement-reference |

## Integration relevance

These are low-priority for the initial integration but matter for **data quality**: when creating Sales Invoices programmatically, line items should use valid `tax_id` and `uom_code` values pulled from these lookups (cached locally, refreshed periodically) rather than hardcoded guesses, to avoid `400` validation errors at invoice-creation time.

**Gap:** exact request/response schemas for the two dynamic lookup endpoints (`tax-list`, `uom-list`) were not fetched in full detail in this pass — only their existence and purpose were confirmed via the Paper Utilities overview page.
