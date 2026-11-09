# Mapao - Boundaries (Thai admin areas, H3)

### `POST /api/v1/address/parse`

_Parse Address_

Admin-unit candidates for a Thai address, best first. `admin` is set only when `confidence` ≥ 0.6; below that,
show `candidates` and let the user choose. The postcode is extracted into `parts` but never used for matching.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Request body** (`application/json`)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `text` | `string` | yes | One address line, Thai or English (up to 300 characters) |
| `within` | `string \| null` | no | Only units under this P-code, e.g. TH10 |
| `limit` | `integer` | no | Candidates to return |

**Responses**

- **200** Successful Response - `ParseResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `admin` | `Admin \| null` | yes | The best candidate's units, only when `confidence` ≥ 0.6; else null |
| `candidates` | `list[Candidate]` | yes | Up to `limit` candidates whose levels nest, best first |
| `parts` | `object` | yes | Only the parts found, as written; a part that is not in the text is omitted, never null |
| `residual` | `string` | yes | Words not used by the best candidate, admin words left out; "" when everything was read. A place name here was not used: misspelt, too short, or it disagrees with the other levels |
| `confidence` | `number` | yes | 0-1: the best score, scaled down when a rival candidate (not its own parent) scores within 0.30 of it; equal rivals give 0 |
| `dataset_version` | `string` | yes | The active boundary dataset these units come from |
| `attribution` | `string` | yes | Show it wherever you display the units |

**`Admin`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `adm1` | `Unit \| null` | yes | Province |
| `adm2` | `Unit \| null` | yes | District; null when the text named a province only |
| `adm3` | `Unit \| null` | yes | Tambon; null when the text named no tambon that fits (ask for it if needed) |

**`Candidate`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `adm1` | `Unit \| null` | yes | Province |
| `adm2` | `Unit \| null` | yes | District (Bangkok: เขต) |
| `adm3` | `Unit \| null` | yes | Tambon (Bangkok: แขวง); null when the text named none this candidate uses |
| `score` | `number` | yes | 0-1: about 0.30 per level read from the text, more when its admin word agrees, less when the word names another level or the match is fuzzy; +0.10 for a tambon |
| `matched` | `list[Matched]` | yes | One entry per level the candidate has, province first |

**`Unit`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pcode` | `string` | yes |  |
| `name_th` | `string` | yes |  |
| `name_en` | `string` | yes |  |

**`Matched`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `level` | `integer` | yes | 1 province, 2 district, 3 tambon |
| `span` | `array \| null` | yes | [start, end) character range in your text; null when inferred |
| `text` | `string \| null` | yes | The words of your text read as this level; null when inferred |
| `inferred` | `boolean` | yes | True when this level came from the P-code of a deeper level, not your text |
| `fuzzy` | `boolean` | yes | True for a guess at a misspelt Latin name: confirm it with the user |


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


### `GET /api/v1/boundaries/search`

_Search Boundaries_

Provinces, districts and tambons of the active dataset whose name contains ``q``, best first
(exact, prefix, contains, then fuzzy for Latin text), each with its parent chain. For typeahead:
debounce at least 250 ms and send from 2 characters; a 304 (If-None-Match) costs no quota.

**Auth:** API key (`X-API-Key: sk_…` or `Authorization: Bearer sk_…`) or a user session

**Parameters**

| Name | In | Type | Required | Description |
| --- | --- | --- | --- | --- |
| `q` | query | `string` | yes | Thai or English name; the admin word (เขต, Amphoe ...), spaces and case are ignored |
| `level` | query | `integer \| null` | no | 1 province, 2 district, 3 tambon |
| `within` | query | `string \| null` | no | Only units under this P-code, e.g. TH10 |
| `limit` | query | `integer` | no |  |

**Responses**

- **200** Successful Response - `SearchResponse`
- **422** Validation Error - `HTTPValidationError`

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `items` | `list[SearchItem]` | yes |  |
| `truncated` | `boolean` | yes |  |
| `dataset_version` | `string` | yes |  |
| `attribution` | `string` | yes |  |

**`SearchItem`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pcode` | `string` | yes |  |
| `name_th` | `string` | yes |  |
| `name_en` | `string` | yes |  |
| `level` | `integer` | yes |  |
| `parents` | `list[AdminUnit]` | yes |  |
| `match` | `string` | yes |  |

**`AdminUnit`**

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `pcode` | `string` | yes |  |
| `name_th` | `string` | yes |  |
| `name_en` | `string` | yes |  |
| `level` | `integer` | yes |  |


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

