---
name: content
description: Astro content collections and MDX patterns. Use when working with blog posts, documentation, or any structured content.
---

# Content Collections Skill

## Contents

- [Overview](#overview)
- [Collection Configuration](#collection-configuration)
- [Querying Content](#querying-content)
- [MDX Components](#mdx-components)
- [Frontmatter Patterns](#frontmatter-patterns)
- [Related Content](#related-content)
- [Content Rendering](#content-rendering)

---

## Overview

Astro Content Collections provide:
- Type-safe frontmatter with Zod schemas
- Built-in validation
- Automatic TypeScript types
- MDX support with custom components

---

## Collection Configuration

```typescript
// src/content/config.ts
import { defineCollection, z, reference } from 'astro:content';

const blogCollection = defineCollection({
  type: 'content', // Markdown/MDX files
  schema: ({ image }) => z.object({
    title: z.string(),
    description: z.string(),
    date: z.coerce.date(),
    updatedDate: z.coerce.date().optional(),
    draft: z.boolean().default(false),

    // Image with optimization
    cover: image().optional(),

    // References to other collections
    author: reference('authors'),

    // Arrays
    tags: z.array(z.string()).default([]),

    // Enums
    category: z.enum(['tech', 'lifestyle', 'news']),
  }),
});

const authorsCollection = defineCollection({
  type: 'data', // JSON/YAML files
  schema: ({ image }) => z.object({
    name: z.string(),
    bio: z.string(),
    avatar: image(),
    social: z.object({
      twitter: z.string().url().optional(),
      github: z.string().url().optional(),
    }).optional(),
  }),
});

export const collections = {
  blog: blogCollection,
  authors: authorsCollection,
};
```

---

## Querying Content

### Get All Entries

```astro
---
import { getCollection } from 'astro:content';

// Get all published posts
const posts = await getCollection('blog', ({ data }) => {
  return data.draft !== true;
});

// Sort by date
const sortedPosts = posts.sort(
  (a, b) => b.data.date.valueOf() - a.data.date.valueOf()
);
---
```

### Get Single Entry

```astro
---
import { getEntry, getEntries } from 'astro:content';

// By slug
const post = await getEntry('blog', 'my-post-slug');

// Get referenced author
const author = await getEntry(post.data.author);

// Get multiple references
const relatedPosts = await getEntries(post.data.relatedPosts);
---
```

### Filter and Transform

```typescript
// Get posts by tag
const techPosts = await getCollection('blog', ({ data }) => {
  return data.tags.includes('tech');
});

// Get posts by author
const authorPosts = await getCollection('blog', ({ data }) => {
  return data.author.id === 'john-doe';
});

// Get recent posts
const recentPosts = await getCollection('blog', ({ data }) => {
  const oneMonthAgo = new Date();
  oneMonthAgo.setMonth(oneMonthAgo.getMonth() - 1);
  return data.date > oneMonthAgo;
});
```

---

## MDX Components

### Register Components Globally

```typescript
// src/content/config.ts
import { defineCollection, z } from 'astro:content';

// Components available in all MDX files
export const mdxComponents = {
  Callout: () => import('../components/mdx/Callout.astro'),
  CodeBlock: () => import('../components/mdx/CodeBlock.astro'),
  YouTube: () => import('../components/mdx/YouTube.astro'),
};
```

### Use in MDX

```mdx
---
title: My Post
---

# Introduction

<Callout type="warning">
  This is important information!
</Callout>

<YouTube id="dQw4w9WgXcQ" />

## Code Example

<CodeBlock lang="typescript" filename="example.ts">
const hello = "world";
</CodeBlock>
```

### Callout Component Example

```astro
---
// src/components/mdx/Callout.astro
interface Props {
  type?: 'info' | 'warning' | 'error' | 'tip';
  title?: string;
}

const { type = 'info', title } = Astro.props;

const icons = {
  info: 'ℹ️',
  warning: '⚠️',
  error: '❌',
  tip: '💡',
};

const colors = {
  info: 'bg-blue-50 border-blue-500',
  warning: 'bg-yellow-50 border-yellow-500',
  error: 'bg-red-50 border-red-500',
  tip: 'bg-green-50 border-green-500',
};
---

<aside class={`callout border-l-4 p-4 my-4 ${colors[type]}`}>
  <div class="flex items-center gap-2 font-semibold mb-2">
    <span>{icons[type]}</span>
    {title && <span>{title}</span>}
  </div>
  <div class="callout-content">
    <slot />
  </div>
</aside>
```

---

## Frontmatter Patterns

### Draft System

```typescript
schema: z.object({
  draft: z.boolean().default(false),
  publishedAt: z.coerce.date().optional(),
  scheduledFor: z.coerce.date().optional(),
}),
```

```astro
---
// Filter drafts in production
const posts = await getCollection('blog', ({ data }) => {
  if (import.meta.env.PROD) {
    return data.draft !== true &&
           (!data.scheduledFor || data.scheduledFor <= new Date());
  }
  return true; // Show all in dev
});
---
```

### SEO Metadata

```typescript
schema: z.object({
  title: z.string().max(60),
  description: z.string().max(160),
  ogImage: image().optional(),
  canonicalUrl: z.string().url().optional(),
  noIndex: z.boolean().default(false),
}),
```

### Reading Time

```typescript
// Calculate in page template
const { Content, remarkPluginFrontmatter } = await post.render();
const readingTime = remarkPluginFrontmatter.readingTime;
```

Or use `remark-reading-time` plugin:

```javascript
// astro.config.mjs
import remarkReadingTime from 'remark-reading-time';

export default defineConfig({
  markdown: {
    remarkPlugins: [remarkReadingTime],
  },
});
```

---

## Related Content

### By Tags

```astro
---
const { post } = Astro.props;

const allPosts = await getCollection('blog');
const relatedPosts = allPosts
  .filter((p) => p.slug !== post.slug)
  .filter((p) => p.data.tags.some((tag) => post.data.tags.includes(tag)))
  .slice(0, 3);
---

{relatedPosts.length > 0 && (
  <section>
    <h2>Related Posts</h2>
    <ul>
      {relatedPosts.map((p) => (
        <li><a href={`/blog/${p.slug}`}>{p.data.title}</a></li>
      ))}
    </ul>
  </section>
)}
```

### Explicit References

```typescript
// In schema
schema: z.object({
  relatedPosts: z.array(reference('blog')).default([]),
}),
```

```mdx
---
title: Advanced TypeScript
relatedPosts:
  - typescript-basics
  - generics-guide
---
```

---

## Content Rendering

### Basic Rendering

```astro
---
import { getEntry } from 'astro:content';

const post = await getEntry('blog', Astro.params.slug);
if (!post) return Astro.redirect('/404');

const { Content, headings } = await post.render();
---

<article>
  <h1>{post.data.title}</h1>

  <!-- Table of contents from headings -->
  <nav>
    {headings.map((h) => (
      <a href={`#${h.slug}`} style={`margin-left: ${h.depth * 1}rem`}>
        {h.text}
      </a>
    ))}
  </nav>

  <!-- Rendered content -->
  <div class="prose">
    <Content />
  </div>
</article>
```

### With Custom Components

```astro
---
import { getEntry } from 'astro:content';
import Callout from '../components/mdx/Callout.astro';
import CodeBlock from '../components/mdx/CodeBlock.astro';

const post = await getEntry('blog', Astro.params.slug);
const { Content } = await post.render();
---

<Content components={{ Callout, CodeBlock }} />
```

---

## Best Practices

| Area | Recommendation |
|------|----------------|
| **Schemas** | Define strict schemas, use defaults for optional fields |
| **Validation** | Let Zod catch errors at build time |
| **References** | Use `reference()` for relationships, not strings |
| **Images** | Always use `image()` helper for optimized images |
| **Drafts** | Implement draft system from the start |
| **Types** | Export schema types for reuse |
| **Filtering** | Filter drafts/scheduled posts in production builds |
