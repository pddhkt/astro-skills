---
name: astro
description: Astro content-focused web framework patterns. Use when implementing pages, layouts, components, or API routes in Astro projects.
---

# Astro Development Skill

## Contents

- [Stack](#stack)
- [Directory Structure](#directory-structure)
- [Reference Files](#reference-files)
- [Component Template](#astro-component-template)
- [Naming Conventions](#naming-conventions)
- [TypeScript in Frontmatter](#typescript-in-frontmatter)
- [Client Directives](#client-directives)

---

## Stack

- Astro 4.x with TypeScript
- File-based routing
- Content collections for structured content
- Islands architecture for interactivity
- Optional: React, Vue, Svelte integrations
- Tailwind CSS for styling

---

## Directory Structure

```
src/
├── pages/                     # File-based routing
│   ├── index.astro           # / route
│   ├── about.astro           # /about route
│   ├── blog/
│   │   ├── index.astro       # /blog route
│   │   └── [slug].astro      # /blog/:slug dynamic route
│   └── api/
│       └── posts.ts          # /api/posts endpoint
├── layouts/
│   ├── BaseLayout.astro      # Base HTML structure
│   └── BlogLayout.astro      # Blog-specific layout
├── components/
│   ├── Header.astro          # Static components
│   ├── Footer.astro
│   └── islands/              # Interactive components
│       ├── Counter.tsx       # React island
│       └── Search.vue        # Vue island
├── content/
│   ├── config.ts             # Collection schemas
│   └── blog/                 # Blog collection
│       └── *.mdx
├── styles/
│   └── global.css            # Global styles
└── lib/
    └── utils.ts              # Shared utilities
public/
└── assets/                   # Static assets
```

---

## Reference Files

Detailed patterns and examples are in separate reference files:

| Topic | File | Description |
|-------|------|-------------|
| **Routing** | [reference/routing.md](reference/routing.md) | File-based routing, dynamic routes, redirects |
| **Components** | [reference/components.md](reference/components.md) | .astro syntax, slots, props, styling |

---

## Astro Component Template

```astro
---
// Frontmatter: Runs at build time (or request time with SSR)
import Layout from '../layouts/BaseLayout.astro';
import { getCollection } from 'astro:content';

interface Props {
  title: string;
  description?: string;
}

const { title, description = 'Default description' } = Astro.props;

// Data fetching at build time
const posts = await getCollection('blog');
---

<Layout title={title}>
  <main class="container mx-auto px-4 py-8">
    <h1 class="text-3xl font-bold mb-4">{title}</h1>
    {description && <p class="text-muted-foreground">{description}</p>}

    <ul class="space-y-4 mt-8">
      {posts.map((post) => (
        <li>
          <a href={`/blog/${post.slug}`} class="hover:underline">
            {post.data.title}
          </a>
        </li>
      ))}
    </ul>
  </main>
</Layout>

<style>
  /* Scoped CSS - only applies to this component */
  h1 {
    color: var(--accent);
  }
</style>
```

---

## Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Pages | lowercase with hyphens | `about-us.astro` |
| Layouts | PascalCase | `BlogLayout.astro` |
| Components | PascalCase | `Header.astro` |
| Dynamic routes | brackets | `[slug].astro`, `[...path].astro` |
| API routes | lowercase | `posts.ts` |
| Content collections | lowercase plural | `blog/`, `authors/` |

---

## TypeScript in Frontmatter

```astro
---
// Type imports
import type { CollectionEntry } from 'astro:content';

// Props interface (required for type safety)
interface Props {
  post: CollectionEntry<'blog'>;
  showAuthor?: boolean;
}

// Destructure with defaults
const { post, showAuthor = true } = Astro.props;

// Type assertions
const { title, date, tags } = post.data;
---
```

---

## Client Directives

Use client directives to hydrate interactive islands:

| Directive | When Hydrated | Use Case |
|-----------|---------------|----------|
| `client:load` | Immediately on page load | Critical interactivity (nav menus) |
| `client:idle` | After page is idle | Non-critical widgets |
| `client:visible` | When scrolled into view | Below-fold content |
| `client:media` | When media query matches | Mobile-only features |
| `client:only` | Client-only, no SSR | Browser-specific code |

```astro
---
import Counter from '../components/islands/Counter.tsx';
import Comments from '../components/islands/Comments.tsx';
import MobileMenu from '../components/islands/MobileMenu.tsx';
---

<!-- Load immediately - needed right away -->
<Counter client:load initialCount={0} />

<!-- Load when visible - below the fold -->
<Comments client:visible postId={post.id} />

<!-- Load only on mobile -->
<MobileMenu client:media="(max-width: 768px)" />

<!-- React-only component, no server rendering -->
<SomeReactComponent client:only="react" />
```

---

## API Routes

```typescript
// src/pages/api/posts.ts
import type { APIRoute } from 'astro';

export const GET: APIRoute = async ({ request, params }) => {
  const posts = await fetchPosts();

  return new Response(JSON.stringify(posts), {
    status: 200,
    headers: {
      'Content-Type': 'application/json',
    },
  });
};

export const POST: APIRoute = async ({ request }) => {
  const body = await request.json();

  // Validate with Zod
  const result = postSchema.safeParse(body);
  if (!result.success) {
    return new Response(JSON.stringify({ error: result.error }), {
      status: 400,
    });
  }

  const post = await createPost(result.data);
  return new Response(JSON.stringify(post), { status: 201 });
};
```

---

## Best Practices

| Area | Recommendation |
|------|----------------|
| **Interactivity** | Use islands only where needed, prefer static |
| **Images** | Always use `<Image />` component for optimization |
| **SEO** | Include meta tags in layouts |
| **Data fetching** | Fetch in frontmatter, not client-side |
| **Styling** | Scoped CSS or Tailwind, avoid global styles |
| **Types** | Define Props interface for all components |
| **Content** | Use content collections for structured data |
