# Mapao - Metadata (UI Generation)

### `GET /api/v1/metadata/form/{record_type_slug}`

_Get Form Metadata_

Get form metadata for a specific record type by slug.
Frontend ใช้ endpoint นี้สร้าง form สำหรับ record type นั้นๆ

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_slug` | path | `string` | yes |  |

**Responses**

- **200** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/metadata/record-types`

_Get All Types With Schemas_

Get all record types with their current schemas.
Frontend ใช้ endpoint นี้เพื่อสร้างเมนู + forms อัตโนมัติ

**Auth:** required (cookie or Bearer)

**Responses**

- **200** Successful Response


### `POST /api/v1/metadata/schemas/{record_type_id}/check-compatibility`

_Check Schema Compatibility_

Check if a new schema is backward compatible with the current one.
ใช้ก่อน create schema version ใหม่ เพื่อดูว่ามี breaking changes ไหม

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `string` | yes |  |

**Request body** (`application/json`)

**Responses**

- **200** Successful Response
- **422** Validation Error - `HTTPValidationError`

