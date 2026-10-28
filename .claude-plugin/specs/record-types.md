# Mapao - Record Types

### `GET /api/v1/record-types/`

_List Record Types_

List all record types for current tenant (RLS-filtered).

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Responses**

- **200** Successful Response - `list[RecordTypeResponse]`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `tenant_id` | `string` | yes |  |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string \| null` | yes |  |
| `icon` | `string \| null` | yes |  |
| `color` | `string \| null` | yes |  |
| `default_map_config` | `object` | yes |  |
| `is_active` | `boolean` | yes |  |
| `sort_order` | `integer` | yes |  |
| `visibility` | `string` | yes |  |
| `trust_config` | `object \| null` | no |  |
| `created_at` | `string` | yes |  |
| `updated_at` | `string` | yes |  |


### `POST /api/v1/record-types/`

_Create Record Type_

Create a new record type (admin only).

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string \| null` | no |  |
| `icon` | `string \| null` | no |  |
| `color` | `string \| null` | no |  |
| `default_map_config` | `object` | no |  |
| `sort_order` | `integer` | no |  |
| `visibility` | `string` | no | 'public' exposes the collection via the anonymous OGC API |

**Responses**

- **201** Successful Response - `RecordTypeResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `tenant_id` | `string` | yes |  |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string \| null` | yes |  |
| `icon` | `string \| null` | yes |  |
| `color` | `string \| null` | yes |  |
| `default_map_config` | `object` | yes |  |
| `is_active` | `boolean` | yes |  |
| `sort_order` | `integer` | yes |  |
| `visibility` | `string` | yes |  |
| `trust_config` | `object \| null` | no |  |
| `created_at` | `string` | yes |  |
| `updated_at` | `string` | yes |  |


### `GET /api/v1/record-types/{record_type_id}`

_Get Record Type_

Get record type details.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `RecordTypeResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `tenant_id` | `string` | yes |  |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string \| null` | yes |  |
| `icon` | `string \| null` | yes |  |
| `color` | `string \| null` | yes |  |
| `default_map_config` | `object` | yes |  |
| `is_active` | `boolean` | yes |  |
| `sort_order` | `integer` | yes |  |
| `visibility` | `string` | yes |  |
| `trust_config` | `object \| null` | no |  |
| `created_at` | `string` | yes |  |
| `updated_at` | `string` | yes |  |


### `PATCH /api/v1/record-types/{record_type_id}`

_Update Record Type_

Update a record type (admin only).

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `string` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string \| null` | no |  |
| `description` | `string \| null` | no |  |
| `icon` | `string \| null` | no |  |
| `color` | `string \| null` | no |  |
| `default_map_config` | `object \| null` | no |  |
| `is_active` | `boolean \| null` | no |  |
| `sort_order` | `integer \| null` | no |  |
| `visibility` | `string \| null` | no |  |
| `trust_config` | `TrustConfig \| null` | no |  |

**`TrustConfig`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `max_accuracy_m` | `number \| null` | no |  |
| `max_speed_kmh` | `number \| null` | no |  |
| `max_fix_age_s` | `number \| null` | no |  |

**Responses**

- **200** Successful Response - `RecordTypeResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `tenant_id` | `string` | yes |  |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string \| null` | yes |  |
| `icon` | `string \| null` | yes |  |
| `color` | `string \| null` | yes |  |
| `default_map_config` | `object` | yes |  |
| `is_active` | `boolean` | yes |  |
| `sort_order` | `integer` | yes |  |
| `visibility` | `string` | yes |  |
| `trust_config` | `object \| null` | no |  |
| `created_at` | `string` | yes |  |
| `updated_at` | `string` | yes |  |


### `GET /api/v1/record-types/{record_type_id}/schemas`

_List Schemas_

List all schema versions for a record type.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `list[RecordSchemaResponse]`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `record_type_id` | `string` | yes |  |
| `version` | `integer` | yes |  |
| `json_schema` | `object` | yes |  |
| `ui_schema` | `object` | yes |  |
| `is_current` | `boolean` | yes |  |
| `created_by` | `string \| null` | yes |  |
| `created_at` | `string` | yes |  |


### `POST /api/v1/record-types/{record_type_id}/schemas`

_Create Schema Version_

Create a new schema version (admin only; append-only, auto-increments version).

Computes ``warnings`` against the previous current schema so the
operator can spot breaking changes (removed fields, type changes,
new required keys) before relying on the new version. Warnings are
informational — the new version is always created.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `string` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `record_type_id` | `string` | yes |  |
| `json_schema` | `object` | yes | JSON Schema spec for validation |
| `ui_schema` | `object` | no | UI rendering hints |

**Responses**

- **201** Successful Response - `RecordSchemaCreateResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `record_type_id` | `string` | yes |  |
| `version` | `integer` | yes |  |
| `json_schema` | `object` | yes |  |
| `ui_schema` | `object` | yes |  |
| `is_current` | `boolean` | yes |  |
| `created_by` | `string \| null` | yes |  |
| `created_at` | `string` | yes |  |
| `warnings` | `list[string]` | no |  |


### `GET /api/v1/record-types/{record_type_id}/schemas/current`

_Get Current Schema_

Get the current (latest) schema for a record type — used by Frontend for auto-UI.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `RecordSchemaResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `string` | yes |  |
| `record_type_id` | `string` | yes |  |
| `version` | `integer` | yes |  |
| `json_schema` | `object` | yes |  |
| `ui_schema` | `object` | yes |  |
| `is_current` | `boolean` | yes |  |
| `created_by` | `string \| null` | yes |  |
| `created_at` | `string` | yes |  |

