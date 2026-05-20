# Sottos Payload MCP Plugin

Hosted Sottos Payload CMS MCP connector, packaged as a Codex plugin and usable
from Claude Code, Claude Desktop/Claude.ai, and Cursor.

## What It Adds

- MCP server: `sottos-payload`
- Safe blog tools only:
  - posts: find, create, update
  - tags: find, create, update
  - categories: find, create, update
  - media: find, create, update
- No delete tools.
- No users tools.
- Clerk OAuth opens Sottos login; only current Sottos admins get tools.

## Image Workflow

For new posts, agents should generate cover images with the host's available
image tool, upload the local file through MCP `createMedia`, then create a
draft post with that media id.

Examples:

- Codex / ChatGPT: built-in image generation.
- Claude Code / Desktop: any configured image-generation tool.
- Cursor: any configured image model/tool.

If no image generator is available, write the exact cover prompt and stop
before creating the post.

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

## Install In Claude Code

```bash
claude mcp add-json sottos-payload \
  '{"type":"http","url":"https://sottos-web.vercel.app/api/payload/mcp"}'
```

Then run `/mcp` in Claude Code and authenticate `sottos-payload`.

## Install In Claude Desktop / Claude.ai

Settings -> Connectors -> Add custom connector:

- Name: `Sottos Payload CMS`
- URL: `https://sottos-web.vercel.app/api/payload/mcp`

Complete the OAuth flow when Clerk opens.

## Install In Cursor

Add this to `~/.cursor/mcp.json` or project `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "sottos-payload": {
      "url": "https://sottos-web.vercel.app/api/payload/mcp"
    }
  }
}
```

Then authenticate:

```bash
cursor-agent mcp login sottos-payload
```

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
