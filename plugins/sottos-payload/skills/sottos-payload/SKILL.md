---
name: sottos-payload
description: Use when working with Sottos Payload CMS, blog posts, tags, categories, media, or the hosted Sottos Payload MCP server.
---

# Sottos Payload CMS

Use the native `sottos-payload` MCP server for Sottos blog content work.

Default stance: draft-first, rollback-safe, and user-approved for destructive changes.

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

This should open Sottos admin login. If an admin session exists, it uses `https://sottos-web.vercel.app/admin`; otherwise it sends the user to `https://sottos-web.vercel.app/sign-in`. Do not ask for an API key. Clerk OAuth is required.

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

There should be no users tools. Delete tools should normally be absent. If a delete tool is ever exposed, do not call it until the user explicitly confirms the exact post/media/tag/category to delete in the current conversation.

## Safety Rules

- Create drafts first unless the user explicitly asks to publish.
- Prefer rollback-safe edits: update drafts or move posts back to draft instead of deleting.
- Prefer `select` to keep reads small.
- Never delete content without explicit current-turn user confirmation. Prefer unpublishing/drafting over deletion.
- Never create, update, or elevate users.
- Never print or commit credentials.
- If auth fails, use the MCP OAuth login flow first: `codex mcp login sottos-payload`.
- Auth must come from the current Sottos Clerk admin user. Do not use MCP API-key or bearer-token fallback instructions.
- For new cover images, use the host agent's available image-generation capability first, then upload the resulting local file with `createMedia`.
- Stay agent-agnostic: Codex/ChatGPT may use built-in image generation; Claude/Cursor should use whatever configured image-generation tool is available. If no image generator is available, write the exact image prompt and stop before creating the post.
- Do not fall back to REST, SQL, raw `curl`, seed scripts, or local env-secret workflows for media upload unless the user explicitly approves.
- Preserve rollback: Sottos posts use Payload drafts/version history. Do not hard-delete or overwrite published content when a draft/update path is available.
- Include SEO fields on blog drafts: meta title, meta description, cover image alt text, internal links, related posts, tags, category, and reading time when possible.
- Match current Sottos blog style: field-manual voice, specific comparisons/tests, clear TL;DR, H2/H3 structure, FAQ section, and CTA to `/download` or relevant blog posts.

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
4. Upload the generated image with `createMedia`, including descriptive SEO/accessibility alt text. Media is image-only; `alt` is required; `caption` and `credit` are optional. Keep the file reasonably small before upload.
5. Read related existing posts and pick 2-3 relevant `relatedPosts`.
6. Create or update posts as drafts using the returned media ID as `coverImage`. Required post fields: `title`, `excerpt`, `coverImage`, and Lexical `content`. Use `_status: "draft"` first. Let `slug` auto-fill unless the user asks for a specific URL. Use `publishedAt` only for user-approved scheduling.
7. Return the post title, slug, status, cover media ID, and admin URL.

## Delete / Unpublish Workflow

1. If the user asks to remove a post, recommend changing `_status` to `draft` first.
2. If the user still wants deletion and a delete tool exists, ask for explicit confirmation naming the exact item.
3. After confirmation, delete only that item and report what was deleted.
4. If no delete tool exists, explain that deletion is intentionally unavailable through MCP.
