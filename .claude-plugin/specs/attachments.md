# Mapao - Attachments

### `DELETE /api/v1/attachments/{attachment_id}`

_Delete Attachment_

Remove the attachment from MinIO and the DB.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `attachment_id` | path | `string` | yes |  |

**Responses**

- **204** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/attachments/{attachment_id}`

_Get Attachment Url_

Return a presigned download URL for a stored attachment.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `attachment_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `AttachmentDownload`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `file_name` | `string` | yes |  |
| `file_type` | `string \| null` | yes |  |
| `file_size` | `integer \| null` | yes |  |
| `url` | `string` | yes |  |
| `expires_in_seconds` | `integer` | yes |  |


### `POST /api/v1/records/{record_id}/attachments`

_Upload Attachment_

Upload a file and attach it to a record.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |

**Request body** (`multipart/form-data`)

`Body_upload_attachment_api_v1_records__record_id__attachments_post`

**Responses**

- **201** Successful Response - `AttachmentResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `record_id` | `string` | yes |  |
| `tenant_id` | `string` | yes |  |
| `file_name` | `string` | yes |  |
| `file_type` | `string \| null` | yes |  |
| `storage_path` | `string` | yes |  |
| `file_size` | `integer \| null` | yes |  |
| `created_at` | `string` | yes |  |

