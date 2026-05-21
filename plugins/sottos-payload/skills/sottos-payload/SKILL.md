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

This should open Sottos admin login. If an admin session exists, it uses `https://sottos.ai/admin`; otherwise it sends the user to `https://sottos.ai/sign-in`. Do not ask for an API key. Clerk OAuth is required.

After a successful login, clients should keep access via OAuth refresh tokens. If auth repeatedly expires after about an hour, the server deployment is stale and should be updated before asking the user to log in again.

## Expected Tools

- `findPosts`
- `createBlogDraft`
- `updateBlogDraft`
- `findTags`
- `createTags`
- `updateTags`
- `findCategories`
- `createCategories`
- `updateCategories`
- `getBlogDraftContext`
- `findMedia`
- `getMediaReferences`
- `createMediaSourceUpload`
- `uploadMedia`
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
- For new draft posts, call `getBlogDraftContext` first. It replaces separate inventory reads and returns cover-image style references.
- Prefer omitting optional `getBlogDraftContext` limits. Hard caps are `maxImages<=6`, `mediaLimit<=8`, and `postLimit<=30`; default is metadata-only for speed. Call `getMediaReferences` when visual references are needed.
- Do not read full existing post bodies just to learn the content shape. Build Lexical content directly with root/paragraph/heading/list/link nodes.
- For new cover images, use the references from `getBlogDraftContext` or `getMediaReferences`, then use the best image-generation model/tool available in the current host agent.
- Production covers: call `createMediaSourceUpload`, upload the original local image bytes to the returned signed `uploadUrl`, then call `uploadMedia` with the returned `sourceUrl`. Use base64 only for small diagnostics or when signed source upload is unavailable.
- Upload the generated image as-is unless it exceeds 12MB. Source images must be at least 1400x875 pixels. Do not downscale, upscale, or recompress first; `uploadMedia` normalizes covers server-side to 1600x1000 WebP and rejects low-resolution derivatives to prevent blurry covers.
- If the image tool only produced a smaller file, regenerate at a higher resolution or stop before creating the post. Never upscale a small image locally just to pass validation.
- Stay agent-agnostic: Codex/ChatGPT should use their strongest available image-generation capability; Claude/Cursor should use the best configured image-generation tool or connected image MCP available. If no image generator is available, write the exact image prompt and stop before creating the post.
- Do not fall back to REST, SQL, seed scripts, or local env-secret workflows for media upload unless the user explicitly approves. The signed PUT URL returned by `createMediaSourceUpload` is the only expected direct HTTP upload step.
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

1. Call `getBlogDraftContext` first. If unavailable, fall back to compact `findPosts`, `findTags`, `findCategories`, and `getMediaReferences`.
2. Create missing tags/categories only when needed.
3. Generate a 16:10 image locally using the strongest available image model/tool in the active host agent, matching the Sottos editorial cover style without copying exact layouts or text.
4. For production covers, call `createMediaSourceUpload` with `fileName`, `mimeType`, and optional `contentLength`; upload the original local image bytes to the returned signed `uploadUrl`; then call `uploadMedia` with `sourceUrl`, `fileName`, `mimeType`, and descriptive SEO/accessibility `alt` text. Use `base64Data` only for small diagnostics or when signed source upload is unavailable. Media is image-only; `alt` is required; captions and credits are optional. Keep the image under 12MB and at least 1400x875 pixels. The server validates and normalizes it to a 1600x1000 WebP cover.
5. Pick 2-3 relevant `relatedPosts` from `getBlogDraftContext`.
6. Create or update posts only through `createBlogDraft` / `updateBlogDraft` using the returned media ID as `coverImage`. Prefer `bodyMarkdown` unless exact Lexical nodes are required. These tools force draft status server-side; do not use generic post create/update tools for blog posts.
7. Return the post title, slug, status, cover media ID, and the returned canonical admin URL. Human-facing admin URLs must use `https://sottos.ai`, not `sottos-web.vercel.app`.

## Delete / Unpublish Workflow

1. If the user asks to remove a post, recommend changing `_status` to `draft` first.
2. If the user still wants deletion and a delete tool exists, ask for explicit confirmation naming the exact item.
3. After confirmation, delete only that item and report what was deleted.
4. If no delete tool exists, explain that deletion is intentionally unavailable through MCP.
