# Sottos Payload Codex Plugin

Private Codex plugin for the hosted Sottos Payload CMS MCP server.

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

```bash
codex plugin marketplace add kaankolcu/sottos-payload-codex-plugin
```

Then set the MCP token:

```bash
export SOTTOS_PAYLOAD_MCP_TOKEN="<payload-mcp-api-key>"
```

For Codex Desktop on macOS:

```bash
launchctl setenv SOTTOS_PAYLOAD_MCP_TOKEN "<payload-mcp-api-key>"
```

Fully quit and reopen Codex.

## Get An API Key

1. Open `https://sottos-web.vercel.app/admin`.
2. Sign in as a Clerk-backed admin.
3. Go to `MCP -> API Keys`.
4. Create a key linked to your admin user.
5. Enable only blog-safe permissions:
   - posts: find, create, update
   - tags: find, create, update
   - categories: find, create, update
   - media: find
6. Do not enable delete permissions.
7. Put the key in `SOTTOS_PAYLOAD_MCP_TOKEN`.

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

- Treat the API key like a password.
- Never commit the key.
- Revoke leaked keys in `/admin -> MCP -> API Keys`.
- Access depends on the linked Clerk user still being admin in Supabase.

