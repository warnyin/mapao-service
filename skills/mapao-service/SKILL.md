---
name: mapao-service
description: Build apps on the public Mapao geospatial API - frontend (Next.js, React, Vue), mobile (Flutter, React Native, Swift, Kotlin) and server/back-end apps (Node, Python, Go, jobs, webhooks). Covers records and record types, Thai administrative area resolve (province/district/tambon), GPS location trust and anti-spoof checks for check-ins, vector tiles for maps, campaigns, attachments, shares and PDPA. Say what you want - "tambon check-in game with spoof rejection", "map of claimed tambons", "show flooded points on a map", "add a login screen" - and this skill picks the endpoints, detects your stack and writes code with your project's conventions. The endpoint catalog and resource schemas ship with the skill.
---

# Mapao - API Integration Skill

This skill helps people build apps on the **public Mapao API** at `https://mapao-api.warnyin.com/api/v1`. It ships a snapshot of every public resource under `./specs/` (relative to this `SKILL.md`). Always read the spec file before writing code. Do **not** invent endpoints, fields, or verbs.

Always-current reference (agent-oriented, newer than this snapshot if they disagree): `https://mapao-api.warnyin.com/llms.txt`.

## What Mapao is

A multi-tenant geospatial backend. Each tenant has record types (collections with JSON schemas) holding GeoJSON records in WGS84, plus built-in Thai admin boundaries (ADM0-3), GPS trust grading and vector tiles.

| Resource | Purpose |
| --- | --- |
| `auth` | Login / refresh / logout for end users (httpOnly cookie or Bearer) |
| `me` | PDPA self-service: export, delete, link external identity |
| `record-types` | Collection catalog and dynamic field schemas |
| `records` | Geospatial records: CRUD, bbox/proximity/intersects/clusters, GeoJSON |
| `boundaries` | Resolve a lat/lng to country, province, district, tambon (+ H3 cell); look up one unit |
| `location` | Grade a GPS fix (spoof / low accuracy / impossible travel) without storing it; erase an actor |
| `tiles` | Vector tiles (MVT) of records and admin boundaries for MapLibre / Mapbox / OpenLayers |
| `campaigns` | Campaign metadata |
| `attachments` | File uploads attached to records |
| `metadata` | UI-generation helpers (form schemas, compatibility checks) |
| `shares` | Public share links: no-auth read via opaque token |

Tenant, user, role and audit administration is not part of this skill. Don't scaffold against it.

## Auth - read this first

There are two ways to authenticate. Pick by who the caller is.

**1. API key (apps, servers, jobs, anything without an end-user login)**

1. Sign up and log in at `/console`, create an app, then create a key for it and choose its scopes (narrowest that works). The full key is shown once.
2. Put it in the server environment as `MAPAO_API_KEY` (never in source) and send it on every request:

```http
X-API-Key: sk_...
```

`Authorization: Bearer sk_...` also works; if both are sent, `X-API-Key` wins. In code that reads the env var: `X-API-Key: $MAPAO_API_KEY` (a value starting with `sk_`).

- Key types: `sk_` is a secret key, server-side only, with an explicit scope list. Never put `sk_` keys in browser code, mobile apps, bundles, URLs, logs or commits.
- `pk_` is a publishable key: read-only, bound to exact `allowed_origins` (scheme, host, port), made for browsers and mobile. A `pk_` key gets 403 on `/location/assess` and on every write.
- A key belongs to one app, so one tenant; it only sees that tenant's data. Clients never pass a tenant id.
- A key cannot create keys, apps or record types, and cannot call admin routes.
- Record types (collections) are created by a tenant admin in `/console`; a key cannot create them, so list them with `GET /api/v1/record-types/` and ask the user for the collection if none fits.
- Typical scopes: `records:list`, `records:read`, `records:create`, `record_types:read`, `location:erase`. `POST /api/v1/location/assess` needs `records:list`. Report a missing scope (403 `KEY_SCOPE_DENIED`) to the developer; don't look for another route.

**2. End-user login (apps where people sign in)**

- **Login:** `POST /api/v1/auth/login` with `{ email, password }`. The server returns `{ access_token, refresh_token, token_type }` AND sets `mapao_access_token` as an `HttpOnly; SameSite=Lax` cookie.
- **Browsers / webviews:** rely on the cookie: `credentials: 'include'` (`fetch`), `withCredentials: true` (`axios`), `dio.options.credentials` (Flutter web).
- **Native mobile (no cookie jar):** `Authorization: Bearer <access_token>` from the response body.
- **Refresh:** `POST /api/v1/auth/refresh` with the refresh token. **Logout:** `POST /api/v1/auth/logout`.
- **Lockout:** 10 consecutive failures lock the account 15 minutes (HTTP 429 with `Retry-After`).
- **Cross-origin:** the cookie is bound to the API host. If the frontend is on a different origin, reverse-proxy `/api/*` through the frontend origin, or use the Bearer token.

## Errors and limits

