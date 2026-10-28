---
description: Mapao authentication — secret API keys for servers (X-API-Key), publishable keys for browsers, and the end-user login flow (cookie / Bearer, refresh, logout)
---

# Mapao — Auth Flow

The user wants to authenticate against Mapao. Help them with the **exact** model Mapao uses, not a generic OAuth template. First decide who calls Mapao:

- **Their server / job / back end** (no end-user login) -> a secret API key. Create it in the console at `/console` (app -> key, narrowest scopes; shown once), keep it in the server env as `MAPAO_API_KEY`, and send `X-API-Key: sk_...` (or `Authorization: Bearer sk_...`) on every request. Never put `sk_` keys in browser code, mobile apps, bundles or commits.
- **A browser or mobile app reading public-ish data** -> a publishable `pk_` key: read-only, bound to `allowed_origins`; it gets 403 on writes and on `/location/assess`.
- **An app where people sign in** -> the end-user login flow below.

Most apps that write data or grade GPS fixes use a server route holding the `sk_` key.

## Step 1 — Load context

Read `${CLAUDE_PLUGIN_ROOT}/specs/auth.md`. Fall back to `.claude-plugin/specs/auth.md` in the workspace, then in parent dirs, then under `~/.claude/plugins/`.

If missing, tell the user the plugin install is incomplete and stop.

## Step 2 — Detect the user's stack

Before writing code, look for:

- `package.json` → JS/TS (check for `next`, `react`, `react-native`, `vue`, `svelte`)
- `pubspec.yaml` → Flutter/Dart
- `*.xcodeproj` / `Package.swift` → Swift / iOS
- `build.gradle*` / `settings.gradle*` → Android (Kotlin/Java)
- `Cargo.toml`, `go.mod`, `requirements.txt`, `composer.json` → other backends/CLIs

If you cannot determine the stack, ask the user before scaffolding code. Do not assume.

## Step 3 — Explain the flow, then offer code

In your reply:

1. **Login** — POST `/api/v1/auth/login` with `{ email, password }`. Response: `{ access_token, refresh_token, token_type }` AND a `Set-Cookie: mapao_access_token=…; HttpOnly; SameSite=Lax` header. Browsers store the cookie automatically.
2. **Authenticated requests** — Web/mobile webviews: send credentials so the cookie tags along (`credentials: 'include'` in `fetch`, `withCredentials: true` in `axios`, `dio.options.credentials` in Flutter web). Native mobile (no cookie jar): use the `access_token` from the response body as `Authorization: Bearer <token>`.
3. **Refresh** — POST `/api/v1/auth/refresh` with the refresh token; the server rotates and re-sets the cookie.
4. **Logout** — POST `/api/v1/auth/logout` clears the cookie server-side.
5. **Lockout** — 10 consecutive failures lock the account for 15 minutes (HTTP 429 with `Retry-After`).

After explaining, offer to generate a concrete client for the detected stack (e.g. a `MapaoAuthClient` class for Dart/Flutter, a `useMapaoAuth` hook for React, a Retrofit service for Kotlin).

## Step 4 — Cross-origin gotcha

If the user is building a browser frontend that runs on a **different origin** than the Mapao API, warn them:

- The cookie is bound to the Mapao API host. Either reverse-proxy `/api/*` through the frontend origin (recommended for dev), or rely on the `access_token` body value as Bearer.
- Without the cookie or Bearer, every authenticated request returns 401.

Keep examples to one stack per reply. If the user asked for multiple, ask which to start with.
