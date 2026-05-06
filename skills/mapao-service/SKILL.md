---
name: mapao-service
description: Use this skill when the user wants to build, integrate, or scaffold a frontend or mobile client that consumes the public Mapao geospatial API — covers auth, records, record-types, campaigns, attachments, shares, metadata, and /me. Triggers on requests like "connect to mapao", "add mapao login", "show mapao records on a map", "scaffold a mapao client", or any work targeting `mapao-api.warnyin.com`. Loads the endpoint catalog and resource schemas from `./specs/`, detects the user's stack (Next.js, Flutter, Swift, Kotlin, etc.), and writes client code that matches real fields and verbs — without reading OpenAPI or Swagger.
---

# Mapao — API Integration Skill

You are about to help the user build a frontend or mobile client that talks to the **public Mapao API** at `https://mapao-api.warnyin.com/api/v1`.

This skill ships its own snapshot of every public resource under `./specs/` (relative to this `SKILL.md`). Always read the spec file before writing code — do **not** invent endpoints, fields, or verbs.

## What Mapao is

A multi-tenant geospatial backend. Public resources exposed to external integrations:

| Resource | Purpose |
| --- | --- |
| `auth` | Login / refresh / logout (httpOnly cookie or Bearer) |
| `me` | PDPA self-service — export, delete, link external identity |
| `record-types` | Record-type catalog and dynamic field schemas |
| `records` | Geospatial records — CRUD, bbox/proximity queries, GeoJSON output |
| `campaigns` | Campaign metadata |
| `attachments` | File uploads attached to records |
| `metadata` | UI-generation helpers (form schemas, compatibility checks) |
| `shares` | Public share links — no-auth read via opaque token |

Admin endpoints (tenants, users, roles, permissions, audit) are intentionally **excluded** — those are managed via the Mapao admin console, not external integrators.

## Auth model — the one thing every integration must get right

- **Login:** `POST /api/v1/auth/login` with `{ email, password }`. Server returns `{ access_token, refresh_token, token_type }` AND sets `mapao_access_token` as an `HttpOnly; SameSite=Lax` cookie.
- **Browsers / webviews:** rely on the cookie — `credentials: 'include'` (`fetch`), `withCredentials: true` (`axios`), `dio.options.credentials` (Flutter web).
- **Native mobile (no cookie jar):** use `Authorization: Bearer <access_token>` from the response body.
- **Refresh:** `POST /api/v1/auth/refresh` with the refresh token. Server rotates and re-sets the cookie.
- **Logout:** `POST /api/v1/auth/logout`.
- **Lockout:** 10 consecutive failures → account locked 15 minutes (HTTP 429 with `Retry-After`).
- **Tenancy:** every resource is tenant-scoped from the authenticated session. Clients never pass a tenant id explicitly.

## Cross-origin gotcha

If the frontend runs on a **different origin** than `mapao-api.warnyin.com`, the cookie is bound to the API host. Either reverse-proxy `/api/*` through the frontend origin (recommended for dev), or rely on the `access_token` body value as a Bearer header. Without the cookie or Bearer, every authenticated request returns 401.

## How to use this skill

The user will arrive with one of these intents. Pick the matching workflow.

### Intent A — "What is Mapao?" / overview

Read `./specs/overview.md`, then summarize in 4–6 lines:

1. One-sentence purpose (multi-tenant geospatial backend).
2. The list of public resources.
3. The auth model in 2 sentences (cookie vs Bearer).
4. Mention follow-ups they can ask for: endpoint catalog, deep-dive on a resource, scaffold for their stack.

Do **not** dump the full spec — orient them; they'll ask for detail next.

### Intent B — "List endpoints" / catalog

Read `./specs/endpoints.md` and render a compact Markdown table grouped by resource (order: `auth`, `me`, `record-types`, `records`, `campaigns`, `attachments`, `metadata`, `shares`):

```
### auth
| Method | Path | Purpose |
| --- | --- | --- |
| POST | /api/v1/auth/login | … |
```

Omit any row that isn't in `endpoints.md`. End with a one-liner offering `<resource>` deep-dive or scaffold next.

### Intent C — Deep-dive a resource

