# Mapao - Endpoint Catalog

### auth

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/v1/auth/accept-invite` | Accept Invite |
| GET | `/api/v1/auth/external/{tenant_slug}/{provider_slug}/authorize` | Authorize External Login |
| GET | `/api/v1/auth/external/{tenant_slug}/{provider_slug}/callback` | External Auth Callback |
| POST | `/api/v1/auth/forgot-password` | Forgot Password |
| POST | `/api/v1/auth/login` | Login |
| GET | `/api/v1/auth/login-options/{tenant_slug}` | Get Login Options |
| POST | `/api/v1/auth/logout` | Logout |
| GET | `/api/v1/auth/me` | Get Current User Profile |
| POST | `/api/v1/auth/refresh` | Refresh Token |
| POST | `/api/v1/auth/register` | Register |
| POST | `/api/v1/auth/reset-password` | Reset Password |

### me

| Method | Path | Purpose |
| --- | --- | --- |
| DELETE | `/api/v1/me` | Delete My Account |
| GET | `/api/v1/me/export` | Export My Data |
| GET | `/api/v1/me/external-identities` | List My External Identities |
| DELETE | `/api/v1/me/external-identities/{identity_id}` | Unlink External Identity |
| POST | `/api/v1/me/external-identities/{provider_slug}/authorize` | Authorize External Identity Link |

### record-types

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/v1/record-types/` | List Record Types |
| POST | `/api/v1/record-types/` | Create Record Type |
| GET | `/api/v1/record-types/{record_type_id}` | Get Record Type |
| PATCH | `/api/v1/record-types/{record_type_id}` | Update Record Type |
| GET | `/api/v1/record-types/{record_type_id}/schemas` | List Schemas |
| POST | `/api/v1/record-types/{record_type_id}/schemas` | Create Schema Version |
| GET | `/api/v1/record-types/{record_type_id}/schemas/current` | Get Current Schema |

### records

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/v1/records/` | List Records |
| POST | `/api/v1/records/` | Create Record |
| DELETE | `/api/v1/records/{record_id}` | Delete Record |
| GET | `/api/v1/records/{record_id}` | Get Record |
| PATCH | `/api/v1/records/{record_id}` | Update Record |

### campaigns

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/v1/campaigns/` | List Campaigns |
| POST | `/api/v1/campaigns/` | Create Campaign |
| GET | `/api/v1/campaigns/{campaign_id}` | Get Campaign |
| PATCH | `/api/v1/campaigns/{campaign_id}` | Update Campaign |

### attachments

| Method | Path | Purpose |
| --- | --- | --- |
| DELETE | `/api/v1/attachments/{attachment_id}` | Delete Attachment |
| GET | `/api/v1/attachments/{attachment_id}` | Get Attachment Url |
| POST | `/api/v1/records/{record_id}/attachments` | Upload Attachment |

### metadata

| Method | Path | Purpose |
| --- | --- | --- |
| GET | `/api/v1/metadata/form/{record_type_slug}` | Get Form Metadata |
| GET | `/api/v1/metadata/record-types` | Get All Types With Schemas |
| POST | `/api/v1/metadata/schemas/{record_type_id}/check-compatibility` | Check Schema Compatibility |

### shares

| Method | Path | Purpose |
| --- | --- | --- |
| POST | `/api/v1/records/{record_id}/share` | Create Share Link |
| GET | `/api/v1/records/{record_id}/shares` | List Share Links |
| DELETE | `/api/v1/shares/{share_id}` | Revoke Share Link |
| GET | `/api/v1/shares/{share_id}/views` | List Share Views |
| GET | `/public/r/{token}` | Public View Record |
