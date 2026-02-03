---
name: islands
description: Astro islands architecture and hydration patterns. Use when implementing interactive components with React, Vue, or Svelte.
---

# Islands Architecture Skill

## Contents

- [Overview](#overview)
- [Client Directives](#client-directives)
- [Framework Integration](#framework-integration)
- [State Management](#state-management)
- [Communication Between Islands](#communication-between-islands)
- [Performance Optimization](#performance-optimization)
- [Common Patterns](#common-patterns)

---

## Overview

Islands architecture allows you to add interactive "islands" of JavaScript within a sea of static HTML. Benefits:

- Ship zero JS by default
- Hydrate only what needs interactivity
- Mix multiple frameworks in one page
- Automatic code splitting per island

---

## Client Directives

| Directive | Hydration Timing | Bundle Impact | Use Case |
|-----------|-----------------|---------------|----------|
| `client:load` | Immediately | Highest | Critical interactions (nav, forms) |
| `client:idle` | After requestIdleCallback | Medium | Non-critical widgets |
| `client:visible` | When in viewport | Lowest | Below-fold content |
| `client:media` | When query matches | Conditional | Device-specific features |
| `client:only` | Client-only (no SSR) | Highest | Browser-only code |

### Usage Examples

```astro
---
import Navigation from './Navigation.tsx';
import Comments from './Comments.tsx';
import MobileMenu from './MobileMenu.vue';
import HeavyChart from './Chart.svelte';
import BrowserOnlyMap from './Map.tsx';
---

<!-- Immediate - above the fold, critical -->
<Navigation client:load />

<!-- Idle - important but not critical -->
<Comments client:idle postId={post.id} />

<!-- Visible - lazy load when scrolled into view -->
<HeavyChart client:visible data={chartData} />

<!-- Media - only load on mobile -->
<MobileMenu client:media="(max-width: 768px)" />

<!-- Only - no SSR, browser-specific APIs -->
<BrowserOnlyMap client:only="react" location={coords} />
```

---

## Framework Integration

### Configuration

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import vue from '@astrojs/vue';
import svelte from '@astrojs/svelte';

export default defineConfig({
  integrations: [
    react(),
    vue(),
    svelte(),
  ],
});
```

### React Island

```tsx
// src/components/islands/Counter.tsx
import { useState } from 'react';

interface Props {
  initialCount?: number;
  onCountChange?: (count: number) => void;
}

export default function Counter({ initialCount = 0 }: Props) {
  const [count, setCount] = useState(initialCount);

  return (
    <div className="flex items-center gap-4">
      <button
        onClick={() => setCount(c => c - 1)}
        className="px-4 py-2 bg-gray-200 rounded"
      >
        -
      </button>
      <span className="text-xl font-bold">{count}</span>
      <button
        onClick={() => setCount(c => c + 1)}
        className="px-4 py-2 bg-gray-200 rounded"
      >
        +
      </button>
    </div>
  );
}
```

### Vue Island

```vue
<!-- src/components/islands/SearchBox.vue -->
<script setup lang="ts">
import { ref, watch } from 'vue';

interface Props {
  placeholder?: string;
}

const props = withDefaults(defineProps<Props>(), {
  placeholder: 'Search...',
});

const emit = defineEmits<{
  (e: 'search', query: string): void;
}>();

const query = ref('');

watch(query, (value) => {
  emit('search', value);
});
</script>

<template>
  <input
    v-model="query"
    type="search"
    :placeholder="placeholder"
    class="px-4 py-2 border rounded-lg"
  />
</template>
```

### Svelte Island

```svelte
<!-- src/components/islands/Toggle.svelte -->
<script lang="ts">
  export let checked = false;
  export let label: string;

  function toggle() {
    checked = !checked;
  }
</script>

<button
  on:click={toggle}
  class="toggle"
  class:active={checked}
  role="switch"
  aria-checked={checked}
>
  {label}: {checked ? 'On' : 'Off'}
</button>

<style>
  .toggle {
    padding: 0.5rem 1rem;
    border-radius: 0.5rem;
    background: #e5e7eb;
  }
  .toggle.active {
    background: #3b82f6;
    color: white;
  }
</style>
```

---

## State Management

### Local State (Simple)

Use framework's built-in state for isolated islands:
- React: `useState`, `useReducer`
- Vue: `ref`, `reactive`
- Svelte: `$state` (runes) or reactive declarations

### Shared State with Nanostores

For cross-island state, use [nanostores](https://github.com/nanostores/nanostores):

```typescript
// src/stores/cart.ts
import { atom, map } from 'nanostores';

export interface CartItem {
  id: string;
  name: string;
  quantity: number;
  price: number;
}

// Simple atom
export const isCartOpen = atom(false);

// Map for complex state
export const cartItems = map<Record<string, CartItem>>({});

// Computed values
export function getCartTotal() {
  return Object.values(cartItems.get()).reduce(
    (sum, item) => sum + item.price * item.quantity,
    0
  );
}

// Actions
export function addToCart(item: Omit<CartItem, 'quantity'>) {
  const existing = cartItems.get()[item.id];
  if (existing) {
    cartItems.setKey(item.id, { ...existing, quantity: existing.quantity + 1 });
  } else {
    cartItems.setKey(item.id, { ...item, quantity: 1 });
  }
}

export function removeFromCart(id: string) {
  const current = { ...cartItems.get() };
  delete current[id];
  cartItems.set(current);
}
```

### React Integration

```tsx
// src/components/islands/CartButton.tsx
import { useStore } from '@nanostores/react';
import { cartItems, isCartOpen } from '../../stores/cart';

export default function CartButton() {
  const $cartItems = useStore(cartItems);
  const $isOpen = useStore(isCartOpen);

  const itemCount = Object.values($cartItems).reduce(
    (sum, item) => sum + item.quantity,
    0
  );

  return (
    <button onClick={() => isCartOpen.set(!$isOpen)}>
      Cart ({itemCount})
    </button>
  );
}
```

### Vue Integration

```vue
<!-- src/components/islands/CartDrawer.vue -->
<script setup lang="ts">
import { useStore } from '@nanostores/vue';
import { cartItems, isCartOpen, removeFromCart } from '../../stores/cart';

const $cartItems = useStore(cartItems);
const $isOpen = useStore(isCartOpen);
</script>

<template>
  <aside v-if="$isOpen" class="cart-drawer">
    <div v-for="item in Object.values($cartItems)" :key="item.id">
      {{ item.name }} x {{ item.quantity }}
      <button @click="removeFromCart(item.id)">Remove</button>
    </div>
  </aside>
</template>
```

---

## Communication Between Islands

### Via Nanostores (Recommended)

```typescript
// Islands update shared store, others react automatically
addToCart(item);  // From any island
// All islands subscribed to cartItems re-render
```

### Via Custom Events

```tsx
// Island A - Dispatch
function ProductCard({ product }) {
  const handleAdd = () => {
    window.dispatchEvent(
      new CustomEvent('add-to-cart', { detail: product })
    );
  };
  return <button onClick={handleAdd}>Add to Cart</button>;
}

// Island B - Listen
function CartWidget() {
  const [items, setItems] = useState<Product[]>([]);

  useEffect(() => {
    const handler = (e: CustomEvent) => {
      setItems(prev => [...prev, e.detail]);
    };
    window.addEventListener('add-to-cart', handler);
    return () => window.removeEventListener('add-to-cart', handler);
  }, []);

  return <span>Cart: {items.length}</span>;
}
```

### Via URL State

```tsx
// Good for filters, pagination, search
function FilterPanel() {
  const searchParams = new URLSearchParams(window.location.search);

  const updateFilter = (key: string, value: string) => {
    searchParams.set(key, value);
    window.history.pushState({}, '', `?${searchParams}`);
    window.dispatchEvent(new PopStateEvent('popstate'));
  };
}
```

---

## Performance Optimization

### Minimize Island Size

```astro
---
// Split interactive parts from static
import ProductImage from './ProductImage.astro';  // Static
import AddToCartButton from './AddToCartButton.tsx';  // Interactive
---

<!-- Static wrapper, small interactive island -->
<article class="product-card">
  <ProductImage src={product.image} alt={product.name} />
  <h2>{product.name}</h2>
  <p>{product.description}</p>
  <AddToCartButton client:visible productId={product.id} />
</article>
```

### Lazy Load Heavy Dependencies

```tsx
// Don't import heavy libraries at top level
import { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));

export default function ChartContainer({ data }) {
  return (
    <Suspense fallback={<div>Loading chart...</div>}>
      <HeavyChart data={data} />
    </Suspense>
  );
}
```

### Use `client:visible` for Below-Fold

```astro
<!-- These won't load until scrolled into view -->
<Comments client:visible postId={post.id} />
<RelatedProducts client:visible productId={product.id} />
<NewsletterSignup client:visible />
```

---

## Common Patterns

### Modal/Dialog

```astro
---
import Modal from './Modal.tsx';
---

<!-- Modal trigger doesn't need JS -->
<button id="open-modal">Open Modal</button>

<!-- Modal island handles all interactivity -->
<Modal client:idle triggerId="open-modal">
  <h2>Modal Content</h2>
  <p>This is rendered on the server but interactive on client.</p>
</Modal>
```

### Form with Validation

```astro
---
import ContactForm from './ContactForm.tsx';
---

<!-- Form island handles validation and submission -->
<ContactForm
  client:load
  action="/api/contact"
  successRedirect="/thank-you"
/>
```

### Search with Results

```astro
---
import SearchInput from './SearchInput.tsx';
import SearchResults from './SearchResults.tsx';
---

<div class="search-container">
  <SearchInput client:load />
  <SearchResults client:idle />
</div>
```

---

## Best Practices

| Area | Recommendation |
|------|----------------|
| **Default to static** | Only add `client:*` when interactivity is needed |
| **Smallest possible island** | Extract static parts to Astro components |
| **Choose directive wisely** | Match directive to actual need |
| **Shared state** | Use nanostores for cross-island communication |
| **Heavy libraries** | Lazy load or use `client:visible` |
| **Accessibility** | Ensure islands work with JS disabled (progressive) |
| **Testing** | Test islands independently from Astro |
