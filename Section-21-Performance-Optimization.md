# Section 21: Performance Optimization

> **Critical Career Skill:** Performance optimization separates junior from mid-level developers. This section covers bundle analysis, code splitting, lazy loading, and advanced optimization techniques for production-grade SvelteKit applications.

## 📚 Learning Objectives

By the end of this section, you will be able to:

- Analyze and optimize bundle sizes
- Implement code splitting strategies
- Use lazy loading for routes and components
- Optimize images and assets
- Implement virtual scrolling for large lists
- Use Web Workers for CPU-intensive tasks
- Optimize database queries and API calls
- Implement caching strategies
- Use Lighthouse for performance auditing
- Achieve 90+ performance scores

---

## Table of Contents

- [Section 21: Performance Optimization](#section-21-performance-optimization)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Performance Fundamentals](#1-performance-fundamentals)
    - [Core Web Vitals](#core-web-vitals)
    - [Performance Budget](#performance-budget)
    - [Measurement Tools](#measurement-tools)
  - [2. Bundle Analysis \& Optimization](#2-bundle-analysis--optimization)
    - [Analyzing Your Bundle](#analyzing-your-bundle)
    - [Tree Shaking](#tree-shaking)
    - [Reducing Dependencies](#reducing-dependencies)
  - [3. Code Splitting \& Lazy Loading](#3-code-splitting--lazy-loading)
    - [Route-Based Code Splitting](#route-based-code-splitting)
    - [Component Lazy Loading](#component-lazy-loading)
    - [Dynamic Imports](#dynamic-imports)
  - [4. Image Optimization](#4-image-optimization)
    - [Modern Image Formats](#modern-image-formats)
    - [Responsive Images](#responsive-images)
    - [Lazy Loading Images](#lazy-loading-images)
  - [5. Rendering Optimization](#5-rendering-optimization)
    - [Virtual Scrolling](#virtual-scrolling)
    - [Debouncing \& Throttling](#debouncing--throttling)
    - [Memoization](#memoization)
  - [6. Network Optimization](#6-network-optimization)
    - [API Optimization](#api-optimization)
    - [Caching Strategies](#caching-strategies)
    - [Prefetching Data](#prefetching-data)
  - [7. Database Optimization](#7-database-optimization)
    - [Query Optimization](#query-optimization)
    - [Connection Pooling](#connection-pooling)
    - [Caching Database Results](#caching-database-results)
  - [8. Advanced Techniques](#8-advanced-techniques)
    - [Web Workers](#web-workers)
    - [Service Workers](#service-workers)
    - [Streaming SSR](#streaming-ssr)
  - [🎯 Practice Exercises](#-practice-exercises)
    - [Exercise 1: Bundle Analysis](#exercise-1-bundle-analysis)
    - [Exercise 2: Image Optimization](#exercise-2-image-optimization)
    - [Exercise 3: Virtual List](#exercise-3-virtual-list)
    - [Exercise 4: Performance Audit](#exercise-4-performance-audit)
  - [🎓 Summary](#-summary)

---

## 1. Performance Fundamentals

### Core Web Vitals

**Three Key Metrics (2026 Standards):**

1. **Largest Contentful Paint (LCP)** - Loading performance

   - **Goal:** < 2.5 seconds
   - Measures: Time until largest content element renders
   - Impact: User perceived load time

2. **Interaction to Next Paint (INP)** - Interactivity (replaced FID in 2024)

   - **Goal:** < 200ms
   - Measures: Responsiveness to user interactions
   - Impact: How quickly app responds to clicks/taps

3. **Cumulative Layout Shift (CLS)** - Visual stability
   - **Goal:** < 0.1
   - Measures: Unexpected layout shifts
   - Impact: Prevents accidental clicks

**Performance Scoring:**

```
Good:    LCP < 2.5s,  INP < 200ms,  CLS < 0.1
Needs Improvement: 2.5-4s, 200-500ms, 0.1-0.25
Poor:    > 4s,       > 500ms,      > 0.25
```

### Performance Budget

**Set Realistic Budgets:**

```typescript
// performance-budget.json
{
  "budgets": [
    {
      "resource": "script",
      "maximum": "300kb",  // JavaScript bundle
      "warning": "250kb"
    },
    {
      "resource": "style",
      "maximum": "50kb",   // CSS bundle
      "warning": "40kb"
    },
    {
      "resource": "image",
      "maximum": "500kb",  // Total images per page
      "warning": "400kb"
    },
    {
      "resource": "total",
      "maximum": "1000kb", // Total page weight
      "warning": "800kb"
    }
  ],
  "timings": [
    {
      "metric": "lcp",
      "maximum": 2500,
      "warning": 2000
    },
    {
      "metric": "inp",
      "maximum": 200,
      "warning": 150
    }
  ]
}
```

### Measurement Tools

**1. Lighthouse (Built into Chrome DevTools):**

```bash
# Run Lighthouse from CLI
npm install -g lighthouse
lighthouse https://your-site.com --view
```

**2. WebPageTest:**

- Visit: https://www.webpagetest.org
- Tests from multiple locations
- Filmstrip view of loading

**3. Chrome DevTools Performance Tab:**

```javascript
// Performance profiling in code
performance.mark("start-operation");
// ... expensive operation ...
performance.mark("end-operation");
performance.measure("operation", "start-operation", "end-operation");

// View results
const measures = performance.getEntriesByType("measure");
console.log(measures);
```

**4. Bundle Analyzer:**

```bash
npm install -D vite-bundle-analyzer
```

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import { sveltekit } from "@sveltejs/kit/vite";
import { analyzer } from "vite-bundle-analyzer";

export default defineConfig({
  plugins: [
    sveltekit(),
    analyzer({
      analyzerMode: "static",
      reportFilename: "bundle-report.html",
    }),
  ],
});
```

---

## 2. Bundle Analysis & Optimization

### Analyzing Your Bundle

**Build and analyze:**

```bash
npm run build
```

**Vite output example:**

```
vite v5.0.0 building for production...
✓ 125 modules transformed.
.svelte-kit/output/client/_app/immutable/
├─ entry/start-abc123.js      42.5 kB │ gzip: 12.3 kB
├─ entry/app-def456.js         38.2 kB │ gzip: 10.8 kB
├─ chunks/index-ghi789.js      156.3 kB │ gzip: 45.2 kB  ⚠️ Large!
└─ chunks/vendor-jkl012.js     89.1 kB │ gzip: 28.7 kB
```

**Identify large chunks:**

```bash
# Install bundle analyzer
npm install -D rollup-plugin-visualizer

# Add to vite.config.ts
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    sveltekit(),
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true
    })
  ]
});
```

### Tree Shaking

**✅ Import only what you need:**

```typescript
// ❌ Bad: Imports entire library (200kb)
import _ from "lodash";
const result = _.debounce(fn, 300);

// ✅ Good: Import only debounce (5kb)
import debounce from "lodash-es/debounce";
const result = debounce(fn, 300);

// ✅ Even better: Write your own (0.5kb)
function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout>;
  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}
```

**Configure for tree shaking** (`package.json`):

```json
{
  "sideEffects": false,
  "type": "module"
}
```

### Reducing Dependencies

**Audit dependencies:**

```bash
npm install -D depcheck
npx depcheck
```

**Replace heavy libraries:**

```typescript
// ❌ Moment.js: 67kb minified
import moment from "moment";
const formatted = moment().format("YYYY-MM-DD");

// ✅ Native Intl API: 0kb (built-in)
const formatted = new Intl.DateTimeFormat("en-US").format(new Date());

// ✅ Or date-fns: 2kb per function
import { format } from "date-fns";
const formatted = format(new Date(), "yyyy-MM-dd");
```

**Common replacements:**

- `moment` → `date-fns` or native `Intl`
- `lodash` → `lodash-es` (tree-shakeable) or native methods
- `axios` → native `fetch`
- `uuid` → `crypto.randomUUID()` (native)
- `classnames` → template literals
- `validator` → native regex

---

## 3. Code Splitting & Lazy Loading

### Route-Based Code Splitting

**SvelteKit automatically code-splits routes!**

```typescript
// Each route is a separate chunk
src/routes/
  +page.svelte           → chunk-home.js
  about/+page.svelte     → chunk-about.js
  blog/+page.svelte      → chunk-blog.js
  blog/[id]/+page.svelte → chunk-blog-post.js
```

**Verify code splitting:**

```bash
npm run build
# Check .svelte-kit/output/client/_app/immutable/chunks/
# Each route should have its own chunk
```

### Component Lazy Loading

**Lazy load heavy components:**

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import type { Component } from 'svelte';

	let { props }: { props: Record<string, any> } = $props();

	let ChartComponent: Component | null = $state(null);
	let loading = $state(true);

	onMount(async () => {
		// Only load chart library when component mounts
		const module = await import('./HeavyChart.svelte');
		ChartComponent = module.default;
		loading = false;
	});
</script>

{#if loading}
	<div class="skeleton">Loading chart...</div>
{:else if ChartComponent}
	<ChartComponent {...props} />
{/if}
```

> **Note:** In Svelte 5, you can use dynamic components directly without `<svelte:component this={...}>`. The component reference is simply used as a tag: `<ChartComponent />`.

**Conditional loading based on viewport:**

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import type { Component } from 'svelte';

	let visible = $state(false);
	let VideoPlayer: Component | null = $state(null);

	onMount(() => {
		const observer = new IntersectionObserver((entries) => {
			if (entries[0].isIntersecting) {
				visible = true;
				loadVideoPlayer();
				observer.disconnect();
			}
		});

		observer.observe(document.getElementById('video-container')!);
	});

	async function loadVideoPlayer() {
		const module = await import('./VideoPlayer.svelte');
		VideoPlayer = module.default;
	}
</script>

<div id="video-container">
	{#if VideoPlayer}
		<VideoPlayer />
	{:else}
		<div class="placeholder">Click to load video</div>
	{/if}
</div>
```

### Dynamic Imports

**Load libraries on-demand:**

```typescript
// ❌ Bad: Load PDF library on initial page load (500kb)
import { PDFDocument } from "pdf-lib";

export async function generatePDF() {
  const doc = await PDFDocument.create();
  // ...
}

// ✅ Good: Load only when user clicks "Download PDF"
export async function generatePDF() {
  const { PDFDocument } = await import("pdf-lib");
  const doc = await PDFDocument.create();
  // ...
}
```

**Example: Rich text editor:**

```svelte
<script lang="ts">
	import type { Component } from 'svelte';

	let Editor: Component | null = $state(null);
	let showEditor = $state(false);

	async function loadEditor() {
		const module = await import('$lib/components/RichTextEditor.svelte');
		Editor = module.default;
		showEditor = true;
	}
</script>

{#if showEditor && Editor}
	<Editor />
{:else}
	<button onclick={loadEditor}> Open Editor (loads 150kb on click) </button>
{/if}
```

---

## 4. Image Optimization

### Modern Image Formats

**Use WebP/AVIF with fallbacks:**

```svelte
<picture>
	<!-- Modern browsers: 50% smaller -->
	<source srcset="/images/hero.avif" type="image/avif" />
	<source srcset="/images/hero.webp" type="image/webp" />

	<!-- Fallback for old browsers -->
	<img src="/images/hero.jpg" alt="Hero image" width="1200" height="600" loading="lazy" />
</picture>
```

**Automatic conversion with Sharp:**

```typescript
// src/lib/server/imageOptimizer.ts
import sharp from "sharp";
import { writeFile } from "fs/promises";

export async function optimizeImage(inputPath: string, outputPath: string) {
  const image = sharp(inputPath);

  // Generate multiple formats
  await Promise.all([
    // AVIF (smallest, best quality)
    image
      .clone()
      .avif({ quality: 80 })
      .toFile(outputPath.replace(/\.\w+$/, ".avif")),

    // WebP (good fallback)
    image
      .clone()
      .webp({ quality: 85 })
      .toFile(outputPath.replace(/\.\w+$/, ".webp")),

    // Optimized JPEG (universal fallback)
    image.clone().jpeg({ quality: 80, progressive: true }).toFile(outputPath),
  ]);
}
```

### Responsive Images

**Serve appropriate sizes:**

```svelte
<img
	srcset="
    /images/product-400.webp 400w,
    /images/product-800.webp 800w,
    /images/product-1200.webp 1200w,
    /images/product-1600.webp 1600w
  "
	sizes="
    (max-width: 640px) 100vw,
    (max-width: 1024px) 50vw,
    33vw
  "
	src="/images/product-800.webp"
	alt="Product"
	width="800"
	height="600"
	loading="lazy"
/>
```

**Generate responsive images:**

```typescript
// scripts/generateResponsiveImages.ts
import sharp from "sharp";
import { readdir } from "fs/promises";

const SIZES = [400, 800, 1200, 1600];

async function generateResponsive() {
  const files = await readdir("static/images/originals");

  for (const file of files) {
    const input = `static/images/originals/${file}`;

    for (const size of SIZES) {
      await sharp(input)
        .resize(size, null, { withoutEnlargement: true })
        .webp({ quality: 85 })
        .toFile(`static/images/${file.replace(/\.\w+$/, `-${size}.webp`)}`);
    }
  }
}

generateResponsive();
```

### Lazy Loading Images

**Native lazy loading:**

```svelte
<!-- ✅ Lazy load images below fold -->
<img
	src="/images/product.webp"
	alt="Product"
	loading="lazy"
	decoding="async"
	width="400"
	height="300"
/>

<!-- ⚡ Eager load above-the-fold images -->
<img
	src="/images/hero.webp"
	alt="Hero"
	loading="eager"
	fetchpriority="high"
	width="1200"
	height="600"
/>
```

**Blur placeholder technique:**

```svelte
<script lang="ts">
	let loaded = $state(false);

	const blurDataUrl =
		"data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 400 300'%3E%3Cfilter id='b' color-interpolation-filters='sRGB'%3E%3CfeGaussianBlur stdDeviation='20'/%3E%3C/filter%3E%3Cimage filter='url(%23b)' x='0' y='0' height='100%25' width='100%25' href='data:image/jpeg;base64,/9j...'/%3E%3C/svg%3E";
</script>

<div class="image-container">
	<!-- Blur placeholder -->
	{#if !loaded}
		<img src={blurDataUrl} alt="" class="blur-placeholder" aria-hidden="true" />
	{/if}

	<!-- Actual image -->
	<img
		src="/images/product.webp"
		alt="Product"
		loading="lazy"
		onload={() => (loaded = true)}
		class:loaded
	/>
</div>

<style>
	.image-container {
		position: relative;
		overflow: hidden;
	}

	.blur-placeholder {
		position: absolute;
		inset: 0;
		filter: blur(20px);
	}

	img {
		opacity: 0;
		transition: opacity 0.3s;
	}

	img.loaded {
		opacity: 1;
	}
</style>
```

---

## 5. Rendering Optimization

### Virtual Scrolling

**For large lists (1000+ items):**

```svelte
<script lang="ts">
	import { onMount } from 'svelte';

	let { items = [] }: { items: any[] } = $props();

	let containerHeight = $state(600);
	let scrollTop = $state(0);
	let itemHeight = 50;

	// Calculate visible range
	let startIndex = $derived(Math.floor(scrollTop / itemHeight));
	let endIndex = $derived(
		Math.min(startIndex + Math.ceil(containerHeight / itemHeight) + 1, items.length)
	);

	let visibleItems = $derived(items.slice(startIndex, endIndex));
	let offsetY = $derived(startIndex * itemHeight);

	function handleScroll(e: Event) {
		scrollTop = (e.target as HTMLElement).scrollTop;
	}
</script>

<div class="virtual-list" style="height: {containerHeight}px" onscroll={handleScroll}>
	<!-- Total height spacer -->
	<div style="height: {items.length * itemHeight}px; position: relative;">
		<!-- Visible items -->
		<div style="transform: translateY({offsetY}px);">
			{#each visibleItems as item (item.id)}
				<div class="item" style="height: {itemHeight}px;">
					{item.name}
				</div>
			{/each}
		</div>
	</div>
</div>

<style>
	.virtual-list {
		overflow-y: auto;
		border: 1px solid #ccc;
	}

	.item {
		padding: 1rem;
		border-bottom: 1px solid #eee;
	}
</style>
```

**Using `svelte-virtual-list` library (Svelte 5 compatible):**

```bash
npm install svelte-virtual-list
```

```svelte
<script lang="ts">
	import VirtualList from 'svelte-virtual-list';

	let items = $state(Array.from({ length: 10000 }, (_, i) => ({
		id: i,
		name: `Item ${i}`
	})));
</script>

<VirtualList {items} height="600px" itemHeight={50}>
	{#snippet item(data)}
		<div class="item">
			{data.name}
		</div>
	{/snippet}
</VirtualList>
```

> **Note:** Check `svelte-virtual-list` documentation for the latest Svelte 5 snippet API. If the library hasn't been updated yet, you can use a native implementation:

**Native Svelte 5 Virtual Scrolling:**

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';

	let {
		items,
		itemHeight = 50,
		height = '400px',
		row
	}: {
		items: any[];
		itemHeight?: number;
		height?: string;
		row: Snippet<[{ item: any; index: number }]>;
	} = $props();

	let scrollTop = $state(0);
	let containerHeight = $state(0);

	let visibleStart = $derived(Math.floor(scrollTop / itemHeight));
	let visibleCount = $derived(Math.ceil(containerHeight / itemHeight) + 1);
	let visibleItems = $derived(items.slice(visibleStart, visibleStart + visibleCount));
	let totalHeight = $derived(items.length * itemHeight);

	function handleScroll(e: Event) {
		scrollTop = (e.target as HTMLElement).scrollTop;
	}
</script>

<div
	class="virtual-container"
	style:height
	onscroll={handleScroll}
	bind:clientHeight={containerHeight}
>
	<div class="virtual-spacer" style:height="{totalHeight}px">
		<div class="virtual-items" style:transform="translateY({visibleStart * itemHeight}px)">
			{#each visibleItems as item, i (item.id ?? visibleStart + i)}
				{@render row({ item, index: visibleStart + i })}
			{/each}
		</div>
	</div>
</div>

<style>
	.virtual-container {
		overflow-y: auto;
	}
	.virtual-spacer {
		position: relative;
	}
	.virtual-items {
		position: absolute;
		top: 0;
		left: 0;
		right: 0;
	}
</style>
```

**Using the native implementation:**

```svelte
<script lang="ts">
	import VirtualScroll from '$lib/components/VirtualScroll.svelte';

	let items = $state(Array.from({ length: 10000 }, (_, i) => ({
		id: i,
		name: `Item ${i}`
	})));
</script>

<VirtualScroll {items} height="600px" itemHeight={50}>
	{#snippet row({ item })}
		<div class="p-4 border-b border-gray-200">
			{item.name}
		</div>
	{/snippet}
</VirtualScroll>
```

### Debouncing & Throttling

**Debounce search input:**

```svelte
<script lang="ts">
	import { debounce } from '$lib/utils/performance';

	let searchQuery = $state('');
	let results = $state<any[]>([]);

	// Only search after user stops typing for 300ms
	const debouncedSearch = debounce(async (query: string) => {
		if (query.length < 2) return;

		const res = await fetch(`/api/search?q=${query}`);
		results = await res.json();
	}, 300);

	$effect(() => {
		debouncedSearch(searchQuery);
	});
</script>

<input type="search" bind:value={searchQuery} placeholder="Search..." />

{#each results as result}
	<div>{result.title}</div>
{/each}
```

**Throttle scroll events:**

```typescript
// src/lib/utils/performance.ts

export function debounce<T extends (...args: any[]) => any>(
  fn: T,
  delay: number
): (...args: Parameters<T>) => void {
  let timeoutId: ReturnType<typeof setTimeout>;

  return (...args) => {
    clearTimeout(timeoutId);
    timeoutId = setTimeout(() => fn(...args), delay);
  };
}

export function throttle<T extends (...args: any[]) => any>(
  fn: T,
  limit: number
): (...args: Parameters<T>) => void {
  let inThrottle: boolean;

  return (...args) => {
    if (!inThrottle) {
      fn(...args);
      inThrottle = true;
      setTimeout(() => (inThrottle = false), limit);
    }
  };
}
```

```svelte
<script lang="ts">
	import { throttle } from '$lib/utils/performance';

	let scrollY = $state(0);

	// Only update every 100ms
	const handleScroll = throttle(() => {
		scrollY = window.scrollY;
	}, 100);

	onMount(() => {
		window.addEventListener('scroll', handleScroll);
		return () => window.removeEventListener('scroll', handleScroll);
	});
</script>
```

### Memoization

**Cache expensive calculations:**

```typescript
// src/lib/utils/memoize.ts

export function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map();

  return ((...args: any[]) => {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}
```

```typescript
// Example: Expensive Fibonacci calculation
const fibonacci = memoize((n: number): number => {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
});

// First call: ~500ms for fib(40)
console.time("first");
fibonacci(40); // Calculates
console.timeEnd("first");

// Second call: < 1ms (cached)
console.time("cached");
fibonacci(40); // Returns cached result
console.timeEnd("cached");
```

**Memoize in components:**

```svelte
<script lang="ts">
	import { memoize } from '$lib/utils/memoize';

	let { data } = $props();

	// Expensive computation only runs when data changes
	const processData = memoize((input: any[]) => {
		console.log('Processing data...');
		return input.map((item) => ({
			...item,
			computed: expensiveCalculation(item)
		}));
	});

	let processed = $derived(processData(data));
</script>

{#each processed as item}
	<div>{item.computed}</div>
{/each}
```

---

## 6. Network Optimization

### API Optimization

**Reduce API calls:**

```typescript
// ❌ Bad: Multiple sequential calls
async function loadUserData(userId: string) {
  const user = await fetch(`/api/users/${userId}`).then((r) => r.json());
  const posts = await fetch(`/api/users/${userId}/posts`).then((r) => r.json());
  const comments = await fetch(`/api/users/${userId}/comments`).then((r) =>
    r.json()
  );

  return { user, posts, comments };
}

// ✅ Good: Parallel requests
async function loadUserData(userId: string) {
  const [user, posts, comments] = await Promise.all([
    fetch(`/api/users/${userId}`).then((r) => r.json()),
    fetch(`/api/users/${userId}/posts`).then((r) => r.json()),
    fetch(`/api/users/${userId}/comments`).then((r) => r.json()),
  ]);

  return { user, posts, comments };
}

// ✅ Best: Single aggregated endpoint
async function loadUserData(userId: string) {
  return fetch(`/api/users/${userId}/dashboard`).then((r) => r.json());
  // Returns { user, posts, comments } in one request
}
```

**Implement pagination:**

```typescript
// API endpoint with cursor pagination
export const GET: RequestHandler = async ({ url }) => {
  const cursor = url.searchParams.get("cursor");
  const limit = 20;

  const posts = await db
    .select()
    .from(posts)
    .where(cursor ? gt(posts.id, cursor) : undefined)
    .limit(limit)
    .orderBy(desc(posts.createdAt));

  return json({
    posts,
    nextCursor: posts.length === limit ? posts[posts.length - 1].id : null,
  });
};
```

### Caching Strategies

**HTTP caching headers:**

```typescript
// src/routes/api/posts/+server.ts
import { json } from "@sveltejs/kit";

export const GET: RequestHandler = async () => {
  const posts = await db.select().from(posts);

  return json(posts, {
    headers: {
      // Cache for 5 minutes
      "Cache-Control": "public, max-age=300, s-maxage=300",

      // Revalidate in background
      "Cache-Control": "public, max-age=300, stale-while-revalidate=86400",
    },
  });
};
```

**In-memory caching:**

```typescript
// src/lib/server/cache.ts
const cache = new Map<string, { data: any; expires: number }>();

export function getCached<T>(
  key: string,
  ttl: number,
  fetcher: () => Promise<T>
): Promise<T> {
  const cached = cache.get(key);

  if (cached && cached.expires > Date.now()) {
    return Promise.resolve(cached.data);
  }

  return fetcher().then((data) => {
    cache.set(key, {
      data,
      expires: Date.now() + ttl,
    });
    return data;
  });
}
```

```typescript
// Usage in load function
export const load: PageServerLoad = async () => {
  return {
    posts: await getCached(
      "posts:all",
      5 * 60 * 1000, // 5 minutes
      () => db.select().from(posts)
    ),
  };
};
```

### Prefetching Data

**SvelteKit prefetching:**

```svelte
<!-- Prefetch on hover -->
<a href="/blog" data-sveltekit-preload-data="hover"> Blog </a>

<!-- Prefetch when visible -->
<a href="/about" data-sveltekit-preload-data="viewport"> About </a>

<!-- Prefetch immediately -->
<a href="/products" data-sveltekit-preload-data> Products </a>
```

**Prefetch critical API calls:**

```svelte
<script lang="ts">
	import { onMount } from 'svelte';

	let prefetchedData = $state(null);

	onMount(() => {
		// Prefetch data for next likely page
		fetch('/api/products?featured=true')
			.then((r) => r.json())
			.then((data) => (prefetchedData = data));
	});
</script>
```

---

## 7. Database Optimization

### Query Optimization

**Add indexes for common queries:**

```typescript
// schema.ts
import { pgTable, serial, text, timestamp, index } from "drizzle-orm/pg-core";

export const posts = pgTable(
  "posts",
  {
    id: serial("id").primaryKey(),
    title: text("title").notNull(),
    userId: serial("user_id").notNull(),
    createdAt: timestamp("created_at").defaultNow(),
    status: text("status").notNull(),
  },
  (table) => ({
    // Indexes for common queries
    userIdIdx: index("user_id_idx").on(table.userId),
    statusIdx: index("status_idx").on(table.status),
    createdAtIdx: index("created_at_idx").on(table.createdAt),
  })
);
```

**Optimize queries:**

```typescript
// ❌ Bad: N+1 query problem
const users = await db.select().from(users);
for (const user of users) {
  user.posts = await db.select().from(posts).where(eq(posts.userId, user.id));
}

// ✅ Good: Single query with join
const usersWithPosts = await db
  .select()
  .from(users)
  .leftJoin(posts, eq(users.id, posts.userId));
```

**Select only needed columns:**

```typescript
// ❌ Bad: Select all columns (wasteful)
const posts = await db.select().from(posts);

// ✅ Good: Select only needed columns
const posts = await db
  .select({
    id: posts.id,
    title: posts.title,
    excerpt: posts.excerpt,
  })
  .from(posts);
```

### Connection Pooling

**Configure connection pool:**

```typescript
// src/lib/server/db.ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";

const client = postgres(process.env.DATABASE_URL!, {
  // Connection pooling
  max: 10, // Maximum connections
  idle_timeout: 20, // Close idle connections after 20s
  connect_timeout: 10, // Timeout for new connections

  // Performance
  prepare: true, // Use prepared statements
  types: {
    // Custom type parsing for better performance
    bigint: postgres.BigInt,
  },
});

export const db = drizzle(client);
```

### Caching Database Results

**Redis caching layer:**

```typescript
// src/lib/server/redis.ts
import { Redis } from "@upstash/redis";

export const redis = new Redis({
  url: process.env.UPSTASH_REDIS_URL!,
  token: process.env.UPSTASH_REDIS_TOKEN!,
});

export async function getCachedOrFetch<T>(
  key: string,
  fetcher: () => Promise<T>,
  ttl: number = 300
): Promise<T> {
  // Try cache first
  const cached = await redis.get(key);
  if (cached) return cached as T;

  // Fetch from database
  const data = await fetcher();

  // Cache for next time
  await redis.setex(key, ttl, JSON.stringify(data));

  return data;
}
```

```typescript
// Usage in API route
export const GET: RequestHandler = async ({ params }) => {
  const post = await getCachedOrFetch(
    `post:${params.id}`,
    () => db.select().from(posts).where(eq(posts.id, params.id)),
    600 // Cache for 10 minutes
  );

  return json(post);
};
```

---

## 8. Advanced Techniques

### Web Workers

**Offload CPU-intensive tasks:**

```typescript
// src/lib/workers/dataProcessor.worker.ts
self.addEventListener("message", (e) => {
  const { data, operation } = e.data;

  let result;

  switch (operation) {
    case "processLargeDataset":
      result = processLargeDataset(data);
      break;
    case "calculateStatistics":
      result = calculateStatistics(data);
      break;
  }

  self.postMessage(result);
});

function processLargeDataset(data: any[]) {
  // CPU-intensive work happens in worker thread
  return data.map((item) => ({
    ...item,
    computed: expensiveCalculation(item),
  }));
}
```

```svelte
<script lang="ts">
	import { onMount } from 'svelte';

	let result = $state(null);
	let processing = $state(false);

	let worker: Worker;

	onMount(() => {
		worker = new Worker(new URL('$lib/workers/dataProcessor.worker.ts', import.meta.url), {
			type: 'module'
		});

		worker.addEventListener('message', (e) => {
			result = e.data;
			processing = false;
		});

		return () => worker.terminate();
	});

	function processData(data: any[]) {
		processing = true;
		worker.postMessage({ data, operation: 'processLargeDataset' });
	}
</script>

<button onclick={() => processData(largeDataset)} disabled={processing}>
	{processing ? 'Processing...' : 'Process Data'}
</button>
```

### Service Workers

**Offline support and caching:**

```typescript
// src/service-worker.ts
import { build, files, version } from "$service-worker";

const CACHE = `cache-${version}`;
const ASSETS = [...build, ...files];

// Install: Cache all assets
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches
      .open(CACHE)
      .then((cache) => cache.addAll(ASSETS))
      .then(() => self.skipWaiting())
  );
});

// Activate: Clean up old caches
self.addEventListener("activate", (event) => {
  event.waitUntil(
    caches
      .keys()
      .then((keys) =>
        Promise.all(
          keys.filter((key) => key !== CACHE).map((key) => caches.delete(key))
        )
      )
      .then(() => self.clients.claim())
  );
});

// Fetch: Serve from cache, fallback to network
self.addEventListener("fetch", (event) => {
  if (event.request.method !== "GET") return;

  event.respondWith(
    caches.match(event.request).then((cached) => cached || fetch(event.request))
  );
});
```

### Streaming SSR

**Stream large pages:**

```typescript
// src/routes/feed/+page.server.ts
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async () => {
  return {
    // Stream the posts as they're ready
    streamed: {
      posts: fetchPosts(), // Returns a Promise
      suggestions: fetchSuggestions(), // Returns a Promise
    },
  };
};
```

```svelte
<script lang="ts">
	let { data } = $props();
</script>

<!-- Shows immediately -->
<h1>Your Feed</h1>

<!-- Streams in when ready -->
{#await data.streamed.posts}
	<div class="skeleton">Loading posts...</div>
{:then posts}
	{#each posts as post}
		<PostCard {post} />
	{/each}
{/await}

<!-- Streams in independently -->
{#await data.streamed.suggestions}
	<div class="skeleton">Loading suggestions...</div>
{:then suggestions}
	<SuggestionsList {suggestions} />
{/await}
```

---

## 🎯 Practice Exercises

### Exercise 1: Bundle Analysis

1. Run bundle analyzer on your project
2. Identify the 3 largest chunks
3. Reduce at least one chunk by 30%+

### Exercise 2: Image Optimization

1. Convert all images to WebP/AVIF
2. Implement responsive images with srcset
3. Add lazy loading to below-fold images
4. Measure before/after page weight

### Exercise 3: Virtual List

Build a virtual scrolling list that:

- Handles 10,000+ items smoothly
- Maintains 60fps scrolling
- Works on mobile devices
- Preserves scroll position on refresh

### Exercise 4: Performance Audit

1. Run Lighthouse on your app
2. Achieve 90+ performance score
3. Fix all Core Web Vitals issues
4. Document optimizations made

---

## 🎓 Summary

You now know how to:

- ✅ Analyze and optimize bundle sizes
- ✅ Implement code splitting and lazy loading
- ✅ Optimize images with modern formats
- ✅ Use virtual scrolling for large lists
- ✅ Debounce and throttle expensive operations
- ✅ Implement effective caching strategies
- ✅ Optimize database queries and connections
- ✅ Use Web Workers for CPU-intensive tasks
- ✅ Measure and improve Core Web Vitals
- ✅ Achieve 90+ Lighthouse scores

**Performance Checklist:**

- [ ] Bundle size < 300kb gzipped
- [ ] LCP < 2.5s
- [ ] INP < 200ms
- [ ] CLS < 0.1
- [ ] All images optimized (WebP/AVIF)
- [ ] Lazy loading implemented
- [ ] Database queries indexed
- [ ] API responses cached
- [ ] Critical CSS inlined
- [ ] Lighthouse score 90+

**Career Impact:**
Performance optimization is what separates junior from mid-level developers. Companies lose 7% conversion rate for every 100ms delay. These skills make you valuable and demonstrate senior-level thinking.
