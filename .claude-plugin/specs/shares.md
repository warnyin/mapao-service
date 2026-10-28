# Mapao - Share Links

### `POST /api/v1/records/{record_id}/share`

_Create Share Link_

Issue a time-limited public share link for a record.

Only admin/editor can share. The response is the only place the
cleartext token appears — callers must save the URL immediately.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `expires_in_hours` | `integer` | no | Link validity (max 90 days) |
| `label` | `string | null` | no | Human-readable note, e.g. 'shared with partner X' |

**Responses**

- **201** Successful Response - `ShareLinkResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/records/{record_id}/shares`

_List Share Links_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `list[ShareLinkResponse]`
- **422** Validation Error - `HTTPValidationError`


### `DELETE /api/v1/shares/{share_id}`

_Revoke Share Link_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `share_id` | path | `string` | yes |  |

**Responses**

- **204** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/shares/{share_id}/views`

_List Share Views_

Audit log of who has viewed this share link.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `share_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `list[ShareLinkViewResponse]`
- **422** Validation Error - `HTTPValidationError`


### `GET /public/r/{token}`

_Public View Record_

Anonymous read-only view of a record via share link.

Validates the token, logs the view (IP/UA/referer), and returns a
minimal GeoJSON Feature. The link is found by ``resolve_share_link()``
(record_share_links is RLS-scoped and no tenant is known yet); everything
after that runs as the link's own tenant.

**Auth:** public

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `token` | path | `string` | yes |  |

**Responses**

- **200** Successful Response
- **422** Validation Error - `HTTPValidationError`

