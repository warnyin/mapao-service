# Mapao - Attachments

### `DELETE /api/v1/attachments/{attachment_id}`

_Delete Attachment_

Remove the attachment from MinIO and the DB.

**Auth:** required (cookie or Bearer)

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

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `attachment_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `AttachmentDownload`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/records/{record_id}/attachments`

_Upload Attachment_

Upload a file and attach it to a record.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |

**Request body** (`multipart/form-data`)

`Body_upload_attachment_api_v1_records__record_id__attachments_post`

**Responses**

- **201** Successful Response - `AttachmentResponse`
- **422** Validation Error - `HTTPValidationError`

