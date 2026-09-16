# Response Format, Status Codes & Errors

Source: https://open-api-paper-id.readme.io/reference/responses

Paper.id's API is inconsistent between an older "v1" response shape and a newer "v2" shape — worth knowing so a client library can normalize both.

## Success (v2-style, e.g. Partners endpoints)

```json
{
  "message": "success",
  "data": { }
}
```

## Error (v1-style, seen generally and on several endpoints)

```json
{
  "status_code": 400,
  "message": "error message",
  "data": { }
}
```

In practice, individual endpoints documented their own error shapes with small variations, e.g.:
- Sales Invoice delete: `{"status_code":404,"code":"ERR-CMN-000","error":"Invoice not found","message":"Not Found"}`
- Sales Invoice create: `{"error":{"status_code":400,"message":"number sudah dipakai."}}`
- Purchase Invoice list/update: `{"message":"...", "type":"invalid_request_error", "errors":{...}}`
- Partner endpoints: `{"status_code":..., "error_code":"...", "message":"...", "data":{}, "errors":{}}`

**Practical implication:** don't assume one universal error envelope — an integration layer should defensively check for `status_code` at top level, `error.status_code`, and `errors`/`data` sub-objects, and treat any of them as failure signals.

## HTTP status codes used

| Code | Meaning |
|---|---|
| 200 | Success |
| 201 | Success, resource created |
| 204 | Success, no content |
| 400 | Bad request — missing/invalid parameters |
| 401 | Authentication failed / insufficient permission |
| 403 | Access denied |
| 404 | Resource not found |
| 405 | Method not allowed for this resource |
| 500 | Server error |
