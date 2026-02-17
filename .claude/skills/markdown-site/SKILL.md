---
name: markdown-site
description: Comprehensive reference for markdown-site development with Convex, React, and Vite. Use when working on content sync, frontmatter configuration, Convex queries/mutations, or full-stack features.
---

# Markdown Site Development

Complete guide for developing the markdown-site project - a React + Vite + Convex blog platform with instant content sync.

## Quick Reference

| Topic | File | Use When |
|-------|------|----------|
| Content sync | [references/sync.md](references/sync.md) | Running sync commands, adding frontmatter fields |
| Frontmatter | [references/frontmatter.md](references/frontmatter.md) | Writing posts/pages, configuring metadata |
| Convex patterns | [references/convex.md](references/convex.md) | Writing queries, mutations, indexes |
| Development | [references/dev.md](references/dev.md) | General coding, React, auth, design |

## Project Architecture

```
content/blog/*.md  ──┐
                     ├──▶ npm run sync ──▶ Convex DB ──▶ React Site
content/pages/*.md ──┘
```

**Tech Stack:**
- Frontend: React 18 + TypeScript + Vite
- Backend: Convex (real-time serverless)
- Styling: CSS variables (no preprocessor)
- Hosting: Netlify with edge functions
- Content: Markdown with gray-matter frontmatter

## Common Commands

```bash
# Sync content to development
npm run sync

# Sync to production
npm run sync:prod

# Sync everything including discovery files
npm run sync:all

# Import external URL as post
npm run import https://example.com/article

# Development server
npm run dev

# Build
npm run build
```

## Content Structure

### Blog Posts

Location: `content/blog/*.md`

Required frontmatter:
```yaml
---
title: "Post Title"
description: "SEO description"
date: "2025-01-15"
slug: "post-slug"
published: true
tags: ["tag1", "tag2"]
---
```

### Pages

Location: `content/pages/*.md`

Required frontmatter:
```yaml
---
title: "Page Title"
slug: "page-slug"
published: true
---
```

## Convex Patterns

### Always Use Indexes

```typescript
// Good - uses index
const post = await ctx.db
  .query("posts")
  .withIndex("by_slug", (q) => q.eq("slug", args.slug))
  .first();

// Bad - table scan
const post = await ctx.db
  .query("posts")
  .filter((q) => q.eq(q.field("slug"), args.slug))
  .first();
```

### Function Structure

```typescript
export const myQuery = query({
  args: { slug: v.string() },
  returns: v.union(v.object({...}), v.null()),
  handler: async (ctx, args) => {
    // Always use validators
  },
});
```

### Idempotent Mutations

```typescript
export const update = mutation({
  args: { id: v.id("posts"), content: v.string() },
  returns: v.null(),
  handler: async (ctx, args) => {
    // Patch directly without reading
    await ctx.db.patch(args.id, { content: args.content });
    return null;
  },
});
```

## Key Indexes

```typescript
posts
  .index("by_slug", ["slug"])
  .index("by_published", ["published"])
  .index("by_featured", ["featured"])
  .searchIndex("search_title", { searchField: "title" })

pages
  .index("by_slug", ["slug"])
  .index("by_published", ["published"])
```

## File Locations

| File | Purpose |
|------|---------|
| `convex/schema.ts` | Database schema |
| `convex/posts.ts` | Post queries/mutations |
| `convex/pages.ts` | Page queries/mutations |
| `convex/stats.ts` | Analytics |
| `scripts/sync-posts.ts` | Content sync script |
| `src/config/siteConfig.ts` | Site configuration |

## Workflow

1. Create/edit markdown in `content/blog/` or `content/pages/`
2. Run `npm run sync` to sync to Convex
3. Changes appear instantly on the site
4. No rebuild needed for content changes

## Detailed Documentation

See reference files for complete documentation on each topic.
