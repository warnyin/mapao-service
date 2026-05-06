# mapao-service

> **Build a frontend or mobile client for the public Mapao geospatial API in minutes.**
> Just say what you want — *"show flooded points on a map"*, *"add a login screen"*,
> *"upload a photo to a record"* — and Claude picks the right endpoints, detects your
> stack, and writes code that uses your project's existing conventions.
> No Swagger, no OpenAPI JSON, no copy-pasting curl examples.

[![skills.sh](https://img.shields.io/badge/skills.sh-warnyin/mapao--service-blue)](https://skills.sh/warnyin/mapao-service)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](./LICENSE)

## Try asking — concrete examples

Once installed (see [Install](#install)), drop any of these into a Claude Code
session. The skill auto-detects your stack and writes real code, not boilerplate.

| You ask | What gets generated |
| --- | --- |
| `Show flooded points on a map of Bangkok` | Map view + `records?bbox=...` query + `record-types` for icon/color, in your stack |
| `Add a login screen` | Email/password form, login action, cookie/Bearer handling, 401 / 422 / 429 (lockout) error states |
| `Upload a photo when I report an incident` | File picker → `POST /records/{id}/attachments` flow with progress bar + retry |
| `Show records I created, paginated` | List view with `records?created_by_me=true&limit=20`, infinite-scroll wired to your state lib |
| `Make a "Share" button that gives a public link` | "Share" UI → `POST /records/{id}/share` → opaque-token URL → public viewer route at `/public/r/{token}` |
| `Build a dynamic edit form for a record-type` | Form that reads `metadata` schema and adapts fields (text, number, select, geometry) |
| `Add "Export my data" + "Delete my account" (PDPA)` | Two flows from `/me/export` + `DELETE /me`, with confirmation dialogs |
| `List campaigns I'm part of` | Campaign list + detail view with paging |

Generated code uses your existing HTTP client (`dio` / `axios` / `ktor` / etc.),
state management (Riverpod, Zustand, Redux, SwiftUI `@Observable`, …), and
routing. It reads `MAPAO_API_BASE_URL` from env so dev/staging swap stays one
config change.

## What's covered

The full **public** API surface — everything an external integrator needs:

- `auth` — login / refresh / logout (httpOnly cookie or Bearer)
- `me` — PDPA self-service (export, delete, external identity link)
- `record-types` — record-type catalog and dynamic schemas
- `records` — geospatial records (CRUD, bbox/proximity queries, GeoJSON)
- `campaigns` — campaign metadata
- `attachments` — record file uploads
- `metadata` — UI generation helpers (form schemas, compatibility checks)
- `shares` — public share links (no-auth read via opaque token)

This is the entire surface area: there are no admin or platform-management
endpoints here, by design. The plugin is built for external integrators
building independent clients on top of Mapao as a backend-as-a-service.

## Base URL

Production: **`https://mapao-api.warnyin.com`**

All endpoints below are relative to this base + `/api/v1`. Clients should
read the host from a config/env var (`MAPAO_API_BASE_URL`) so dev/staging
swaps stay one config change away.

## Install

Via [skills.sh](https://skills.sh):

```bash
npx skills add warnyin/mapao-service
```

Or as a Claude Code plugin directly:

```bash
# from inside Claude Code
/plugin install warnyin/mapao-service
```

## Slash commands

| Command | Purpose |
| --- | --- |
| `/mapao-overview` | One-screen pitch: what Mapao is, public resources, auth model |
| `/mapao-auth` | Login flow, cookie vs Bearer, how to wire credentials in any HTTP client |
| `/mapao-endpoints` | Compact catalog of every public endpoint (method + path + summary) |
| `/mapao-spec <resource>` | Deep dive on one resource — params, request/response schema, error codes |
| `/mapao-scaffold <feature>` | Auto-detects your stack (Flutter, Next.js, Swift, Kotlin, …) and generates client code for a use case |

## Typical flow

The skill always **plans before writing**: it tells you the stack it detected,
the endpoints it'll call, and the files it'll create. You confirm or
course-correct, then it writes the code.

### Example 1 — Flutter app, flood incidents on a map

```text
You:    Show flood incident records on a map of Bangkok

Claude: [reads specs/records.md + record-types.md + auth.md]

        Detected stack: Flutter 3.x with Riverpod (saw pubspec.yaml).
        Endpoints I'll call:
          GET  /api/v1/records/?bbox=100.3,13.5,100.9,14.0&record_type_slug=flood
          GET  /api/v1/record-types/

        Files I'll create:
          lib/mapao/api_client.dart           — Dio client + cookie/Bearer handling
          lib/mapao/records_repository.dart   — bbox query + GeoJSON parsing
          lib/features/map/flood_map_view.dart — flutter_map widget with markers

        Env: MAPAO_API_BASE_URL (default https://mapao-api.warnyin.com)
        New dep: flutter_map ^7.0.0  (already had dio + riverpod)

        OK to write?
```

### Example 2 — Next.js App Router, login + protected dashboard

```text
You:    Add Mapao login and a protected dashboard

Claude: [reads specs/auth.md + me.md]

        Detected stack: Next.js 15 App Router + TypeScript (saw next.config.ts).
        Endpoints I'll call:
          POST /api/v1/auth/login
          POST /api/v1/auth/refresh
          POST /api/v1/auth/logout
          GET  /api/v1/auth/me

        Files I'll create:
          src/lib/mapao/client.ts            — fetch wrapper with credentials: 'include'
          src/app/login/page.tsx             — login form (server action)
          src/app/login/actions.ts           — login server action with redirect
          src/middleware.ts                  — gate /dashboard routes by cookie
          src/app/dashboard/page.tsx         — protected page reading /auth/me
          src/app/api/auth/refresh/route.ts  — proxy to API (same-origin cookie path)

        OK to write?
```

### Example 3 — SwiftUI iOS app, list of nearby records

```text
You:    Show Mapao records within 5km of the user's location

Claude: [reads specs/records.md + auth.md]

        Detected stack: SwiftUI + Swift 5.10 (saw *.xcodeproj, no AppDelegate).
        Endpoints I'll call:
          GET  /api/v1/records/?lng=...&lat=...&radius_m=5000

        Files I'll create:
          MapaoApp/MapaoClient.swift          — async/await client with Bearer token
          MapaoApp/RecordsRepository.swift    — proximity query
          MapaoApp/Views/NearbyRecordsView.swift — List view with CoreLocation prompt

        Permissions to add to Info.plist: NSLocationWhenInUseUsageDescription

        OK to write?
```

## How the plugin stays accurate

Specs are regenerated **automatically** from the live FastAPI app — there's
no human in the loop maintaining them.

```
backend change in main Mapao repo (FastAPI router/schema)
   ↓ push
sync-public-api-spec workflow                       ← generates markdown from app.openapi()
   ↓ cross-repo push
.claude-plugin/specs/  in this repo                 ← canonical snapshot
   ↓ trigger
sync-skill-specs workflow                           ← mirrors to skill copy
   ↓ commit
skills/mapao-service/specs/  in this repo           ← what `npx skills add` installs
```

A drift check on every PR fails CI if the two spec directories diverge, so
the skill copy can never silently fall behind the plugin copy. This means
every commit to `main` is a fresh, point-in-time snapshot of
`https://mapao-api.warnyin.com/api/v1`.

## License

MIT — see [LICENSE](./LICENSE).
