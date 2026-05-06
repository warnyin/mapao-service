# Mapao - Campaigns

### `GET /api/v1/campaigns/`

_List Campaigns_

List campaigns (RLS-filtered). Optional filter by record type or status.

Pagination is mandatory — the previous unpaged version would OOM the
server on tenants with hundreds of campaigns. Default page_size=20,
hard cap 100 per request.

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `record_type_id` | query | `uuid | null` | no |  |
| `status` | query | `string | null` | no |  |
| `page` | query | `integer` | no | 1-indexed page number |
| `page_size` | query | `integer` | no | Hard cap 100 prevents OOM |

**Responses**

- **200** Successful Response - `array<CampaignResponse>`
- **422** Validation Error - `HTTPValidationError`


### `POST /api/v1/campaigns/`

_Create Campaign_

Create a new campaign (e.g. 'ไฟป่าภาคเหนือ 2026').

**Auth:** required (cookie or Bearer)

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `record_type_id` | `uuid` | yes |  |
| `name` | `string` | yes |  |
| `slug` | `string` | yes |  |
| `description` | `string | null` | no |  |
| `start_date` | `datetime | null` | no |  |
| `end_date` | `datetime | null` | no |  |
| `settings` | `object` | no |  |

**Responses**

- **201** Successful Response - `CampaignResponse`
- **422** Validation Error - `HTTPValidationError`


### `GET /api/v1/campaigns/{campaign_id}`

_Get Campaign_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `campaign_id` | path | `uuid` | yes |  |

**Responses**

- **200** Successful Response - `CampaignResponse`
- **422** Validation Error - `HTTPValidationError`


### `PATCH /api/v1/campaigns/{campaign_id}`

_Update Campaign_

**Auth:** required (cookie or Bearer)

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `campaign_id` | path | `uuid` | yes |  |

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `name` | `string | null` | no |  |
| `description` | `string | null` | no |  |
| `start_date` | `datetime | null` | no |  |
| `end_date` | `datetime | null` | no |  |
| `status` | `string | null` | no |  |
| `settings` | `object | null` | no |  |

**Responses**

- **200** Successful Response - `CampaignResponse`
- **422** Validation Error - `HTTPValidationError`

