---
description: Scaffold client code in the user's project for a Mapao feature — auto-detects stack (React/Next.js, Flutter, Swift, Kotlin, Vue, etc.)
argument-hint: <feature description, e.g. "records on a map", "login screen", "campaign list">
---

# Mapao — Scaffold

**Feature requested:** `$ARGUMENTS`

If `$ARGUMENTS` is empty, ask the user what they want to build (one short phrase). Do not guess.

## Step 1 — Detect the user's stack

Inspect the workspace root for stack signals (in this priority order):

1. `pubspec.yaml` -> Flutter/Dart
2. `package.json`:
   - dependency `next` -> Next.js (App Router unless `pages/` exists)
   - `react-native` -> React Native
   - `react` only -> plain React (Vite/CRA)
   - `vue` -> Vue
   - `svelte` -> Svelte
   - none of the above -> plain Node/TS
3. `*.xcodeproj`, `Package.swift`, `Podfile` -> Swift/iOS
4. `build.gradle*`, `settings.gradle*` -> Android (Kotlin preferred unless Java sources dominate)
5. `Cargo.toml` -> Rust; `go.mod` -> Go; `requirements.txt`/`pyproject.toml` -> Python; `composer.json` -> PHP

If multiple match (e.g. monorepo), ask the user which target. If none match, ask what stack they're using.

## Step 2 — Identify the Mapao endpoints needed

Based on `$ARGUMENTS`, decide which resources to involve. Common mappings:

- "records on a map" / "show flooded points" / "incidents nearby" -> `records` (geospatial query: `bbox`, `lng`+`lat`+`radius_m`), plus `record-types` for icons/colors
- "login" / "sign in" / "auth" -> `auth`
- "user profile" / "my account" / "delete my data" -> `me` + `auth`
- "list campaigns" / "campaign details" -> `campaigns`
- "upload photo to a record" / "attach file" -> `attachments`
- "dynamic form" / "edit record fields" -> `metadata` (form schema) + `records`
- "share a record by link" / "public viewer" -> `shares`

For each involved resource, read `${CLAUDE_PLUGIN_ROOT}/specs/<resource>.md` so the generated code matches real fields and verbs. Also read `${CLAUDE_PLUGIN_ROOT}/specs/auth.md` if any endpoint requires auth.

If a spec file is missing, stop and tell the user the plugin install is incomplete.

## Step 3 — Plan, then write

Before writing files, restate:

1. The detected stack (e.g. "Flutter 3.x using Riverpod — saw `riverpod` in `pubspec.yaml`").
2. The Mapao endpoints you will call.
3. The files you will create or edit (paths + one-line purpose each).

Wait for the user to confirm or course-correct unless the workspace is obviously empty (greenfield) and the answer is unambiguous.

Then generate code that:

- Uses the project's existing HTTP client / state management / routing conventions when present (do not introduce a new HTTP library if `dio`/`axios`/`ktor`/etc. is already in use).
- Reads `MAPAO_API_BASE_URL` (or the project's existing config pattern) — never hard-code the host.
- Sends credentials so the auth cookie tags along on browser/webview targets, or accepts a Bearer token on native targets.
- Handles 401 (re-auth), 403 (insufficient permission), 404, 422 (validation errors with `detail[]`), 429 (`Retry-After`), and 5xx (transient — retry with backoff for idempotent verbs only).
- Includes one usage example in a comment at the top of the main generated file.

## Step 4 — Verify

After writing, in your reply:

- List files created/edited.
- Show the one new dependency the user must install (if any).
- Show the env var(s) the user must set (`MAPAO_API_BASE_URL` etc.).
- Suggest a quick smoke test (a `curl` against `/api/v1/auth/login` plus one feature call).

Do not run tests or builds unless the user asks. Do not mock the network in generated code.
