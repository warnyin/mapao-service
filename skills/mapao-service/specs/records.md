# Mapao - Records

### `GET /api/v1/records/`

_List Records_

List records as GeoJSON FeatureCollection.
Supports: type filter, campaign filter, bbox, proximity, JSONB attribute filter.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | query | `string | null` | no | Filter by record type |
| `campaign_id` | query | `string | null` | no | Filter by campaign |
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

``Idempotency-Key`` makes retries create at most one record: the same key and body
within 24 h replays the stored 201 (``Idempotent-Replayed: true``).

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `record_type_id` | `string` | yes |  |
| `campaign_id` | `string | null` | no |  |
| `geom` | `object | null` | no | GeoJSON geometry |
| `status` | `string` | no |  |
| `attributes` | `object` | no | Dynamic JSONB data |
| `external_ref` | `string | null` | no |  |
| `event_date` | `string | null` | no |  |

**Responses**

- **201** Successful Response - `RecordResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/records/clusters`

_Record Clusters_

DBSCAN clusters of records inside a bbox (tenant-scoped via RLS).

Like GET /records/, every status is included unless ``status`` is given.
At most 50000 newest records are clustered; at most 1000 clusters returned.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `min_lng` | query | `number` | yes |  |
| `min_lat` | query | `number` | yes |  |
| `max_lng` | query | `number` | yes |  |
| `max_lat` | query | `number` | yes |  |
| `eps_m` | query | `number` | no | Cluster distance, meters (eps_m / 111000 degrees) |
| `record_type_id` | query | `string | null` | no |  |
| `status` | query | `string | null` | no |  |

**Responses**

- **200** Successful Response - `ClusterResponse`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/records/intersects`

_Records Intersecting_

Records whose geometry intersects the given GeoJSON geometry.

Like GET /records/, every status is included unless ``status`` is given.
``total_count`` is the number returned; ``truncated`` means ``limit`` was hit.

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `geometry` | `object` | yes |  |
| `record_type_id` | `string | null` | no |  |
| `status` | `string | null` | no |  |
| `limit` | `integer` | no |  |

**Responses**

- **200** Successful Response - `IntersectsResponse`
- **422** Validation Error - `HTTPValidationError`


### `DELETE /api/v1/records/{record_id}`

_Delete Record_

Soft-delete a record by setting status to 'deleted' (honours ``If-Match``).

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |

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
| `record_id` | path | `string` | yes |  |

**Responses**

- **200** Successful Response - `GeoJSONFeature`
- **422** Validation Error - `HTTPValidationError`


### `PATCH /api/v1/records/{record_id}`

_Update Record_

Update a record's attributes, status, or geometry.

Optional ``If-Match`` (``"n"``, a comma list of up to 20, or ``*``) makes the update
conditional on the current ``version``; a mismatch is 412 ``PRECONDITION_FAILED``.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |
| `upgrade_schema` | query | `boolean` | no | When true, validate attributes against the *current* schema version and re-pin the record to it. When false (default), validate against the schema version this record was last saved under — keeps old records editable after a breaking schema change. |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `geom` | `object | null` | no |  |
| `status` | `string | null` | no |  |
| `attributes` | `object | null` | no |  |
| `event_date` | `string | null` | no |  |

**Responses**

- **200** Successful Response - `RecordResponse`
- **422** Validation Error - `HTTPValidationError`


### `PATCH /api/v1/records/{record_id}/status`

_Update Record Status_

Status-only update for moderators / triage workflows.

Distinct from PATCH /records/{id}:
  - gated on ``records:update_status`` permission (held by ``moderator``,
    ``editor``, ``tenant_admin``)
  - body accepts only ``status`` and optional ``reason``
  - bypasses ownership: moderators may act on any record in their tenant
  - ``reason`` is captured in the audit-log changes payload

Members / viewers are not granted the permission and will 403.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_id` | path | `string` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `status` | `string` | yes |  |
| `reason` | `string | null` | no |  |

**Responses**

- **200** Successful Response - `RecordResponse`
- **422** Validation Error - `HTTPValidationError`

