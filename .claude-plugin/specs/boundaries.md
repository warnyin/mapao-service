# Mapao - Boundaries (Thai admin areas, H3)

### `GET /api/v1/boundaries/datasets/active`

_Active Dataset_

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Responses**

- **200** Successful Response - `ActiveDataset`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `id` | `integer` | yes |  |
| `version` | `string` | yes |  |
| `source` | `string` | yes |  |
| `counts` | `object` | yes |  |
| `attribution` | `string` | yes |  |
| `activated_at` | `string \| null` | yes |  |


### `GET /api/v1/boundaries/resolve`

_Resolve Point_

ADM0–3 units containing the point (deepest match, lowest P-code on a shared edge)
and its H3 cell. Levels are null outside every boundary or when no dataset is active.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `lat` | query | `number` | yes |  |
| `lng` | query | `number` | yes |  |
| `h3_res` | query | `integer` | no |  |

**Responses**

- **200** Successful Response - `ResolveResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `adm0` | `AdminName \| null` | yes |  |
| `adm1` | `AdminName \| null` | yes |  |
| `adm2` | `AdminName \| null` | yes |  |
| `adm3` | `AdminName \| null` | yes |  |
| `h3` | `string` | yes |  |
| `h3_res` | `integer` | yes |  |
| `dataset_version` | `string \| null` | yes |  |
| `attribution` | `string` | yes |  |

**`AdminName`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pcode` | `string` | yes |  |
| `name_th` | `string` | yes |  |
| `name_en` | `string` | yes |  |


### `GET /api/v1/boundaries/{pcode}`

_Get Boundary_

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `pcode` | path | `string` | yes |  |
| `children` | query | `boolean` | no |  |
| `geometry` | query | `boolean` | no | Include the simplified GeoJSON MultiPolygon |

**Responses**

- **200** Successful Response - `BoundaryUnit`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pcode` | `string` | yes |  |
| `name_th` | `string` | yes |  |
| `name_en` | `string` | yes |  |
| `level` | `integer` | yes |  |
| `parents` | `list[AdminUnit]` | yes |  |
| `children` | `array \| null` | no |  |
| `geometry` | `object \| null` | no |  |
| `dataset_version` | `string` | yes |  |
| `attribution` | `string` | yes |  |

**`AdminUnit`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pcode` | `string` | yes |  |
| `name_th` | `string` | yes |  |
| `name_en` | `string` | yes |  |
| `level` | `integer` | yes |  |

