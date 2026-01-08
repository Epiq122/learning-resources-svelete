# Section 18: Rendering Strategies & Pre-rendering

## 📚 Learning Objectives

By the end of this section, you will:

- Understand SSR, CSR, and SSG rendering strategies
- Configure trailing slash behavior
- Pre-render static pages at build time
- Pre-render API endpoints
- Generate pages with dynamic data
- Pre-render routes with dynamic parameters
- Add dynamic content to pre-rendered pages
- Build and preview production apps
- Optimize performance with the right rendering strategy

---

## Table of Contents

- [Section 18: Rendering Strategies \& Pre-rendering](#section-18-rendering-strategies--pre-rendering)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. SSR, CSR \& Trailing Slashes](#1-ssr-csr--trailing-slashes)
    - [Understanding Rendering Modes](#understanding-rendering-modes)
  - [2. Pre-rendering Routes \& Building the App](#2-pre-rendering-routes--building-the-app)
    - [Static Site Generation](#static-site-generation)
  - [3. Pre-rendering Endpoints](#3-pre-rendering-endpoints)
    - [Static API Responses](#static-api-responses)
  - [4. Pre-rendering with Dynamic Data](#4-pre-rendering-with-dynamic-data)
    - [Database Data at Build Time](#database-data-at-build-time)
  - [5. Pre-rendering Dynamic Parameters](#5-pre-rendering-dynamic-parameters)
    - [Generating Multiple Static Pages](#generating-multiple-static-pages)
  - [6. Adding Dynamic Content to Pre-rendered Pages](#6-adding-dynamic-content-to-pre-rendered-pages)
    - [Hybrid Rendering Approach](#hybrid-rendering-approach)
  - [7. Complete Example: Documentation Site with Blog](#7-complete-example-documentation-site-with-blog)
    - [Full Pre-rendering Strategy](#full-pre-rendering-strategy)
    - [📁 Project Structure](#-project-structure)
    - [Configuration](#configuration)
    - [Database Schema](#database-schema)
    - [Documentation Pages](#documentation-pages)
    - [Blog List](#blog-list)
    - [Blog Post](#blog-post)
    - [Robots.txt](#robotstxt)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. SSR, CSR & Trailing Slashes

### Understanding Rendering Modes

SvelteKit supports multiple rendering strategies for different use cases.

**Rendering Modes:**

```typescript
// src/routes/+page.ts (or +page.server.ts)

// 1. SSR (Server-Side Rendering) - Default
// Page rendered on the server for each request
export const ssr = true; // Default, can omit

// 2. CSR (Client-Side Rendering)
// Page rendered in the browser
export const csr = true; // Default, can omit

// 3. Pre-rendering (SSG - Static Site Generation)
// Page rendered at build time
export const prerender = true;

// 4. Disable both (SPA mode for this route)
export const ssr = false;
export const csr = true;
```

**Real-World Scenarios:**

```typescript
// Marketing pages - Pre-render for speed
// Marketing pages - Pre-render for speed and SEO
// src/routes/(marketing)/about/+page.ts
// prerender = true: Page HTML generated at BUILD time, not request time
// Perfect for static content that rarely changes
export const prerender = true;

// Dashboard - SSR for fresh data + CSR for interactivity
// src/routes/(app)/dashboard/+page.ts
// ssr = true: Server renders on each request (default)
// csr = true: Hydrates on client for interactivity (default)
// Best for dynamic, user-specific data
export const ssr = true;
export const csr = true;

// Admin panel - CSR only (SPA mode)
// src/routes/(admin)/+layout.ts
// ssr = false: No server rendering, client-only
// csr = true: Pure SPA behavior
// Use when heavy client state or not concerned with SEO
export const ssr = false;
export const csr = true;

// API endpoint - Pre-render for static data
// src/routes/api/config/+server.ts
export const prerender = true;
```

**Trailing Slash Configuration:**

```typescript
// svelte.config.js
import adapter from "@sveltejs/adapter-auto";

/** @type {import('@sveltejs/kit').Config} */
const config = {
  kit: {
    adapter: adapter(),

    // Trailing slash options:
    // 'never' - /about (default)
    // 'always' - /about/
    // 'ignore' - both work
    trailingSlash: "never",

    // Pre-render configuration
    prerender: {
      entries: ["*"], // Crawl all pages
      crawl: true, // Follow links to find pages
      handleHttpError: "warn", // How to handle errors
      handleMissingId: "warn",
    },
  },
};

export default config;
```

**Per-route trailing slash:**

```typescript
// src/routes/blog/+page.ts
export const trailingSlash = "always"; // /blog/ only
```

> 💡 **Best Practice**: Choose one trailing slash strategy and stick with it for SEO consistency.

**Comparison Table:**

| Strategy       | When to Use           | Pros                     | Cons                        |
| -------------- | --------------------- | ------------------------ | --------------------------- |
| **SSR**        | Dynamic content, auth | Fresh data, SEO-friendly | Server load                 |
| **CSR**        | Authenticated apps    | Rich interactivity       | No SEO, slower initial load |
| **Pre-render** | Static content        | Fastest, cheap hosting   | Build time, stale data      |
| **SPA mode**   | Admin panels          | Full client control      | No SEO, no SSR              |

---

## 2. Pre-rendering Routes & Building the App

### Static Site Generation

Pre-render pages at build time for maximum performance.

**Enable pre-rendering:**

```typescript
// src/routes/(marketing)/+layout.ts
// Pre-render all marketing pages
export const prerender = true;
```

**Build and preview:**

```json
// package.json
{
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

```bash
# Build the app
npm run build

# Preview production build
npm run preview
```

**Build output:**

```
.svelte-kit/
└── output/
    ├── client/           # Client-side assets
    │   ├── _app/
    │   │   ├── immutable/
    │   │   └── version.json
    │   └── index.html
    ├── server/           # Server-side code
    │   └── index.js
    └── prerendered/      # Pre-rendered pages
        └── pages/
            ├── index.html
            ├── about.html
            └── blog.html
```

**Conditional pre-rendering:**

```typescript
// src/routes/blog/+page.ts
import { dev } from "$app/environment";

// Only pre-render in production
export const prerender = !dev;
```

**Example marketing site:**

```typescript
// src/routes/(marketing)/+layout.ts
export const prerender = true;
export const ssr = true;
export const csr = true;
```

```svelte
<!-- src/routes/(marketing)/+page.svelte -->
<script lang="ts">
	// This page is pre-rendered at build time
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-5xl font-bold">Welcome to Our App</h1>
			<p class="py-6">This page was pre-rendered at build time for maximum performance.</p>
			<a href="/register" class="btn btn-primary">Get Started</a>
		</div>
	</div>
</div>
```

> ⚠️ **Common Mistake**: Pre-rendering pages that require authentication or user-specific data.

---

## 3. Pre-rendering Endpoints

### Static API Responses

Pre-render API endpoints that return static data.

```typescript
// src/routes/api/config/+server.ts
import { json } from "@sveltejs/kit";
import type { RequestHandler } from "./$types";

export const prerender = true;

export const GET: RequestHandler = async () => {
  // This response is generated at build time
  return json({
    apiVersion: "1.0.0",
    features: ["auth", "workspaces", "pages"],
    maxUploadSize: 5242880, // 5MB
    supportEmail: "support@example.com",
  });
};
```

**Pre-rendered sitemap:**

```typescript
// src/routes/sitemap.xml/+server.ts
import { db } from "$lib/server/db";
import { workspaces, pages } from "$lib/server/db/schema";
import { eq } from "drizzle-orm";
import type { RequestHandler } from "./$types";

export const prerender = true;

export const GET: RequestHandler = async () => {
  // Fetch data at build time
  const allWorkspaces = await db
    .select()
    .from(workspaces)
    .where(eq(workspaces.isPublic, true));

  const allPages = await db
    .select()
    .from(pages)
    .where(eq(pages.isPublished, true));

  const sitemap = `<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
	<url>
		<loc>https://example.com</loc>
		<changefreq>daily</changefreq>
		<priority>1.0</priority>
	</url>
	${allWorkspaces
    .map(
      (workspace) => `
	<url>
		<loc>https://example.com/workspaces/${workspace.slug}</loc>
		<changefreq>weekly</changefreq>
		<priority>0.8</priority>
	</url>
	`
    )
    .join("")}
	${allPages
    .map(
      (page) => `
	<url>
		<loc>https://example.com/pages/${page.id}</loc>
		<lastmod>${page.updatedAt.toISOString()}</lastmod>
		<changefreq>monthly</changefreq>
		<priority>0.6</priority>
	</url>
	`
    )
    .join("")}
</urlset>`;

  return new Response(sitemap, {
    headers: {
      "Content-Type": "application/xml",
      "Cache-Control": "max-age=0, s-maxage=3600",
    },
  });
};
```

**RSS feed:**

```typescript
// src/routes/blog/rss.xml/+server.ts
import { db } from "$lib/server/db";
import { posts } from "$lib/server/db/schema";
import { desc } from "drizzle-orm";
import type { RequestHandler } from "./$types";

export const prerender = true;

export const GET: RequestHandler = async () => {
  const blogPosts = await db
    .select()
    .from(posts)
    .where(eq(posts.published, true))
    .orderBy(desc(posts.publishedAt))
    .limit(20);

  const rss = `<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0">
	<channel>
		<title>My Blog</title>
		<link>https://example.com/blog</link>
		<description>Latest blog posts</description>
		${blogPosts
      .map(
        (post) => `
		<item>
			<title>${escapeXml(post.title)}</title>
			<link>https://example.com/blog/${post.slug}</link>
			<description>${escapeXml(post.excerpt)}</description>
			<pubDate>${new Date(post.publishedAt).toUTCString()}</pubDate>
		</item>
		`
      )
      .join("")}
	</channel>
</rss>`;

  return new Response(rss, {
    headers: { "Content-Type": "application/xml" },
  });
};

function escapeXml(str: string): string {
  return str
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;")
    .replace(/'/g, "&apos;");
}
```

---

## 4. Pre-rendering with Dynamic Data

### Database Data at Build Time

Fetch database data during build to pre-render pages.

```typescript
// src/routes/blog/+page.server.ts
import { db } from "$lib/server/db";
import { posts } from "$lib/server/db/schema";
import { desc, eq } from "drizzle-orm";
import type { PageServerLoad } from "./$types";

export const prerender = true;

export const load: PageServerLoad = async () => {
  // This runs at BUILD TIME, not request time
  const blogPosts = await db
    .select({
      id: posts.id,
      title: posts.title,
      slug: posts.slug,
      excerpt: posts.excerpt,
      publishedAt: posts.publishedAt,
      author: posts.authorName,
    })
    .from(posts)
    .where(eq(posts.published, true))
    .orderBy(desc(posts.publishedAt));

  return {
    posts: blogPosts,
  };
};
```

```svelte
<!-- src/routes/blog/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<div class="container mx-auto px-4 py-12">
	<h1 class="text-4xl font-bold mb-8">Blog</h1>

	<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
		{#each data.posts as post}
			<article class="card bg-base-100 shadow-xl">
				<div class="card-body">
					<h2 class="card-title">{post.title}</h2>
					<p class="text-base-content/60">{post.excerpt}</p>
					<div class="text-sm text-base-content/40">
						{new Date(post.publishedAt).toLocaleDateString()}
					</div>
					<div class="card-actions">
						<a href="/blog/{post.slug}" class="btn btn-primary btn-sm"> Read More </a>
					</div>
				</div>
			</article>
		{/each}
	</div>
</div>
```

> 💡 **Best Practice**: Pre-render content that changes infrequently (blog posts, docs, marketing pages).

---

## 5. Pre-rendering Dynamic Parameters

### Generating Multiple Static Pages

Use `entries` to specify which dynamic routes to pre-render.

```typescript
// src/routes/blog/[slug]/+page.server.ts
import { error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { posts } from "$lib/server/db/schema";
import { eq } from "drizzle-orm";
import type { PageServerLoad, EntryGenerator } from "./$types";

export const prerender = true;

// Tell SvelteKit which [slug] values to pre-render
// WITHOUT this, SvelteKit doesn't know which dynamic routes to build
export const entries: EntryGenerator = async () => {
  // Query database at BUILD time to get all post slugs
  // This runs during 'npm run build', not at request time
  const allPosts = await db
    .select({ slug: posts.slug })
    .from(posts)
    .where(eq(posts.published, true));

  // Return array of parameter objects
  // SvelteKit will pre-render /blog/post-1, /blog/post-2, etc.
  // Each object must have keys matching route parameters
  return allPosts.map((post) => ({ slug: post.slug }));
};

export const load: PageServerLoad = async ({ params }) => {
  const [post] = await db
    .select()
    .from(posts)
    .where(eq(posts.slug, params.slug))
    .limit(1);

  if (!post) {
    throw error(404, "Post not found");
  }

  return { post };
};
```

**Multiple parameters:**

```typescript
// src/routes/docs/[category]/[page]/+page.server.ts
import type { EntryGenerator } from "./$types";

export const prerender = true;

export const entries: EntryGenerator = async () => {
  return [
    { category: "getting-started", page: "installation" },
    { category: "getting-started", page: "configuration" },
    { category: "api", page: "authentication" },
    { category: "api", page: "workspaces" },
    { category: "guides", page: "deployment" },
  ];
};
```

**Wildcard entries:**

```typescript
// src/routes/[...catchall]/+page.server.ts
export const prerender = true;

export const entries: EntryGenerator = () => {
  return [
    { catchall: "about" },
    { catchall: "contact" },
    { catchall: "privacy" },
    { catchall: "terms" },
  ];
};
```

> ⚠️ **Common Mistake**: Forgetting to export `entries` for dynamic routes.

---

## 6. Adding Dynamic Content to Pre-rendered Pages

### Hybrid Rendering Approach

Combine pre-rendered content with client-side dynamic data.

**Pre-rendered shell + dynamic content:**

```typescript
// src/routes/products/+page.server.ts
import { db } from "$lib/server/db";
import { products } from "$lib/server/db/schema";
import type { PageServerLoad } from "./$types";

export const prerender = true;

export const load: PageServerLoad = async () => {
  // Pre-render initial product data
  const featuredProducts = await db
    .select()
    .from(products)
    .where(eq(products.featured, true))
    .limit(6);

  return {
    featuredProducts,
  };
};
```

```svelte
<!-- src/routes/products/+page.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	// Static data (pre-rendered)
	let featuredProducts = $derived(data.featuredProducts);

	// Dynamic data (loaded client-side)
	let liveInventory = $state<Record<number, number>>({});
	let loading = $state(false);

	onMount(async () => {
		// Fetch real-time inventory on the client
		loading = true;
		const res = await fetch('/api/inventory');
		liveInventory = await res.json();
		loading = false;
	});
</script>

<div class="container mx-auto px-4 py-12">
	<h1 class="text-4xl font-bold mb-8">Featured Products</h1>

	<div class="grid md:grid-cols-3 gap-6">
		{#each featuredProducts as product}
			<div class="card bg-base-100 shadow-xl">
				<figure>
					<img src={product.imageUrl} alt={product.name} />
				</figure>
				<div class="card-body">
					<h2 class="card-title">{product.name}</h2>
					<p>${product.price}</p>

					<!-- Dynamic inventory -->
					{#if loading}
						<div class="badge badge-ghost">Loading stock...</div>
					{:else if liveInventory[product.id]}
						{@const stock = liveInventory[product.id]}
						<div class="badge" class:badge-success={stock > 10} class:badge-warning={stock <= 10}>
							{stock} in stock
						</div>
					{/if}

					<div class="card-actions">
						<button class="btn btn-primary btn-block">Add to Cart</button>
					</div>
				</div>
			</div>
		{/each}
	</div>
</div>
```

**Pre-rendered blog + dynamic comments:**

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	// Pre-rendered content
	let post = $derived(data.post);

	// Dynamic comments (client-side)
	let comments = $state([]);
	let loadingComments = $state(true);

	onMount(async () => {
		const res = await fetch(`/api/posts/${post.id}/comments`);
		comments = await res.json();
		loadingComments = false;
	});
</script>

<!-- Pre-rendered blog post -->
<article class="prose lg:prose-xl mx-auto">
	<h1>{post.title}</h1>
	<div class="text-base-content/60">
		By {post.author} • {new Date(post.publishedAt).toLocaleDateString()}
	</div>
	<div>{@html post.content}</div>
</article>

<!-- Dynamic comments section -->
<section class="mt-12 max-w-4xl mx-auto">
	<h2 class="text-2xl font-bold mb-4">Comments</h2>

	{#if loadingComments}
		<div class="loading loading-spinner"></div>
	{:else if comments.length === 0}
		<p class="text-base-content/60">No comments yet. Be the first!</p>
	{:else}
		{#each comments as comment}
			<div class="card bg-base-100 shadow mb-4">
				<div class="card-body">
					<div class="font-bold">{comment.authorName}</div>
					<p>{comment.content}</p>
					<div class="text-sm text-base-content/40">
						{new Date(comment.createdAt).toLocaleString()}
					</div>
				</div>
			</div>
		{/each}
	{/if}

	<!-- Comment form -->
	<form method="POST" action="?/addComment" class="mt-6">
		<textarea
			name="content"
			class="textarea textarea-bordered w-full"
			placeholder="Add a comment..."
		></textarea>
		<button type="submit" class="btn btn-primary mt-2">Post Comment</button>
	</form>
</section>
```

**Benefits:**

- ✅ Fast initial page load (pre-rendered)
- ✅ Fresh dynamic data (client-side)
- ✅ Good for SEO (static content)
- ✅ Real-time features (dynamic parts)

---

## 7. Complete Example: Documentation Site with Blog

### Full Pre-rendering Strategy

Let's build a complete documentation and blog site with optimal pre-rendering!

**Features:**

- ✅ Pre-rendered documentation pages
- ✅ Pre-rendered blog posts
- ✅ Pre-rendered sitemap and RSS
- ✅ Dynamic search
- ✅ Client-side navigation
- ✅ Build-time data fetching

### 📁 Project Structure

```
src/
├── lib/
│   ├── server/
│   │   └── db/
│   │       ├── index.ts
│   │       └── schema.ts
│   └── components/
│       ├── DocsSidebar.svelte
│       ├── BlogCard.svelte
│       └── SearchBar.svelte
├── routes/
│   ├── (marketing)/
│   │   ├── +layout.ts              # Pre-render: true
│   │   ├── +layout.svelte
│   │   ├── +page.svelte
│   │   ├── about/+page.svelte
│   │   └── contact/+page.svelte
│   ├── docs/
│   │   ├── +layout.ts              # Pre-render: true
│   │   ├── +layout.svelte
│   │   ├── +page.svelte
│   │   └── [category]/
│   │       └── [slug]/
│   │           └── +page.server.ts  # entries function
│   ├── blog/
│   │   ├── +page.server.ts         # Pre-render: true
│   │   ├── +page.svelte
│   │   ├── [slug]/
│   │   │   └── +page.server.ts     # entries function
│   │   └── rss.xml/+server.ts      # Pre-render: true
│   ├── api/
│   │   ├── search/+server.ts       # Pre-render: false
│   │   └── config/+server.ts       # Pre-render: true
│   ├── sitemap.xml/+server.ts      # Pre-render: true
│   └── robots.txt/+server.ts       # Pre-render: true
└── svelte.config.js
```

### Configuration

```typescript
// svelte.config.js
import adapter from "@sveltejs/adapter-static";

const config = {
  kit: {
    adapter: adapter({
      pages: "build",
      assets: "build",
      fallback: null,
      precompress: true,
      strict: true,
    }),
    prerender: {
      entries: ["*"],
      crawl: true,
      handleHttpError: ({ path, referrer, message }) => {
        if (path.startsWith("/api/")) return;
        throw new Error(message);
      },
    },
    trailingSlash: "never",
  },
};

export default config;
```

### Database Schema

```typescript
// src/lib/server/db/schema.ts
export const docPages = pgTable("doc_pages", {
  id: serial("id").primaryKey(),
  category: text("category").notNull(),
  slug: text("slug").notNull(),
  title: text("title").notNull(),
  content: text("content").notNull(),
  order: integer("order").default(0),
  updatedAt: timestamp("updated_at").defaultNow(),
});

export const blogPosts = pgTable("blog_posts", {
  id: serial("id").primaryKey(),
  slug: text("slug").notNull().unique(),
  title: text("title").notNull(),
  excerpt: text("excerpt").notNull(),
  content: text("content").notNull(),
  authorName: text("author_name").notNull(),
  authorAvatar: text("author_avatar"),
  published: boolean("published").default(false),
  publishedAt: timestamp("published_at"),
  updatedAt: timestamp("updated_at").defaultNow(),
});
```

### Documentation Pages

```typescript
// src/routes/docs/[category]/[slug]/+page.server.ts
import { error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { docPages } from "$lib/server/db/schema";
import { eq, and } from "drizzle-orm";
import type { PageServerLoad, EntryGenerator } from "./$types";

export const prerender = true;

export const entries: EntryGenerator = async () => {
  const allDocs = await db
    .select({
      category: docPages.category,
      slug: docPages.slug,
    })
    .from(docPages);

  return allDocs.map((doc) => ({
    category: doc.category,
    slug: doc.slug,
  }));
};

export const load: PageServerLoad = async ({ params }) => {
  const [doc] = await db
    .select()
    .from(docPages)
    .where(
      and(
        eq(docPages.category, params.category),
        eq(docPages.slug, params.slug)
      )
    )
    .limit(1);

  if (!doc) {
    throw error(404, "Documentation page not found");
  }

  // Get all docs for sidebar
  const allDocs = await db
    .select({
      category: docPages.category,
      slug: docPages.slug,
      title: docPages.title,
      order: docPages.order,
    })
    .from(docPages)
    .orderBy(docPages.category, docPages.order);

  // Group by category
  const docsByCategory = allDocs.reduce(
    (acc, doc) => {
      if (!acc[doc.category]) {
        acc[doc.category] = [];
      }
      acc[doc.category].push(doc);
      return acc;
    },
    {} as Record<string, typeof allDocs>
  );

  return {
    doc,
    docsByCategory,
  };
};
```

```svelte
<!-- src/routes/docs/[category]/[slug]/+page.svelte -->
<script lang="ts">
	import { marked } from 'marked';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	let renderedContent = $derived(marked(data.doc.content));
</script>

<svelte:head>
	<title>{data.doc.title} - Documentation</title>
	<meta name="description" content={data.doc.excerpt} />
</svelte:head>

<div class="flex">
	<!-- Sidebar -->
	<aside class="w-64 border-r bg-base-200 min-h-screen p-4">
		<div class="sticky top-4">
			<h2 class="font-bold text-lg mb-4">Documentation</h2>
			{#each Object.entries(data.docsByCategory) as [category, docs]}
				<div class="mb-6">
					<h3 class="font-semibold text-sm uppercase text-base-content/60 mb-2">
						{category}
					</h3>
					<ul class="menu menu-compact">
						{#each docs as doc}
							<li>
								<a href="/docs/{doc.category}/{doc.slug}" class:active={doc.slug === data.doc.slug}>
									{doc.title}
								</a>
							</li>
						{/each}
					</ul>
				</div>
			{/each}
		</div>
	</aside>

	<!-- Content -->
	<main class="flex-1 p-8 max-w-4xl">
		<article class="prose lg:prose-xl">
			<h1>{data.doc.title}</h1>
			{@html renderedContent}
		</article>

		<div class="mt-12 pt-8 border-t">
			<p class="text-sm text-base-content/60">
				Last updated: {new Date(data.doc.updatedAt).toLocaleDateString()}
			</p>
		</div>
	</main>
</div>
```

### Blog List

```typescript
// src/routes/blog/+page.server.ts
import { db } from "$lib/server/db";
import { blogPosts } from "$lib/server/db/schema";
import { desc, eq } from "drizzle-orm";
import type { PageServerLoad } from "./$types";

export const prerender = true;

export const load: PageServerLoad = async () => {
  const posts = await db
    .select({
      slug: blogPosts.slug,
      title: blogPosts.title,
      excerpt: blogPosts.excerpt,
      authorName: blogPosts.authorName,
      authorAvatar: blogPosts.authorAvatar,
      publishedAt: blogPosts.publishedAt,
    })
    .from(blogPosts)
    .where(eq(blogPosts.published, true))
    .orderBy(desc(blogPosts.publishedAt));

  return { posts };
};
```

```svelte
<!-- src/routes/blog/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<svelte:head>
	<title>Blog - Latest Articles</title>
	<link rel="alternate" type="application/rss+xml" href="/blog/rss.xml" />
</svelte:head>

<div class="container mx-auto px-4 py-12">
	<div class="flex justify-between items-center mb-8">
		<h1 class="text-4xl font-bold">Blog</h1>
		<a href="/blog/rss.xml" class="btn btn-ghost btn-sm gap-2">
			<svg class="w-5 h-5" fill="currentColor" viewBox="0 0 20 20">
				<path d="M5 3a1 1 0 000 2c5.523 0 10 4.477 10 10a1 1 0 102 0C17 8.373 11.627 3 5 3z" />
				<path
					d="M4 9a1 1 0 011-1 7 7 0 017 7 1 1 0 11-2 0 5 5 0 00-5-5 1 1 0 01-1-1zM3 15a2 2 0 114 0 2 2 0 01-4 0z"
				/>
			</svg>
			RSS Feed
		</a>
	</div>

	<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
		{#each data.posts as post}
			<article class="card bg-base-100 shadow-xl hover:shadow-2xl transition">
				<div class="card-body">
					<h2 class="card-title">{post.title}</h2>
					<p class="text-base-content/60">{post.excerpt}</p>

					<div class="flex items-center gap-3 mt-4">
						<img
							src={post.authorAvatar || 'https://i.pravatar.cc/150'}
							alt={post.authorName}
							class="w-10 h-10 rounded-full"
						/>
						<div>
							<p class="font-medium text-sm">{post.authorName}</p>
							<p class="text-xs text-base-content/60">
								{new Date(post.publishedAt).toLocaleDateString()}
							</p>
						</div>
					</div>

					<div class="card-actions justify-end mt-4">
						<a href="/blog/{post.slug}" class="btn btn-primary btn-sm"> Read More </a>
					</div>
				</div>
			</article>
		{/each}
	</div>
</div>
```

### Blog Post

```typescript
// src/routes/blog/[slug]/+page.server.ts
import { error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { blogPosts } from "$lib/server/db/schema";
import { eq, and } from "drizzle-orm";
import type { PageServerLoad, EntryGenerator } from "./$types";

export const prerender = true;

export const entries: EntryGenerator = async () => {
  const posts = await db
    .select({ slug: blogPosts.slug })
    .from(blogPosts)
    .where(eq(blogPosts.published, true));

  return posts.map((post) => ({ slug: post.slug }));
};

export const load: PageServerLoad = async ({ params }) => {
  const [post] = await db
    .select()
    .from(blogPosts)
    .where(and(eq(blogPosts.slug, params.slug), eq(blogPosts.published, true)))
    .limit(1);

  if (!post) {
    throw error(404, "Post not found");
  }

  return { post };
};
```

### Robots.txt

```typescript
// src/routes/robots.txt/+server.ts
import type { RequestHandler } from "./$types";

export const prerender = true;

export const GET: RequestHandler = () => {
  return new Response(
    `User-agent: *
Allow: /
Sitemap: https://example.com/sitemap.xml`,
    {
      headers: { "Content-Type": "text/plain" },
    }
  );
};
```

**Key Features:**

- ✅ **Full Pre-rendering**: All docs and blog posts generated at build time
- ✅ **Dynamic Routes**: Multiple pages from database
- ✅ **SEO Optimized**: Sitemap, RSS, robots.txt
- ✅ **Fast Performance**: Static HTML served instantly
- ✅ **Client Hydration**: Interactive after load
- ✅ **Build-time Data**: Fresh database queries during build

---

## 📝 Key Takeaways

✅ SSR is default, great for dynamic content  
✅ CSR for rich client-side apps  
✅ Pre-rendering for static content (fastest)  
✅ Use `entries` to specify dynamic routes  
✅ Pre-render endpoints for static APIs  
✅ Combine pre-rendered shell with dynamic content  
✅ Pre-render marketing, docs, and blog pages  
✅ Build time queries with database connection  
✅ `npm run build` generates static files  
✅ Configure trailing slashes consistently  
✅ Pre-render sitemap and RSS feeds  
✅ Use static adapter for fully pre-rendered sites

---

## 🚀 Next Steps

You've mastered rendering strategies! Next up:

- **Section 19**: Deployment & Adapters
- Advanced caching strategies
- Incremental static regeneration
- Edge rendering patterns
