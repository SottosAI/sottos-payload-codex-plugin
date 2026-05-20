---
name: sottos-payload
description: Use when working with Sottos Payload CMS, blog posts, tags, categories, media, or the hosted Sottos Payload MCP server.
---

# Sottos Payload CMS

Use the native `sottos-payload` MCP server for Sottos blog content work.

## First Step

If the `mcp__sottos_payload__` namespace is not visible, run tool discovery for:

```text
sottos-payload Payload CMS blog posts
```

Do not fall back to REST, SQL, or raw `curl` until MCP discovery has been tried.

If discovery shows the MCP server is unauthenticated, ask the user to run:

```bash
codex mcp login sottos-payload
```

This should open a browser window for Clerk login. Do not ask for an API key unless the server explicitly still uses legacy API-key auth.

## Expected Tools

- `findPosts`
- `createPosts`
- `updatePosts`
- `findTags`
- `createTags`
- `updateTags`
- `findCategories`
- `createCategories`
- `updateCategories`
- `findMedia`
- `createMedia`
- `updateMedia`

There should be no delete tools and no users tools.

## Safety Rules

- Create drafts first unless the user explicitly asks to publish.
- Prefer `select` to keep reads small.
- Never delete content.
- Never create, update, or elevate users.
- Never print or commit credentials.
- If auth fails, use the MCP OAuth login flow first: `codex mcp login sottos-payload`.
- For new cover images, use the host agent's available image-generation capability first, then upload the resulting local file with `createMedia`.
- Stay agent-agnostic: Codex/ChatGPT may use built-in image generation; Claude/Cursor should use whatever configured image-generation tool is available. If no image generator is available, write the exact image prompt and stop before creating the post.
- Do not fall back to REST, SQL, raw `curl`, seed scripts, or local env-secret workflows for media upload unless the user explicitly approves.

## Common Reads

List post titles:

```json
{
  "limit": 100,
  "depth": 0,
  "select": "{\"title\":true,\"slug\":true,\"_status\":true,\"updatedAt\":true}"
}
```

Find a post by slug:

```json
{
  "limit": 1,
  "depth": 1,
  "where": "{\"slug\":{\"equals\":\"post-slug\"}}"
}
```

## Draft Workflow

1. Search tags and categories first.
2. Create missing tags/categories only when needed.
3. For posts needing a new cover image, generate a 16:10 image locally using the active host agent's image tool.
4. Upload the generated image with `createMedia`, including descriptive SEO/accessibility alt text.
5. Create or update posts as drafts using the returned media ID as `coverImage`.
6. Return the post title, slug, status, cover media ID, and admin URL.
