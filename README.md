# mapao-service

Claude Code plugin for consuming the public **Mapao** API. Loads the endpoint
catalog, schemas, and auth flow into your Claude Code session, then scaffolds
frontend or mobile clients without ever opening Swagger or the OpenAPI JSON.

## What is Mapao?

Mapao is a multi-tenant geospatial backend platform. Public resources exposed
to external integrations:

- `auth` — login / refresh / logout (httpOnly cookie or Bearer)
- `me` — PDPA self-service (export, delete, external identity link)
- `record-types` — record-type catalog and dynamic schemas
- `records` — geospatial records (CRUD, bbox/proximity queries, GeoJSON)
- `campaigns` — campaign metadata
- `attachments` — record file uploads
- `metadata` — UI generation helpers (form schemas, compatibility checks)
- `shares` — public share links (no-auth read via opaque token)

Admin endpoints (tenants, users, roles, permissions, audit) are intentionally
excluded — those are managed via the Mapao admin console, not by external
integrators.

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

```text
You:    /mapao-scaffold records on a map
Claude: [reads .claude-plugin/specs/records.md + record-types.md + auth.md]
        Detected stack: Flutter (saw pubspec.yaml with riverpod).
        Endpoints I'll call:
          GET  /api/v1/records/?bbox=...
          GET  /api/v1/record-types/
        Files to create:
          lib/mapao/api_client.dart
          lib/mapao/records_repository.dart
          lib/features/map/records_map_view.dart
        OK to write?
```

## How the plugin stays accurate

Spec files under `.claude-plugin/specs/` and `skills/mapao-service/specs/`
are kept in sync automatically: a workflow in this repo
(`.github/workflows/sync-skill-specs.yml`) mirrors any change in the plugin
specs into the skill specs on every push to `main`, and fails PRs that
introduce drift.

Spec content itself is currently **maintained manually** against the live
FastAPI app. An automated regeneration pipeline (export script in the main
Mapao backend + cross-repo push to `.claude-plugin/specs/`) is planned;
until it lands, treat each release of this repo as a point-in-time
snapshot of `https://mapao-api.warnyin.com/api/v1`.

## License

MIT — see [LICENSE](./LICENSE).
