# Mapao - Records

### `GET /api/v1/records/`

_List Records_

List records as GeoJSON FeatureCollection.
Supports: type filter, campaign filter, bbox, proximity, JSONB attribute filter.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | query | `uuid | null` | no | Filter by record type |
| `campaign_id` | query | `uuid | null` | no | Filter by campaign |
| `status` | query | `string | null` | no |  |
| `bbox` | query | `string | null` | no | Bounding box: minLng,minLat,maxLng,maxLat |
| `lng` | query | `number | null` | no | Longitude for proximity search |
| `lat` | query | `number | null` | no | Latitude for proximity search |
| `radius_m` | query | `number | null` | no | Radius in meters for proximity search |
| `attr_filter` | query | `string | null` | no | JSONB filter, e.g. {"diesel_available":true} |
| `page` | query | `integer` | no |  |
| `page_size` | query | `integer` | no |  |

**Responses**

- **200** Successful Response - `GeoJSONFeatureCollection`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/records/`

_Create Record_

Create a new record with optional GeoJSON geometry and JSONB attributes.

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `record_type_id` | `uuid` | yes |  |
| `campaign_id` | `uuid | null` | no |  |
| `geom` | `object | null` | no | GeoJSON geometry |
| `status` | `string` | no |  |
| `attributes` | `object` | no | Dynamic JSONB data |
| `external_ref` | `string | null` | no |  |
| `event_date` | `datetime | null` | no |  |

**Responses**

- **201** Successful Response - `RecordResponse`
- **422** Validation Error - `HTTPValidationError`


### `DELETE /api/v1/records/{record_id}`

_Delete Record_

Soft-delete a record by setting status to 'deleted'.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `uuid` | yes |  |

**Responses**

- **204** Successful Response
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/records/{record_id}`

_Get Record_

Get a single record as GeoJSON Feature.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `uuid` | yes |  |

**Responses**

- **200** Successful Response - `GeoJSONFeature`
- **422** Validation Error - `HTTPValidationError`


### `PATCH /api/v1/records/{record_id}`

_Update Record_

Update a record's attributes, status, or geometry.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `uuid` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `geom` | `object | null` | no |  |
| `status` | `string | null` | no |  |
| `attributes` | `object | null` | no |  |
| `event_date` | `datetime | null` | no |  |

**Responses**

- **200** Successful Response - `RecordResponse`
- **422** Validation Error - `HTTPValidationError`

