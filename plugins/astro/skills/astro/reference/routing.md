# Astro Routing Patterns

## Contents

- [File-Based Routing](#file-based-routing)
- [Dynamic Routes](#dynamic-routes)
- [Rest Parameters](#rest-parameters)
- [Route Priority](#route-priority)
- [Pagination](#pagination)
- [Redirects](#redirects)
- [i18n Routing](#i18n-routing)

---

## File-Based Routing

| File Path | URL |
|-----------|-----|
| `src/pages/index.astro` | `/` |
| `src/pages/about.astro` | `/about` |
| `src/pages/blog/index.astro` | `/blog` |
| `src/pages/blog/post-1.astro` | `/blog/post-1` |
| `src/pages/api/posts.ts` | `/api/posts` |

---

## Dynamic Routes

### Single Parameter

```astro
---
// src/pages/blog/[slug].astro
import { getCollection } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

interface Props {
  post: CollectionEntry<'blog'>;
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<Layout title={post.data.title}>
  <Content />
</Layout>
```

### Multiple Parameters

```astro
---
// src/pages/[lang]/[category]/[slug].astro
export async function getStaticPaths() {
  const pages = await fetchAllPages();
  return pages.map((page) => ({
    params: {
      lang: page.lang,
      category: page.category,
      slug: page.slug
    },
    props: { page },
  }));
}

const { lang, category, slug } = Astro.params;
const { page } = Astro.props;
---
```

---

## Rest Parameters

Catch multiple path segments with `[...path]`:

```astro
---
// src/pages/docs/[...path].astro
// Matches: /docs, /docs/intro, /docs/guides/getting-started

export async function getStaticPaths() {
  const docs = await getCollection('docs');
  return docs.map((doc) => ({
    params: { path: doc.slug || undefined },
    props: { doc },
  }));
}

const { path } = Astro.params;
// path is: undefined for /docs, "intro" for /docs/intro, "guides/getting-started" for nested
---
```

### 404 Fallback

```astro
---
// src/pages/[...slug].astro
// Catches all unmatched routes

// Return empty array to make it a fallback
export function getStaticPaths() {
  return [];
}
---

<Layout title="Page Not Found">
  <h1>404 - Page Not Found</h1>
</Layout>
```

---

## Route Priority

Astro routes are prioritized in this order:

1. Static routes (`/about`)
2. Dynamic routes with single parameter (`/blog/[slug]`)
3. Rest parameters (`/[...slug]`)

```
src/pages/
├── about.astro           # Matches /about first
├── blog/
│   └── [slug].astro     # Matches /blog/anything
└── [...slug].astro      # Fallback for unmatched routes
```

---

## Pagination

```astro
---
// src/pages/blog/[page].astro
import { getCollection } from 'astro:content';

export async function getStaticPaths({ paginate }) {
  const posts = await getCollection('blog');
  const sortedPosts = posts.sort(
    (a, b) => b.data.date.valueOf() - a.data.date.valueOf()
  );

  return paginate(sortedPosts, { pageSize: 10 });
}

interface Props {
  page: Page<CollectionEntry<'blog'>>;
}

const { page } = Astro.props;
---

<Layout title={`Blog - Page ${page.currentPage}`}>
  {page.data.map((post) => (
    <article>
      <a href={`/blog/${post.slug}`}>{post.data.title}</a>
    </article>
  ))}

  <nav class="flex gap-4">
    {page.url.prev && <a href={page.url.prev}>Previous</a>}
    <span>Page {page.currentPage} of {page.lastPage}</span>
    {page.url.next && <a href={page.url.next}>Next</a>}
  </nav>
</Layout>
```

---

## Redirects

### Static Redirects (astro.config.mjs)

```javascript
export default defineConfig({
  redirects: {
    '/old-page': '/new-page',
    '/blog/[...slug]': '/articles/[...slug]',
  },
});
```

### Dynamic Redirects (SSR)

```astro
---
// src/pages/old-path.astro
return Astro.redirect('/new-path', 301);
---
```

### Redirect with Parameters

```astro
---
// src/pages/go/[id].astro
export async function getStaticPaths() {
  const links = await fetchShortLinks();
  return links.map((link) => ({
    params: { id: link.shortId },
    props: { url: link.targetUrl },
  }));
}

const { url } = Astro.props;
return Astro.redirect(url, 302);
---
```

---

## i18n Routing

### Manual i18n Structure

```
src/pages/
├── en/
│   ├── index.astro
│   └── about.astro
├── es/
│   ├── index.astro
│   └── about.astro
└── index.astro          # Redirect to default language
```

### Built-in i18n (Astro 4.x+)

```javascript
// astro.config.mjs
export default defineConfig({
  i18n: {
    defaultLocale: 'en',
    locales: ['en', 'es', 'fr'],
    routing: {
      prefixDefaultLocale: false, // /about vs /en/about
    },
  },
});
```

```astro
---
// Usage in components
import { getRelativeLocaleUrl } from 'astro:i18n';

const aboutUrl = getRelativeLocaleUrl('es', '/about');
// Returns: /es/about
---
```

---

## SSR Dynamic Routes

With SSR enabled, you can have truly dynamic routes:

```typescript
// astro.config.mjs
export default defineConfig({
  output: 'server', // or 'hybrid'
});
```

```astro
---
// src/pages/user/[id].astro
// No getStaticPaths needed with SSR!

const { id } = Astro.params;
const user = await fetchUser(id);

if (!user) {
  return Astro.redirect('/404');
}
---

<Layout title={user.name}>
  <h1>{user.name}</h1>
</Layout>
```