Envelope: `{"error": "<message>", "detail": {"code": "<CODE>"}}`. Handle:

- 401 `KEY_INVALID` / `KEY_REVOKED`: stop, ask for a new key. Don't retry.
- 403 `KEY_SCOPE_DENIED` (scope missing, `pk_` write, route not allowed for keys), `ORIGIN_NOT_ALLOWED`.
- 422: fix the request (`detail[]`). 404 unknown ids or record types.
- 429 `RATE_LIMITED`: wait `Retry-After`, retry. 429 `QUOTA_EXCEEDED`: stop and report `detail.resets_at`; don't loop.
- 5xx: retry with backoff for idempotent calls only.

## Recipe - server-side check-in flow (spoof rejection + map)

Use for "check in at a tambon", "claim a place", field reports. All calls happen on your server with the `sk_` key; the browser or app talks only to your own routes.

1. **Collection:** `GET /api/v1/record-types/` and pick the `id` (a tenant admin creates it in `/console`). `GET /api/v1/record-types/{id}/schemas/current` gives the required `attributes`. Never invent an id.
2. **Assess** (trust + admin area in one call, stores nothing). The device sends `lat`, `lng`, `accuracy_m`, `captured_at` (with timezone offset), `mock_reported` and your opaque player/device id; your server forwards:

```http
POST /api/v1/location/assess
X-API-Key: sk_...
Content-Type: application/json

{"lat": 13.7563, "lng": 100.5018, "accuracy_m": 12, "captured_at": "2026-01-01T10:00:00+07:00",
 "mock_reported": false, "actor_ref": "player-123", "record_type_id": "<uuid>"}
```

   Returns `{trust, admin}`. `trust` is `{flags, score, version, checks}`; the `admin` object in the assess response has the same shape as the resolve response (`adm0`..`adm3` each `{pcode, name_th, name_en}` or null, `h3`, `dataset_version`).
3. **Decide in your app.** Mapao grades, it never rejects. Typical policy: refuse when `trust.flags` contains `IMPOSSIBLE_TRAVEL`, `MOCK_REPORTED` or `FUTURE_FIX`, or when `trust.score` is above your threshold; ask for a retry on `LOW_ACCURACY` / `STALE_FIX`; treat `NEAR_BOUNDARY` as ambiguous. Use `admin.adm3.pcode` as the claimed tambon. Flags and score weights: `IMPOSSIBLE_TRAVEL` 50, `MOCK_REPORTED` 40, `FUTURE_FIX` 40, `LOW_ACCURACY` 30, `NEAR_BOUNDARY` 20, `STALE_FIX` 20 (score = sum, capped at 100). Default limits: accuracy 50 m, speed 250 km/h, fix age 120 s, future tolerance 30 s; each `trust.checks.<name>` shows its `status`, `value` and `limit`. A score of 50 or more means at least one strong flag, a sensible reject threshold.
   `IMPOSSIBLE_TRAVEL` compares the fix with the same `actor_ref`'s previous and next fixes **stored as records**; assess stores nothing, so step 4 (saving the claim with `location.actor_ref`) is what makes the next check-in comparable. Use one stable `actor_ref` per player.
4. **Store the claim:** `POST /api/v1/records/` with `record_type_id`, a Point `geom` (`[lng, lat]`), `attributes` per the schema, and the same `location` object (`{accuracy_m, captured_at, mock_reported, actor_ref}`). Send an `Idempotency-Key` header (1-255 printable ASCII, e.g. a UUID per claim) so retries return the first result (`Idempotent-Replayed: true`). The create response carries `admin_pcode` (deepest P-code, e.g. `TH100107`) and `trust` from the server (client values are ignored); in GeoJSON lists the same data is `properties.admin` (`{adm1, adm2, adm3}`) and `properties.trust`.
5. **Read claims:** `GET /api/v1/records/?record_type_id=...&admin=TH10` (`admin` is a P-code prefix: province `TH10`, district `TH1001`). Results are GeoJSON FeatureCollections; `page_size` max 100.
6. **Map:** use vector tiles, not paged GeoJSON. `GET /api/v1/tiles/records/{z}/{x}/{y}.mvt` (layer `records`, properties `id`, `record_type_id`, `status`, `version`, `admin_pcode`, `trust_score`) and `GET /api/v1/tiles/boundaries/{level}/{z}/{x}/{y}.mvt` (level 0-3, layer `boundaries`, properties `pcode`, `level`, `name_th`, `name_en`). Tiles need the key, so proxy them through your server (for example a Next.js route handler that adds `X-API-Key` and streams the body with `Content-Type: application/vnd.mapbox-vector-tile`), or use a `pk_` key from an allowed origin in the browser. Keep the `ETag` and send `If-None-Match`; a 304 does not count against quota. Point MapLibre `tiles` at your proxy path.

Resolve without recording: `GET /api/v1/boundaries/resolve?lat=&lng=` returns `adm0`..`adm3`, `h3`, `h3_res`, `dataset_version`, `attribution`. Show "OCHA/HDX cod-ab-tha, CC BY-IGO" wherever boundary data is displayed.

