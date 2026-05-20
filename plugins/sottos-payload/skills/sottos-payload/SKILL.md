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

There should be no delete tools and no users tools.

## Safety Rules

- Create drafts first unless the user explicitly asks to publish.
- Prefer `select` to keep reads small.
- Never delete content.
- Never create, update, or elevate users.
- Never print or commit the MCP API key.
- If auth fails, ask the user to create or rotate a key in `/admin -> MCP -> API Keys`.

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
3. Create or update posts as drafts.
4. Return the post title, slug, status, and admin URL.

