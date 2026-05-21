# Sottos Payload MCP Plugin

Connect AI tools to the Sottos blog CMS.

Works with:

- Codex plugin marketplace
- Claude Code
- Claude Desktop / Claude.ai
- Cursor

Server URL:

```text
https://sottos-web.vercel.app/api/payload/mcp
```

## What It Can Do

- Create and update blog drafts.
- Upload and update cover images.
- Fetch compact draft context in one call.
- Show current cover images to the agent as visual style references.
- Manage tags and categories.
- Read existing posts, media, tags, and categories.
- Require Sottos Clerk OAuth.
- Keep post rollback available through Payload drafts/version history.

It should not expose user-management tools.

Not exposed through this MCP plugin today: site settings, redirects, search
index management, or direct user/admin management.

Delete tools are intentionally not exposed. If delete support is ever enabled,
the agent must ask for explicit confirmation first and prefer moving a post
back to draft/unpublished instead.

## Install In Codex

Fresh install:

```bash
codex plugin marketplace remove sottos
codex plugin marketplace add SottosAI/sottos-payload-codex-plugin
codex mcp login sottos-payload
```

Then relaunch Codex and enable `Sottos Payload CMS` if it is not already on.

Normal update:

```bash
codex plugin marketplace add SottosAI/sottos-payload-codex-plugin
```

Only remove the marketplace first if the local plugin cache is broken. Removing
it can require logging in again.

OAuth should open Sottos admin login. If you already have an active admin
session, it uses `https://sottos-web.vercel.app/admin`; otherwise it sends you
to `https://sottos-web.vercel.app/sign-in`. After login, OAuth refresh tokens
keep the MCP session alive without hourly browser re-auth.

## Install In Claude Code

OAuth:

```bash
claude mcp add-json sottos-payload \
  '{"type":"http","url":"https://sottos-web.vercel.app/api/payload/mcp"}'
```

Then run `/mcp`, select `sottos-payload`, and sign in with Sottos Clerk.

## Install In Claude Desktop / Claude.ai

OAuth:

1. Open Settings.
2. Open Connectors.
3. Add custom connector.
4. Name: `Sottos Payload CMS`
5. URL: `https://sottos-web.vercel.app/api/payload/mcp`
6. Sign in when Sottos Clerk opens.

## Install In Cursor

Add this to `~/.cursor/mcp.json` or `.cursor/mcp.json`:

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

## Creating Blog Posts

Ask the agent for a draft. It should:

1. Call `getBlogDraftContext` to get posts, tags, categories, and cover references in one request.
2. Create missing tags/categories only when needed.
3. Generate a 16:10 cover image with the best image model/tool available in the current agent.
4. For production covers, call `createMediaSourceUpload`, upload the original local image bytes to the signed URL, then call `uploadMedia` with the returned `sourceUrl` and descriptive alt text.
5. Create the post as a draft with `draft: true` and `_status: "draft"`.
6. Add SEO title, description, internal links, related posts, and reading time.
7. Return the draft title, slug, status, media id, and admin URL.

No image tool available? The agent should give you the exact cover prompt and
stop before creating the post.

Post drafts need: title, excerpt, cover image, and body content. Slugs can
auto-fill from the title. Future `publishedAt` dates schedule publishing.

Use `getBlogDraftContext` before drafting. It returns compact inventory and
cover reference images. Avoid reading full post bodies unless explicitly needed.

Media uploads are image-only. Use `createMediaSourceUpload` plus
`uploadMedia.sourceUrl` for production covers; keep `base64Data` for small
diagnostics only. Alt text is required; captions and credits are optional.
Uploads are normalized server-side to 1600x1000 WebP covers. Use the original
generated image at 1400x875 or larger; low-res, downscaled, upscaled, or
recompressed files are rejected so the blog never upscales blurry covers. Keep
inputs under 12MB.

## Verify

Ask:

```text
Use Sottos Payload CMS to list current blog posts. Do not create or update anything.
```

Expected namespace in Codex:

```text
mcp__sottos_payload__
```

OAuth metadata should be available at:

```text
https://sottos-web.vercel.app/.well-known/oauth-protected-resource
```

## Auth Notes

- OAuth is required. MCP API keys are not supported.
- Login must use the current Sottos Clerk admin user.
- Only current Sottos admins should receive write tools.
- Treat OAuth login like admin access.