Allowed values: `auth`, `me`, `record-types`, `records`, `campaigns`, `attachments`, `metadata`, `shares`. If the user named something else, list valid options and stop.

Read `./specs/<resource>.md`. For each endpoint in the file present:

1. **`METHOD /path`** as a header.
2. **Auth:** required scope or `public`.
3. **Path/query params:** name, type, required, default, description.
4. **Request body:** field-by-field (name, type, required, constraints).
5. **Responses:** status code → shape; include error codes (401, 403, 404, 409, 422, 429, 5xx) where the spec lists them.
6. **Example:** one minimal `curl` and one minimal JSON request/response — only if the spec contains them.

Do not invent fields. Use precise types (`string`, `uuid`, `integer >= 0`) — never paraphrase ("some text").

### Intent D — Scaffold client code

The most common path. Steps:

**1. Detect the stack** (priority order — check workspace root):

1. `pubspec.yaml` → Flutter/Dart
2. `package.json`:
   - `next` dependency → Next.js (App Router unless `pages/` exists)
   - `react-native` → React Native
   - `react` only → plain React (Vite/CRA)
   - `vue` → Vue · `svelte` → Svelte
   - none of the above → plain Node/TS
3. `*.xcodeproj`, `Package.swift`, `Podfile` → Swift / iOS
4. `build.gradle*` / `settings.gradle*` → Android (Kotlin preferred unless Java sources dominate)
5. `Cargo.toml`, `go.mod`, `requirements.txt` / `pyproject.toml`, `composer.json` → Rust / Go / Python / PHP

If multiple match (monorepo), ask which target. If none match, ask the user.

**2. Map the feature to resources.** Common mappings:

| User asks for | Resources |
| --- | --- |
| "records on a map" / "show flooded points" / "incidents nearby" | `records` (bbox or `lng`+`lat`+`radius_m`) + `record-types` for icons/colors |
| "login" / "sign in" | `auth` |
| "user profile" / "delete my data" | `me` + `auth` |
| "list campaigns" / "campaign details" | `campaigns` |
| "upload photo to a record" | `attachments` |
| "dynamic form" / "edit record fields" | `metadata` (form schema) + `records` |
| "share a record by link" | `shares` |

For each involved resource, read `./specs/<resource>.md`. Always also read `./specs/auth.md` if any endpoint requires auth.

**3. Plan, then write.** Before creating files, restate to the user:

1. Detected stack (e.g. "Flutter 3.x using Riverpod — saw `riverpod` in `pubspec.yaml`").
2. The Mapao endpoints you'll call.
3. Files to create or edit (paths + one-line purpose each).

Wait for confirm or course-correct unless the workspace is obviously empty (greenfield) and the answer is unambiguous.

**4. Write code that:**

- Uses the project's existing HTTP client / state management / routing (do not introduce a new HTTP library if `dio` / `axios` / `ktor` / etc. is already in use).
- Reads `MAPAO_API_BASE_URL` (or the project's existing config pattern) — never hard-code the host.
- Sends credentials so the auth cookie tags along on browser/webview targets, or accepts a Bearer token on native targets.
- Handles 401 (re-auth), 403 (insufficient permission), 404, 422 (validation errors with `detail[]`), 429 (`Retry-After`), and 5xx (transient — retry with backoff for idempotent verbs only).
- Includes one usage example as a comment at the top of the main generated file.

**5. After writing, in your reply:**

- List files created or edited.
- Show the one new dependency the user must install (if any).
- Show env vars the user must set (`MAPAO_API_BASE_URL` etc.).
- Suggest a quick smoke test (`curl` against `/api/v1/auth/login` plus one feature call).

Do not run tests or builds unless the user asks. Do not mock the network in generated code.

## Failure mode

If a spec file you need under `./specs/` is missing, tell the user the skill install is incomplete and stop. **Never invent endpoints to fill the gap.**

## Spec files in this skill

- `./specs/overview.md`
- `./specs/auth.md`
- `./specs/endpoints.md`
- `./specs/me.md`
- `./specs/record-types.md`
- `./specs/records.md`
- `./specs/campaigns.md`
- `./specs/attachments.md`
- `./specs/metadata.md`
- `./specs/shares.md`
