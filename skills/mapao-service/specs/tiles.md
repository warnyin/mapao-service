# Mapao - Vector tiles (MVT)

### `GET /api/v1/tiles/boundaries/{level}/{z}/{x}/{y}.mvt`

_Boundaries Tile_

Admin boundaries of one level (layer ``boundaries``) from the active dataset.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `z` | path | `integer` | yes |  |
| `x` | path | `integer` | yes |  |
| `y` | path | `integer` | yes |  |
| `level` | path | `integer` | yes |  |

**Responses**

- **200** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/tiles/records/{z}/{x}/{y}.mvt`

_Records Tile_

The caller's tenant's records (layer ``records``); soft-deleted records are left out.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `z` | path | `integer` | yes |  |
| `x` | path | `integer` | yes |  |
| `y` | path | `integer` | yes |  |
| `record_type_id` | query | `string | null` | no |  |
| `status` | query | `string | null` | no |  |
| `admin` | query | `string | null` | no | Admin P-code prefix |

**Responses**

- **200** Successful Response
- **422** Validation Error - `HTTPValidationError`

