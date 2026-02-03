# Astro Component Patterns

## Contents

- [Component Anatomy](#component-anatomy)
- [Props and Types](#props-and-types)
- [Slots](#slots)
- [Conditional Rendering](#conditional-rendering)
- [Loops and Lists](#loops-and-lists)
- [Scoped Styles](#scoped-styles)
- [Global Styles](#global-styles)
- [CSS Variables](#css-variables)
- [Built-in Components](#built-in-components)

---

## Component Anatomy

```astro
---
// FRONTMATTER (Component Script)
// Runs on the server at build time (or request time with SSR)

// 1. Imports
import BaseLayout from '../layouts/BaseLayout.astro';
import Card from './Card.astro';

// 2. Props interface
interface Props {
  title: string;
  items: string[];
}

// 3. Props destructuring
const { title, items } = Astro.props;

// 4. Data fetching / processing
const processedItems = items.filter(item => item.length > 0);
---

<!-- TEMPLATE (Component Markup) -->
<BaseLayout title={title}>
  <h1>{title}</h1>

  {processedItems.map((item) => (
    <Card title={item} />
  ))}

  <slot />
</BaseLayout>

<!-- STYLES (Scoped by default) -->
<style>
  h1 {
    color: var(--accent);
  }
</style>

<!-- SCRIPTS (Client-side JavaScript) -->
<script>
  console.log('This runs in the browser');
</script>
```

---

## Props and Types

### Basic Props

```astro
---
interface Props {
  title: string;
  count: number;
  isActive?: boolean;  // Optional
  variant?: 'primary' | 'secondary';  // Union type
}

const {
  title,
  count,
  isActive = false,  // Default value
  variant = 'primary'
} = Astro.props;
---
```

### Complex Props

```astro
---
import type { CollectionEntry } from 'astro:content';

interface Author {
  name: string;
  avatar: string;
}

interface Props {
  post: CollectionEntry<'blog'>;
  author: Author;
  relatedPosts?: CollectionEntry<'blog'>[];
}

const { post, author, relatedPosts = [] } = Astro.props;
---
```

### HTML Attributes Pass-through

```astro
---
interface Props {
  variant: 'primary' | 'secondary';
}

const { variant, ...attrs } = Astro.props;
---

<button class={`btn btn-${variant}`} {...attrs}>
  <slot />
</button>
```

---

## Slots

### Default Slot

```astro
---
// Card.astro
interface Props {
  title: string;
}
const { title } = Astro.props;
---

<div class="card">
  <h2>{title}</h2>
  <div class="card-content">
    <slot />  <!-- Children go here -->
  </div>
</div>
```

### Named Slots

```astro
---
// Layout.astro
---
<div class="layout">
  <header>
    <slot name="header" />
  </header>

  <main>
    <slot />  <!-- Default slot -->
  </main>

  <footer>
    <slot name="footer" />
  </footer>
</div>
```

```astro
---
// Usage
import Layout from './Layout.astro';
---

<Layout>
  <h1 slot="header">Page Title</h1>

  <p>Main content goes in default slot</p>

  <p slot="footer">Footer content</p>
</Layout>
```

### Fallback Content

```astro
<slot>
  <p>This shows if no content is provided</p>
</slot>

<slot name="sidebar">
  <nav>Default sidebar</nav>
</slot>
```

### Checking for Slot Content

```astro
---
const hasHeader = Astro.slots.has('header');
---

{hasHeader && (
  <header>
    <slot name="header" />
  </header>
)}
```

---

## Conditional Rendering

```astro
---
const { isLoggedIn, role, items } = Astro.props;
---

<!-- Simple conditional -->
{isLoggedIn && <p>Welcome back!</p>}

<!-- Ternary -->
{isLoggedIn ? (
  <a href="/dashboard">Dashboard</a>
) : (
  <a href="/login">Login</a>
)}

<!-- Multiple conditions -->
{role === 'admin' && <AdminPanel />}
{role === 'user' && <UserDashboard />}
{!role && <GuestView />}

<!-- Conditional with Fragment -->
{items.length > 0 && (
  <>
    <h2>Items</h2>
    <ItemList items={items} />
  </>
)}
```

---

## Loops and Lists

```astro
---
const items = ['Apple', 'Banana', 'Cherry'];
const posts = await getCollection('blog');
---

<!-- Basic loop -->
<ul>
  {items.map((item) => <li>{item}</li>)}
</ul>

<!-- Loop with index -->
<ul>
  {items.map((item, index) => (
    <li class={index === 0 ? 'first' : ''}>
      {index + 1}. {item}
    </li>
  ))}
</ul>

<!-- Loop with object -->
{posts.map((post) => (
  <article>
    <h2><a href={`/blog/${post.slug}`}>{post.data.title}</a></h2>
    <time datetime={post.data.date.toISOString()}>
      {post.data.date.toLocaleDateString()}
    </time>
  </article>
))}

<!-- Async iteration -->
{await Promise.all(
  posts.map(async (post) => {
    const { Content } = await post.render();
    return <Content />;
  })
)}
```

---

## Scoped Styles

```astro
<style>
  /* Scoped to this component only */
  h1 {
    color: blue;
  }

  .card {
    padding: 1rem;
    border: 1px solid #ddd;
  }

  /* Escape hatch - global within component */
  :global(.prose h2) {
    margin-top: 2rem;
  }
</style>

<!-- Inline styles -->
<div style="color: red; font-size: 1.5rem;">Inline styled</div>

<!-- Dynamic class -->
<div class={isActive ? 'active' : 'inactive'}>...</div>

<!-- Multiple classes -->
<div class:list={['card', { active: isActive }, variant]}>...</div>
```

---

## Global Styles

```astro
<style is:global>
  /* Applied globally to entire page */
  body {
    font-family: system-ui;
  }

  .prose {
    max-width: 65ch;
  }
</style>
```

### External Stylesheet

```astro
---
// In layout
import '../styles/global.css';
---

<head>
  <link rel="stylesheet" href="/styles/external.css" />
</head>
```

---

## CSS Variables

```astro
<style>
  :root {
    --card-padding: 1rem;
    --card-bg: white;
  }

  .card {
    padding: var(--card-padding);
    background: var(--card-bg);
  }
</style>

<!-- Dynamic CSS variables -->
<div style={`--accent-color: ${accentColor}`}>
  <span class="accent">Dynamically colored</span>
</div>

<style>
  .accent {
    color: var(--accent-color);
  }
</style>
```

---

## Built-in Components

### Image Component

```astro
---
import { Image } from 'astro:assets';
import heroImage from '../assets/hero.png';
---

<!-- Local image (optimized) -->
<Image src={heroImage} alt="Hero" />

<!-- With sizing -->
<Image
  src={heroImage}
  alt="Hero"
  width={800}
  height={600}
  quality={80}
/>

<!-- Remote image (must configure domains) -->
<Image
  src="https://example.com/image.jpg"
  alt="Remote"
  width={400}
  height={300}
/>
```

### Picture Component

```astro
---
import { Picture } from 'astro:assets';
import myImage from '../assets/image.png';
---

<Picture
  src={myImage}
  formats={['avif', 'webp']}
  alt="Responsive image"
/>
```

### Debug Component

```astro
---
import { Debug } from 'astro:components';
const data = { foo: 'bar' };
---

<Debug {data} />
```

### Code Component

```astro
---
import { Code } from 'astro:components';
---

<Code code={`const x = 1;`} lang="js" theme="dracula" />
```

---

## Component Composition

### Compound Components

```astro
---
// Tabs.astro
---
<div class="tabs" role="tablist">
  <slot />
</div>

---
// Tab.astro
interface Props {
  id: string;
  active?: boolean;
}
const { id, active = false } = Astro.props;
---
<button
  role="tab"
  id={`tab-${id}`}
  aria-selected={active}
  class:list={['tab', { active }]}
>
  <slot />
</button>

---
// Usage
---
<Tabs>
  <Tab id="1" active>First</Tab>
  <Tab id="2">Second</Tab>
  <Tab id="3">Third</Tab>
</Tabs>
```

### Wrapper Pattern

```astro
---
// Section.astro
interface Props {
  title: string;
  class?: string;
}
const { title, class: className } = Astro.props;
---

<section class:list={['section', className]}>
  <h2 class="section-title">{title}</h2>
  <div class="section-content">
    <slot />
  </div>
</section>
```
