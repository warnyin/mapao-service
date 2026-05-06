---
description: Deep-dive a single Mapao resource — paths, params, request/response schemas, error codes, and example payloads
argument-hint: <resource> (auth | me | record-types | records | campaigns | attachments | metadata | shares)
---

# Mapao — Resource Spec

**Resource requested:** `$ARGUMENTS`

## Step 1 — Validate the argument

Allowed values: `auth`, `me`, `record-types`, `records`, `campaigns`, `attachments`, `metadata`, `shares`.

If `$ARGUMENTS` is empty, ask the user which resource. Do not guess.

If `$ARGUMENTS` does not match an allowed value, list the valid options and stop.

## Step 2 — Load the spec

Read `${CLAUDE_PLUGIN_ROOT}/specs/$ARGUMENTS.md`. Fall back to `.claude-plugin/specs/$ARGUMENTS.md` in the workspace or parent dirs, then under `~/.claude/plugins/`.

If the file is missing for a valid resource, tell the user the plugin install is incomplete and stop.

## Step 3 — Render the spec

For each endpoint in the spec, present:

1. **`METHOD /path`** as a header
2. **Auth:** required scope or "public"
3. **Path/query params:** name, type, required, default, description
4. **Request body:** field-by-field schema (name, type, required, constraints)
5. **Responses:** status code -> shape; include error codes (401, 403, 404, 409, 422, 429, 5xx) where the spec lists them
6. **Example:** one minimal `curl` and one minimal JSON request/response — only if the spec contains them

Do not invent fields. Do not paraphrase types ("string", "uuid", "integer >= 0" are precise; "some text" is not).

End with:
> Want to wire this up in your project? Run `/mapao-scaffold <feature>`.
