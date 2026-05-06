# Mapao - Public API Overview

Mapao is a multi-tenant geospatial backend. Client apps consume REST
endpoints under `/api/v1`. The public surface area exposed to external
integrations is grouped into the following resources:

- **auth** - Auth
- **me** - Me (PDPA self-service)
- **record-types** - Record Types
- **records** - Records
- **campaigns** - Campaigns
- **attachments** - Attachments
- **metadata** - Metadata (UI Generation)
- **shares** - Share Links

## Base URL

- Production: `https://mapao-api.warnyin.com`
- All endpoints below are relative to this base + `/api/v1`.
- Clients should read the host from a config/env var (`MAPAO_API_BASE_URL`)
  rather than hard-code it, so dev/staging swaps stay one config change away.

## Auth model

- Email/password login at `POST /api/v1/auth/login`.
- Server returns `{ access_token, refresh_token }` AND sets `mapao_access_token`
  as an httpOnly cookie.
- Browsers/webviews rely on the cookie automatically (`credentials: 'include'`).
- Native apps use `Authorization: Bearer <access_token>` from the response body.
- Refresh via `POST /api/v1/auth/refresh`. Logout via `POST /api/v1/auth/logout`.
- 10 consecutive login failures lock the account for 15 minutes (HTTP 429).

## Tenancy

- Every resource is tenant-scoped. The tenant is derived from the authenticated
  user's session - clients do not pass a tenant id explicitly.
- Cross-tenant access is admin-only and is intentionally excluded from this plugin.

_Generated from FastAPI app version `0.1.0`._
