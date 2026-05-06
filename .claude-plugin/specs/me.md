# Mapao - Me (PDPA self-service)

### `DELETE /api/v1/me`

_Delete My Account_

Anonymize my user row, scrub PII from my records, and revoke tokens.

PDPA / GDPR "right to erasure":
  1. null PII on the user row (email/full_name/hashed_password/preferences)
     and flag inactive
  2. null `records.created_by` and `campaigns.created_by` on every row
     I authored so the audit trail no longer ties data to my identity
  3. scrub free-text keys in `records.attributes` that commonly hold PII
     (name, email, phone, address, note/description) — we keep the record
     structurally intact (the tenant may still rely on its geometry and
     classification), we only strip the personal-data keys
  4. revoke all refresh tokens

**Auth:** required (cookie or Bearer)

**Responses**

- **204** Successful Response


### `GET /api/v1/me/export`

_Export My Data_

Return a JSON dump of every row I own in this tenant.

Rate-limited via ``settings.rate_limit_export`` (default 5/hour) — this is
a DB-heavy endpoint and abuse risk is high without a cap.

**Auth:** required (cookie or Bearer)

**Responses**

- **200** Successful Response


### `GET /api/v1/me/external-identities`

_List My External Identities_

**Auth:** required (cookie or Bearer)

**Responses**

- **200** Successful Response - `array<ExternalIdentityResponse>`


### `DELETE /api/v1/me/external-identities/{identity_id}`

_Unlink External Identity_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `identity_id` | path | `uuid` | yes |  |

**Responses**

- **204** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/me/external-identities/{provider_slug}/authorize`

_Authorize External Identity Link_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `provider_slug` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `ExternalAuthorizeResponse`
- **422** Validation Error - `HTTPValidationError`

