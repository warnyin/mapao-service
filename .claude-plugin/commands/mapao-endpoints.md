---
description: List every public Mapao API endpoint with method, path, and one-line purpose
---

# Mapao — Endpoint Catalog

The user wants a quick scan of what's available before deciding what to call.

## Step 1 — Load context

Read `${CLAUDE_PLUGIN_ROOT}/specs/endpoints.md`. Fall back to `.claude-plugin/specs/endpoints.md` in the workspace or parent dirs, then under `~/.claude/plugins/`.

If missing, tell the user the plugin install is incomplete and stop.

## Step 2 — Render the catalog

Output the catalog as a compact Markdown table grouped by resource:

```
### auth
| Method | Path | Purpose |
| --- | --- | --- |
| POST | /api/v1/auth/login | …
```

Group order: `auth`, `me`, `record-types`, `records`, `campaigns`, `attachments`, `metadata`, `shares`.

Do not invent endpoints. If a row is not in `endpoints.md`, omit it.

## Step 3 — Suggest the next step

End with one line offering follow-ups:

> Run `/mapao-spec <resource>` for full request/response shapes, or `/mapao-scaffold <feature>` to generate client code.
