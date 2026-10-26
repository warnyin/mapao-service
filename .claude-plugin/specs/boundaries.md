# Mapao - Boundaries (Thai admin areas, H3)

### `GET /api/v1/boundaries/datasets/active`

_Active Dataset_

**Auth:** required (cookie or Bearer)

**Responses**

- **200** Successful Response - `ActiveDataset`


### `GET /api/v1/boundaries/resolve`

_Resolve Point_

ADM0–3 units containing the point (deepest match, lowest P-code on a shared edge)
and its H3 cell. Levels are null outside every boundary or when no dataset is active.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `lat` | query | `number` | yes |  |
| `lng` | query | `number` | yes |  |
| `h3_res` | query | `integer` | no |  |

**Responses**

- **200** Successful Response - `ResolveResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/boundaries/{pcode}`

_Get Boundary_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `pcode` | path | `string` | yes |  |
| `children` | query | `boolean` | no |  |
| `geometry` | query | `boolean` | no | Include the simplified GeoJSON MultiPolygon |

**Responses**

- **200** Successful Response - `BoundaryUnit`
- **422** Validation Error - `HTTPValidationError`

