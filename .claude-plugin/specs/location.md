# Mapao - Location trust (GPS evidence, assess)

### `POST /api/v1/location/actors/erase`

_Erase Actor_

Forget an actor: clear its keyed hash on every record of this tenant (RLS-scoped),
so its travel history is no longer linked. Records themselves are kept.

Secret (`sk_`) keys with `location:erase` only; publishable keys get 403.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `actor_ref` | `string` | yes |  |

**Responses**

- **200** Successful Response - `EraseResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `cleared` | `integer` | yes |  |


### `POST /api/v1/location/assess`

_Assess_

Grade a fix exactly as record creation would; stores nothing.

Secret (`sk_`) keys only; publishable keys get 403. Sending `actor_ref` also needs `records:create`.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `accuracy_m` | `number` | yes |  |
| `captured_at` | `string \| null` | no |  |
| `mock_reported` | `boolean` | no |  |
| `actor_ref` | `string \| null` | no |  |
| `lat` | `number` | yes |  |
| `lng` | `number` | yes |  |
| `record_type_id` | `string \| null` | no |  |

**Responses**

- **200** Successful Response - `AssessResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `trust` | `Trust` | yes |  |
| `admin` | `object` | yes | Same object as GET /api/v1/boundaries/resolve: adm0..adm3 (each {pcode, name_th, name_en} or null), h3, h3_res, dataset_version, attribution |

**`Trust`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `flags` | `list[string]` | yes | Raised flags, e.g. LOW_ACCURACY, MOCK_REPORTED, IMPOSSIBLE_TRAVEL |
| `score` | `integer` | yes | Sum of the raised flags' weights, capped at 100: 0 = no flag, higher = more suspicious |
| `version` | `string` | yes |  |
| `checks` | `object` | yes | One entry per check: accuracy, mock, stale, future, boundary, travel |

