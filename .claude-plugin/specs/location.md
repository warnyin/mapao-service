# Mapao - Location trust (GPS evidence, assess)

### `POST /api/v1/location/actors/erase`

_Erase Actor_

Forget an actor: clear its keyed hash on every record of this tenant (RLS-scoped),
so its travel history is no longer linked. Records themselves are kept.

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `actor_ref` | `string` | yes |  |

**Responses**

- **200** Successful Response - `EraseResponse`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/location/assess`

_Assess_

Grade a fix exactly as record creation would; stores nothing.

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `accuracy_m` | `number` | yes |  |
| `captured_at` | `string | null` | no |  |
| `mock_reported` | `boolean` | no |  |
| `actor_ref` | `string | null` | no |  |
| `lat` | `number` | yes |  |
| `lng` | `number` | yes |  |
| `record_type_id` | `string | null` | no |  |

**Responses**

- **200** Successful Response - `AssessResponse`
- **422** Validation Error - `HTTPValidationError`

