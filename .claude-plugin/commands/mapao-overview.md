---
description: Mapao API overview — what the service does, domain model, and how clients consume it
---

# Mapao — Overview

You are about to help the user build a frontend or mobile client that talks to the Mapao public API.

## Step 1 — Load context

Read the local snapshot at `${CLAUDE_PLUGIN_ROOT}/specs/overview.md`. If `${CLAUDE_PLUGIN_ROOT}` is not set, fall back to searching for `.claude-plugin/specs/overview.md` in the active workspace, then in the parent directories, then under `~/.claude/plugins/`.

If the file is missing, tell the user the plugin install is incomplete and stop — do **not** invent endpoints.

## Step 2 — Summarize for the user

Once the spec is loaded, in your reply:

1. State Mapao's purpose in one sentence (universal geospatial backend with multi-tenant records, record-types, and campaigns).
2. List the public resources: `auth`, `records`, `record-types`, `boundaries` (Thai province/district/tambon resolve), `location` (GPS trust / anti-spoof), `tiles` (vector tiles), `campaigns`, `attachments`, `metadata`, `shares`, `me`.
3. Describe the auth model in 2 sentences (servers send a secret key from `/console` as `X-API-Key: sk_...`; apps with signed-in users use the httpOnly cookie or Bearer from `/auth/login`).
4. Mention the related slash commands the user can run next:
   - `/mapao-endpoints` — list every public endpoint
   - `/mapao-auth` — API keys vs end-user login, and how to wire credentials in any HTTP client
   - `/mapao-spec <resource>` — deep-dive a single resource (request/response shapes, errors, examples)
   - `/mapao-scaffold <feature>` — generate code in the current project for a use case

Keep the answer compact. Do not dump the full spec — just orient the user. They will ask for detail next.
