# Mapao - Record Types

### `GET /api/v1/record-types/`

_List Record Types_

List all record types for current tenant (RLS-filtered).

**Auth:** required (cookie or Bearer)

**Responses**

- **200** Successful Response - `array<RecordTypeResponse>`


### `POST /api/v1/record-types/`

_Create Record Type_

Create a new record type (admin only).

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string | null` | no |  |
| `icon` | `string | null` | no |  |
| `color` | `string | null` | no |  |
| `default_map_config` | `object` | no |  |
| `sort_order` | `integer` | no |  |
| `visibility` | `string` | no | 'public' exposes the collection via the anonymous OGC API |

**Responses**

- **201** Successful Response - `RecordTypeResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/record-types/{record_type_id}`

_Get Record Type_

Get record type details.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `uuid` | yes |  |

**Responses**

- **200** Successful Response - `RecordTypeResponse`
- **422** Validation Error - `HTTPValidationError`


### `PATCH /api/v1/record-types/{record_type_id}`

_Update Record Type_

Update a record type (admin only).

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `uuid` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string | null` | no |  |
| `description` | `string | null` | no |  |
| `icon` | `string | null` | no |  |
| `color` | `string | null` | no |  |
| `default_map_config` | `object | null` | no |  |
| `is_active` | `boolean | null` | no |  |
| `sort_order` | `integer | null` | no |  |
| `visibility` | `string | null` | no |  |

**Responses**

- **200** Successful Response - `RecordTypeResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/record-types/{record_type_id}/schemas`

_List Schemas_

List all schema versions for a record type.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `uuid` | yes |  |

**Responses**

- **200** Successful Response - `array<RecordSchemaResponse>`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/record-types/{record_type_id}/schemas`

_Create Schema Version_

Create a new schema version (admin only; append-only, auto-increments version).

Computes ``warnings`` against the previous current schema so the
operator can spot breaking changes (removed fields, type changes,
new required keys) before relying on the new version. Warnings are
informational — the new version is always created.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `uuid` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `record_type_id` | `uuid` | yes |  |
| `json_schema` | `object` | yes | JSON Schema spec for validation |
| `ui_schema` | `object` | no | UI rendering hints |

**Responses**

- **201** Successful Response - `RecordSchemaCreateResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/record-types/{record_type_id}/schemas/current`

_Get Current Schema_

Get the current (latest) schema for a record type — used by Frontend for auto-UI.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `uuid` | yes |  |

**Responses**

- **200** Successful Response - `RecordSchemaResponse`
- **422** Validation Error - `HTTPValidationError`

