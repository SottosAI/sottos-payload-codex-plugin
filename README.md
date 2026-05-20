# Sottos Payload Codex Plugin

Public Codex plugin for the hosted Sottos Payload CMS MCP server.

## What It Adds

- MCP server: `sottos-payload`
- Safe blog tools only:
  - posts: find, create, update
  - tags: find, create, update
  - categories: find, create, update
  - media: find
- No delete tools.
- No users tools.

## Install In Codex

No GitHub access is needed once this repository is public.

```bash
codex plugin marketplace add kaankolcu/sottos-payload-codex-plugin
```

Then enable `Sottos Payload CMS` from the Codex plugin UI.

Config fallback:

```toml
[plugins."sottos-payload@sottos"]
enabled = true
```

Then authenticate:

```bash
codex mcp login sottos-payload
```

Codex should open a browser window. Sign in with a Clerk account that is admin in Sottos.

## Verify

Ask Codex:

```text
Use Sottos Payload CMS to list the available MCP tools. Do not create or update anything.
```

Expected native namespace after discovery:

```text
mcp__sottos_payload__
```

## Security

- No API key should be needed for the normal plugin flow.
- Treat the browser OAuth login like admin access.
- Access depends on the Clerk user still being admin in Supabase.
- Admin role is checked server-side on every MCP request.