Conventions: GeoJSON is `[lng, lat]`; `bbox` is `minLng,minLat,maxLng,maxLat`; `radius_m` and `eps_m` are meters. Use `If-Match` with the record `ETag` / `properties.version` on `PATCH` and `DELETE` to avoid lost updates (412 means re-read, merge, retry). Attestation (Play Integrity, App Attest) is verified by your backend and sent as `mock_reported`.

## How to use this skill

The user arrives with one of these intents. Pick the matching workflow.

### Intent A - "What is Mapao?" / overview

Read `./specs/overview.md`, then summarize in 4-6 lines: purpose, public resources, the auth model in 2 sentences (API key for servers, cookie or Bearer for logged-in users), and follow-ups they can ask for (endpoint catalog, resource deep-dive, scaffold for their stack). Do not dump the spec.

### Intent B - "List endpoints" / catalog

Read `./specs/endpoints.md` and render a compact Markdown table grouped by resource (order: `auth`, `me`, `record-types`, `records`, `boundaries`, `location`, `tiles`, `campaigns`, `attachments`, `metadata`, `shares`) with Method, Path, Purpose. Omit any row that isn't in `endpoints.md`. End with an offer for a deep-dive or scaffold.

### Intent C - Deep-dive a resource

Allowed values: `auth`, `me`, `record-types`, `records`, `boundaries`, `location`, `tiles`, `campaigns`, `attachments`, `metadata`, `shares`. If the user named something else, say which of these is closest and offer it.

Read `./specs/<resource>.md`. For each endpoint present: the `METHOD /path` header; auth requirement; path/query params; request body field by field; responses (status code to shape, including error codes the spec lists); one minimal example only if the spec has it. Use precise types (`string`, `uuid`, `integer >= 0`). Do not invent fields.

### Intent D - Scaffold code

1. **Detect the stack** from the workspace root: `pubspec.yaml` Flutter; `package.json` with `next` (Next.js), `react-native`, `react`, `vue`, `svelte`, else plain Node/TS; `*.xcodeproj` / `Package.swift` Swift; `build.gradle*` Android; `Cargo.toml`, `go.mod`, `pyproject.toml`, `composer.json` Rust / Go / Python / PHP. If several match, ask which target; if none, ask.
2. **Decide where the key lives.** If the code runs in a browser or app, either use a `pk_` key (read-only) or add a server route that holds the `sk_` key and call that. Writes and `/location/assess` always go through a server.
3. **Map the feature to resources:**

| User asks for | Resources |
| --- | --- |
| "records on a map" / "incidents nearby" | `tiles` (or `records` with bbox or `lng`+`lat`+`radius_m`) + `record-types` for icons |
| "which province/district/tambon is this point in" | `boundaries` (resolve) |
| "check-in", "anti-spoof", "is this GPS real", "claim a tambon" | `location` (assess) + `records` (create with `location`) + `boundaries` |
| "map of claimed areas", "boundary overlay" | `tiles` (records + boundaries) |
| "login" / "sign in" | `auth` |
| "user profile" / "delete my data" | `me` + `auth` |
| "list campaigns" | `campaigns` |
| "upload photo to a record" | `attachments` |
| "dynamic form" / "edit record fields" | `metadata` + `records` |
| "share a record by link" | `shares` |

   Read `./specs/<resource>.md` for each, and `./specs/auth.md` for any authenticated endpoint.
4. **Plan, then write.** Restate the detected stack, the endpoints you'll call, and the files to create or edit; wait for confirmation unless the workspace is clearly empty and the answer unambiguous.
5. **Write code that** uses the project's existing HTTP client, state management and routing; reads `MAPAO_API_BASE_URL` (default `https://mapao-api.warnyin.com`) and `MAPAO_API_KEY` from env, never hard-coded; handles the errors above; and has one usage example in a comment at the top of the main file.
6. **Reply with** files created or edited, the one new dependency if any, env vars to set (`MAPAO_API_BASE_URL`, `MAPAO_API_KEY`), and a smoke test (a `curl` with `X-API-Key` against `/api/v1/record-types/` or `/api/v1/boundaries/resolve`).

Do not run tests or builds unless asked. Do not mock the network in generated code.

## Failure mode

If a spec file you need under `./specs/` is missing, tell the user the skill install is incomplete and fall back to `https://mapao-api.warnyin.com/llms.txt`. **Never invent endpoints to fill the gap.**

## Spec files in this skill

- `./specs/overview.md`
- `./specs/endpoints.md`
- `./specs/auth.md`
- `./specs/me.md`
- `./specs/record-types.md`
- `./specs/records.md`
- `./specs/boundaries.md`
- `./specs/location.md`
- `./specs/tiles.md`
- `./specs/campaigns.md`
- `./specs/attachments.md`
- `./specs/metadata.md`
- `./specs/shares.md`
