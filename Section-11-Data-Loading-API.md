# Section 11: Data Loading & API Routes

## 📚 Learning Objectives

By the end of this section, you will:

- Master load functions for server and client-side data fetching
- Implement type-safe data loading with generated types
- Create API endpoints with POST/GET handlers
- Build pagination and infinite scroll patterns
- Handle authentication with hooks and locals
- Implement error handling strategies
- Add loading progress indicators
- Stream data from server to client
- Build a complete blog platform with authentication

---

## Table of Contents

- [Section 11: Data Loading \& API Routes](#section-11-data-loading--api-routes)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Load Functions Overview](#1-load-functions-overview)
    - [What are Load Functions?](#what-are-load-functions)
  - [2. Universal Load Functions](#2-universal-load-functions)
    - [Running on Server and Client](#running-on-server-and-client)
  - [3. Zero-Effort Type Safety with Generated Types](#3-zero-effort-type-safety-with-generated-types)
    - [Automatic TypeScript Types](#automatic-typescript-types)
  - [4. Server-Only Load Functions](#4-server-only-load-functions)
    - [Secure Server-Side Data Loading](#secure-server-side-data-loading)
  - [5. Layout Load Functions](#5-layout-load-functions)
    - [Loading Data for Multiple Pages](#loading-data-for-multiple-pages)
  - [6. Accessing Parent Data in Child Load Functions](#6-accessing-parent-data-in-child-load-functions)
    - [Inheriting Parent Data](#inheriting-parent-data)
  - [7. Passing Data From Pages to Layouts](#7-passing-data-from-pages-to-layouts)
    - [Child-to-Parent Communication](#child-to-parent-communication)
  - [8. Creating Endpoints](#8-creating-endpoints)
    - [Building API Routes](#building-api-routes)
  - [9. Handling POST Requests in Endpoints](#9-handling-post-requests-in-endpoints)
    - [Processing Form Data](#processing-form-data)
  - [10. Handling Pages \& Endpoints in the Same Route](#10-handling-pages--endpoints-in-the-same-route)
    - [Co-locating UI and API](#co-locating-ui-and-api)
  - [11. Sending Fetch Requests in Load Functions](#11-sending-fetch-requests-in-load-functions)
    - [Fetching External APIs](#fetching-external-apis)
  - [12. Implementing Pagination Using URL Search Params](#12-implementing-pagination-using-url-search-params)
    - [URL-Based Pagination](#url-based-pagination)
  - [13. Load More Pagination Pattern](#13-load-more-pagination-pattern)
    - [Infinite Scroll Implementation](#infinite-scroll-implementation)
  - [14. Running Requests in Parallel](#14-running-requests-in-parallel)
    - [Promise.all for Performance](#promiseall-for-performance)
  - [15. Streaming Data](#15-streaming-data)
    - [Progressive Data Loading](#progressive-data-loading)
  - [16. Navigation Loading Indicators](#16-navigation-loading-indicators)
    - [Progress Bars with beforeNavigate](#progress-bars-with-beforenavigate)
  - [17. Hooks Overview](#17-hooks-overview)
    - [Server-Side Interceptors](#server-side-interceptors)
  - [18. Handle Hook \& Authentication](#18-handle-hook--authentication)
    - [Protecting Routes with Hooks](#protecting-routes-with-hooks)
  - [19. Error Handling](#19-error-handling)
    - [Expected and Unexpected Errors](#expected-and-unexpected-errors)
  - [20. Complete Example: Blog Platform with Auth](#20-complete-example-blog-platform-with-auth)
    - [Full-Stack Blog Application](#full-stack-blog-application)
    - [📁 Project Structure](#-project-structure)
    - [Mock Database](#mock-database)
    - [Auth Helpers](#auth-helpers)
    - [Hooks](#hooks)
    - [Type Definitions](#type-definitions)
    - [Public Blog List](#public-blog-list)
    - [Single Post Page](#single-post-page)
    - [Login Page](#login-page)
    - [Admin Dashboard](#admin-dashboard)
    - [API Endpoints](#api-endpoints)
    - [Components](#components)
    - [Error Page](#error-page)
    - [Navigation with Loading Bar](#navigation-with-loading-bar)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Load Functions Overview

### What are Load Functions?

Load functions fetch data **before** a page renders. They run on the server during SSR and on the client during navigation.

**Real-World Scenario:** You need to fetch blog posts before showing the blog page.

**What it does:** Loads data and makes it available to your page component.

**Key Concepts:**

- `+page.ts` - Universal load (runs on server + client)
- `+page.server.ts` - Server-only load (database, secrets)
- `+layout.ts` / `+layout.server.ts` - Layout load functions
- Return data as an object - it becomes `data` prop

```typescript
// src/routes/blog/+page.ts
export const load = async () => {
  return {
    posts: [
      { id: 1, title: "First Post", slug: "first-post" },
      { id: 2, title: "Second Post", slug: "second-post" },
    ],
  };
};
```

```svelte
<!-- src/routes/blog/+page.svelte -->
<script lang="ts">
	let { data } = $props();
</script>

<div class="grid gap-4">
	{#each data.posts as post}
		<div class="card bg-base-100 shadow">
			<div class="card-body">
				<h2 class="card-title">{post.title}</h2>
				<a href="/blog/{post.slug}" class="btn btn-primary btn-sm">Read More</a>
			</div>
		</div>
	{/each}
</div>
```

> 💡 **Best Practice**: Use load functions instead of `onMount()` for better SEO and faster initial renders.

---

## 2. Universal Load Functions

### Running on Server and Client

Universal loads (`+page.ts`) run on both server and client.

**Real-World Scenario:** Fetching data from a public API that works in both environments.

```typescript
// src/routes/products/+page.ts
export const load = async ({ fetch }) => {
  const response = await fetch("https://api.example.com/products");
  const products = await response.json();

  return { products };
};
```

**When to use:**

- ✅ Public API calls
- ✅ Client-side navigation needs data
- ✅ No secrets involved

---

## 3. Zero-Effort Type Safety with Generated Types

### Automatic TypeScript Types

SvelteKit generates types automatically from your load functions!

```typescript
// src/routes/blog/+page.ts
export const load = async () => {
  return {
    title: "Blog",
    posts: [{ id: 1, title: "Post 1" }],
  };
};
```

```svelte
<!-- src/routes/blog/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types'; // Auto-generated!

	let { data }: { data: PageData } = $props();

	// Full autocomplete for data.title and data.posts!
</script>
```

> 💡 **Best Practice**: Always import types from `./$types` for autocomplete and type safety.

---

## 4. Server-Only Load Functions

### Secure Server-Side Data Loading

Use `+page.server.ts` for database queries and secrets.

**Real-World Scenario:** Fetching user data from a database with API keys.

```typescript
// src/routes/dashboard/+page.server.ts
import { db } from "$lib/server/database";

export const load = async ({ locals }) => {
  // Only runs on server - safe for secrets!
  const user = await db.users.findUnique({
    where: { id: locals.userId },
  });

  return { user };
};
```

**When to use:**

- ✅ Database queries
- ✅ API calls with secrets
- ✅ Server-side only operations
- ✅ Accessing cookies securely

---

## 5. Layout Load Functions

### Loading Data for Multiple Pages

Layout load functions provide data to all child pages.

```typescript
// src/routes/dashboard/+layout.server.ts
export const load = async ({ locals }) => {
  return {
    user: locals.user,
    navigation: [
      { label: "Overview", href: "/dashboard" },
      { label: "Settings", href: "/dashboard/settings" },
    ],
  };
};
```

```svelte
<!-- src/routes/dashboard/+page.svelte -->
<script lang="ts">
	let { data } = $props();
	// Has access to layout data automatically!
</script>

<h1>Welcome, {data.user.name}!</h1>
```

---

## 6. Accessing Parent Data in Child Load Functions

### Inheriting Parent Data

Access parent load data using `await parent()`.

```typescript
// src/routes/dashboard/settings/+page.ts
export const load = async ({ parent }) => {
  const { user } = await parent(); // Get layout data

  return {
    user, // Pass it down
    settings: { theme: "dark", notifications: true },
  };
};
```

---

## 7. Passing Data From Pages to Layouts

### Child-to-Parent Communication

Use the `page` store to pass data up to layouts.

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	let { data } = $props();

	// Set page metadata
	$effect(() => {
		$page.data.meta = {
			title: data.post.title,
			description: data.post.excerpt
		};
	});
</script>
```

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
	import type { Snippet } from 'svelte';

	let { children }: { children: Snippet } = $props();
</script>

<svelte:head>
	<title>{$page.data.meta?.title || 'Default Title'}</title>
</svelte:head>

{@render children()}
```

---

## 8. Creating Endpoints

### Building API Routes

Create API endpoints with `+server.ts` files.

```typescript
// src/routes/api/posts/+server.ts
import { json } from "@sveltejs/kit";

export const GET = async () => {
  const posts = [
    { id: 1, title: "First Post" },
    { id: 2, title: "Second Post" },
  ];

  return json(posts);
};
```

**Access at:** `GET /api/posts`

---

## 9. Handling POST Requests in Endpoints

### Processing Form Data

```typescript
// src/routes/api/posts/+server.ts
import { json } from "@sveltejs/kit";

export const POST = async ({ request }) => {
  const body = await request.json();

  // Save to database
  const newPost = {
    id: Date.now(),
    title: body.title,
    content: body.content,
  };

  return json(newPost, { status: 201 });
};
```

---

## 10. Handling Pages & Endpoints in the Same Route

### Co-locating UI and API

You can have both `+page.svelte` and `+server.ts` in the same route!

```
src/routes/contact/
├── +page.svelte       → GET /contact (renders form)
└── +server.ts         → POST /contact (handles submission)
```

```svelte
<!-- src/routes/contact/+page.svelte -->
<script lang="ts">
	let message = $state('');
	let status = $state('');

	async function handleSubmit() {
		const response = await fetch('/contact', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({ message })
		});

		if (response.ok) {
			status = 'Message sent!';
			message = '';
		}
	}
</script>

<form
	onsubmit={(e) => {
		e.preventDefault();
		handleSubmit();
	}}
>
	<textarea bind:value={message} class="textarea textarea-bordered w-full"></textarea>
	<button type="submit" class="btn btn-primary">Send</button>
	{#if status}<p class="text-success">{status}</p>{/if}
</form>
```

```typescript
// src/routes/contact/+server.ts
import { json } from "@sveltejs/kit";

export const POST = async ({ request }) => {
  const { message } = await request.json();

  // Send email, save to DB, etc.
  console.log("Received message:", message);

  return json({ success: true });
};
```

---

## 11. Sending Fetch Requests in Load Functions

### Fetching External APIs

Use the enhanced `fetch` in load functions for automatic SSR support.

```typescript
// src/routes/weather/+page.ts
export const load = async ({ fetch }) => {
  // This fetch works on both server and client!
  const response = await fetch("https://api.weather.com/forecast");
  const data = await response.json();

  return { weather: data };
};
```

> 💡 **Best Practice**: Always use the `fetch` provided by the load function, not the global `fetch`.

---

## 12. Implementing Pagination Using URL Search Params

### URL-Based Pagination

```typescript
// src/routes/blog/+page.ts
export const load = async ({ url, fetch }) => {
  const page = Number(url.searchParams.get("page") || "1");
  const limit = 10;

  const response = await fetch(`/api/posts?page=${page}&limit=${limit}`);
  const { posts, total } = await response.json();

  return {
    posts,
    page,
    totalPages: Math.ceil(total / limit),
  };
};
```

```svelte
<!-- src/routes/blog/+page.svelte -->
<script lang="ts">
	let { data } = $props();
</script>

<div class="grid gap-4">
	{#each data.posts as post}
		<div class="card bg-base-100 shadow">
			<div class="card-body">
				<h2 class="card-title">{post.title}</h2>
			</div>
		</div>
	{/each}
</div>

<div class="join mt-6">
	{#each Array(data.totalPages) as _, i}
		<a href="?page={i + 1}" class="join-item btn" class:btn-active={data.page === i + 1}>
			{i + 1}
		</a>
	{/each}
</div>
```

---

## 13. Load More Pagination Pattern

### Infinite Scroll Implementation

```svelte
<script lang="ts">
	let posts = $state([]);
	let page = $state(1);
	let loading = $state(false);

	async function loadMore() {
		loading = true;
		const response = await fetch(`/api/posts?page=${page}`);
		const newPosts = await response.json();
		posts = [...posts, ...newPosts];
		page++;
		loading = false;
	}
</script>

<div class="grid gap-4">
	{#each posts as post}
		<div class="card bg-base-100 shadow">
			<div class="card-body">
				<h2 class="card-title">{post.title}</h2>
			</div>
		</div>
	{/each}
</div>

<button onclick={loadMore} class="btn btn-primary" disabled={loading}>
	{loading ? 'Loading...' : 'Load More'}
</button>
```

---

## 14. Running Requests in Parallel

### Promise.all for Performance

```typescript
// src/routes/dashboard/+page.server.ts
export const load = async () => {
  // ❌ Slow: Sequential
  // const user = await fetchUser();
  // const stats = await fetchStats();
  // const notifications = await fetchNotifications();

  // ✅ Fast: Parallel
  const [user, stats, notifications] = await Promise.all([
    fetchUser(),
    fetchStats(),
    fetchNotifications(),
  ]);

  return { user, stats, notifications };
};
```

> 💡 **Performance Tip**: Use `Promise.all()` when requests don't depend on each other.

---

## 15. Streaming Data

### Progressive Data Loading

```typescript
// src/routes/dashboard/+page.server.ts
export const load = async () => {
  return {
    // Fast data - loads immediately
    user: await fetchUser(),

    // Slow data - streams in later
    analytics: fetchAnalytics(), // No await!
  };
};
```

```svelte
<script lang="ts">
	let { data } = $props();
</script>

<h1>Welcome, {data.user.name}</h1>

{#await data.analytics}
	<div class="loading loading-spinner"></div>
{:then analytics}
	<div>Revenue: ${analytics.revenue}</div>
{/await}
```

---

## 16. Navigation Loading Indicators

### Progress Bars with beforeNavigate

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
	import { beforeNavigate, afterNavigate } from '$app/navigation';
	import { navigating } from '$app/stores';

	let progress = $state(0);
	let interval: number;

	beforeNavigate(() => {
		progress = 0;
		interval = setInterval(() => {
			progress = Math.min(progress + 10, 90);
		}, 100);
	});

	afterNavigate(() => {
		clearInterval(interval);
		progress = 100;
		setTimeout(() => (progress = 0), 500);
	});
</script>

{#if progress > 0}
	<div class="fixed top-0 left-0 w-full h-1 bg-primary z-50" style="width: {progress}%"></div>
{/if}

{@render children()}
```

> **Note:** Add `import type { Snippet } from 'svelte';` and `let { children }: { children: Snippet } = $props();` in the script section.

---

## 17. Hooks Overview

### Server-Side Interceptors

Hooks run on every request before route handlers.

**Files:**

- `src/hooks.server.ts` - Server hooks
- `src/hooks.client.ts` - Client hooks (rare)

**Use cases:**

- Authentication checks
- Logging
- Adding headers
- Redirects

---

## 18. Handle Hook & Authentication

### Protecting Routes with Hooks

```typescript
// src/hooks.server.ts
import { redirect } from "@sveltejs/kit";
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  // Get session from cookie
  const sessionId = event.cookies.get("session");

  if (sessionId) {
    // Fetch user from database
    event.locals.user = await getUserBySession(sessionId);
  }

  // Protect dashboard routes
  if (event.url.pathname.startsWith("/dashboard") && !event.locals.user) {
    throw redirect(303, "/login");
  }

  return resolve(event);
};
```

**Access user anywhere:**

```typescript
// Any +page.server.ts or +layout.server.ts
export const load = async ({ locals }) => {
  return {
    user: locals.user, // Set in hooks!
  };
};
```

---

## 19. Error Handling

### Expected and Unexpected Errors

```typescript
// src/routes/blog/[slug]/+page.server.ts
import { error } from "@sveltejs/kit";

export const load = async ({ params }) => {
  const post = await db.posts.findUnique({
    where: { slug: params.slug },
  });

  if (!post) {
    // Expected error - shows custom error page
    throw error(404, "Post not found");
  }

  return { post };
};
```

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-5xl font-bold">{$page.status}</h1>
			<p class="py-6">{$page.error?.message}</p>
			<a href="/" class="btn btn-primary">Go Home</a>
		</div>
	</div>
</div>
```

---

## 20. Complete Example: Blog Platform with Auth

### Full-Stack Blog Application

Let's build a complete blog platform that demonstrates everything!

**Features:**

- ✅ Authentication with hooks
- ✅ Protected admin routes
- ✅ Public blog with pagination
- ✅ API endpoints for CRUD
- ✅ Type-safe data loading
- ✅ Error handling
- ✅ Loading indicators
- ✅ Streaming data

### 📁 Project Structure

```
src/
├── lib/
│   ├── server/
│   │   ├── db.ts                    # Mock database
│   │   └── auth.ts                  # Auth helpers
│   └── components/
│       ├── PostCard.svelte
│       └── Pagination.svelte
├── routes/
│   ├── +layout.svelte               # Root layout
│   ├── +error.svelte                # Error page
│   ├── (public)/                    # Public routes
│   │   ├── +layout.svelte
│   │   ├── +page.svelte             # Home
│   │   ├── blog/
│   │   │   ├── +page.ts             # Blog list with pagination
│   │   │   ├── +page.svelte
│   │   │   └── [slug]/
│   │   │       ├── +page.ts         # Single post
│   │   │       └── +page.svelte
│   │   └── login/
│   │       ├── +page.svelte         # Login form
│   │       └── +server.ts           # Login handler
│   ├── (admin)/                     # Protected admin
│   │   ├── +layout.server.ts        # Auth check
│   │   ├── +layout.svelte           # Admin layout
│   │   └── admin/
│   │       ├── +page.server.ts      # Admin dashboard
│   │       ├── +page.svelte
│   │       └── posts/
│   │           ├── +page.server.ts  # Post management
│   │           ├── +page.svelte
│   │           └── new/
│   │               ├── +page.svelte
│   │               └── +server.ts   # Create post
│   └── api/
│       └── posts/
│           ├── +server.ts           # GET all posts
│           └── [id]/
│               └── +server.ts       # GET/PUT/DELETE post
└── hooks.server.ts                  # Auth middleware
```

### Mock Database

```typescript
// src/lib/server/db.ts
export interface Post {
  id: number;
  title: string;
  slug: string;
  content: string;
  excerpt: string;
  published: boolean;
  createdAt: string;
}

export interface User {
  id: number;
  email: string;
  name: string;
  role: "admin" | "user";
}

// Mock data
let posts: Post[] = [
  {
    id: 1,
    title: "Getting Started with SvelteKit",
    slug: "getting-started-sveltekit",
    content: "Full content here...",
    excerpt: "Learn the basics of SvelteKit",
    published: true,
    createdAt: "2024-01-01",
  },
  {
    id: 2,
    title: "Advanced Routing Patterns",
    slug: "advanced-routing",
    content: "Full content here...",
    excerpt: "Master SvelteKit routing",
    published: true,
    createdAt: "2024-01-02",
  },
];

let users: User[] = [
  { id: 1, email: "admin@example.com", name: "Admin User", role: "admin" },
];

const sessions = new Map<string, number>(); // sessionId -> userId

export const db = {
  posts: {
    findMany: async ({ skip = 0, take = 10, where = {} }) => {
      let filtered = posts.filter((p) => {
        if (where.published !== undefined && p.published !== where.published)
          return false;
        return true;
      });
      return filtered.slice(skip, skip + take);
    },
    count: async ({ where = {} }) => {
      return posts.filter((p) => {
        if (where.published !== undefined && p.published !== where.published)
          return false;
        return true;
      }).length;
    },
    findUnique: async ({ where }) => {
      if (where.id) return posts.find((p) => p.id === where.id);
      if (where.slug) return posts.find((p) => p.slug === where.slug);
      return undefined;
    },
    create: async (data: Omit<Post, "id" | "createdAt">) => {
      const post: Post = {
        ...data,
        id: posts.length + 1,
        createdAt: new Date().toISOString(),
      };
      posts.push(post);
      return post;
    },
    update: async ({ where, data }) => {
      const index = posts.findIndex((p) => p.id === where.id);
      if (index === -1) return null;
      posts[index] = { ...posts[index], ...data };
      return posts[index];
    },
    delete: async ({ where }) => {
      const index = posts.findIndex((p) => p.id === where.id);
      if (index === -1) return null;
      const deleted = posts[index];
      posts.splice(index, 1);
      return deleted;
    },
  },
  users: {
    findUnique: async ({ where }) => {
      if (where.email) return users.find((u) => u.email === where.email);
      if (where.id) return users.find((u) => u.id === where.id);
      return undefined;
    },
  },
  sessions: {
    create: (userId: number) => {
      const sessionId = Math.random().toString(36).slice(2);
      sessions.set(sessionId, userId);
      return sessionId;
    },
    get: (sessionId: string) => {
      return sessions.get(sessionId);
    },
    delete: (sessionId: string) => {
      sessions.delete(sessionId);
    },
  },
};
```

### Auth Helpers

```typescript
// src/lib/server/auth.ts
import { db } from "./db";

export async function validateSession(sessionId: string | undefined) {
  if (!sessionId) return null;

  const userId = db.sessions.get(sessionId);
  if (!userId) return null;

  const user = await db.users.findUnique({ where: { id: userId } });
  return user || null;
}

export async function login(email: string, password: string) {
  // In production: verify password hash
  const user = await db.users.findUnique({ where: { email } });

  if (!user || password !== "password123") {
    return null;
  }

  const sessionId = db.sessions.create(user.id);
  return { user, sessionId };
}
```

### Hooks

```typescript
// src/hooks.server.ts
import { validateSession } from "$lib/server/auth";
import { redirect } from "@sveltejs/kit";
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  const sessionId = event.cookies.get("session");
  event.locals.user = await validateSession(sessionId);

  // Protect admin routes
  if (event.url.pathname.startsWith("/admin") && !event.locals.user) {
    throw redirect(303, "/login?redirect=" + event.url.pathname);
  }

  const response = await resolve(event);
  return response;
};
```

### Type Definitions

```typescript
// src/app.d.ts
import type { User } from "$lib/server/db";

declare global {
  namespace App {
    interface Locals {
      user: User | null;
    }
  }
}

export {};
```

### Public Blog List

```typescript
// src/routes/(public)/blog/+page.ts
import type { PageLoad } from "./$types";

export const load: PageLoad = async ({ url, fetch }) => {
  const page = Number(url.searchParams.get("page") || "1");
  const limit = 6;

  const response = await fetch(`/api/posts?page=${page}&limit=${limit}`);
  const { posts, total } = await response.json();

  return {
    posts,
    page,
    totalPages: Math.ceil(total / limit),
  };
};
```

```svelte
<!-- src/routes/(public)/blog/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';
	import PostCard from '$lib/components/PostCard.svelte';
	import Pagination from '$lib/components/Pagination.svelte';

	let { data }: { data: PageData } = $props();
</script>

<div class="container mx-auto px-4 py-12">
	<h1 class="text-4xl font-bold mb-8">Blog</h1>

	<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
		{#each data.posts as post}
			<PostCard {post} />
		{/each}
	</div>

	<Pagination currentPage={data.page} totalPages={data.totalPages} baseUrl="/blog" />
</div>
```

### Single Post Page

```typescript
// src/routes/(public)/blog/[slug]/+page.ts
import { error } from "@sveltejs/kit";
import type { PageLoad } from "./$types";

export const load: PageLoad = async ({ params, fetch }) => {
  const response = await fetch(`/api/posts/${params.slug}`);

  if (!response.ok) {
    throw error(404, "Post not found");
  }

  const post = await response.json();
  return { post };
};
```

```svelte
<!-- src/routes/(public)/blog/[slug]/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<svelte:head>
	<title>{data.post.title}</title>
	<meta name="description" content={data.post.excerpt} />
</svelte:head>

<article class="container mx-auto px-4 py-12 max-w-3xl">
	<h1 class="text-5xl font-bold mb-4">{data.post.title}</h1>
	<div class="text-base-content/60 mb-8">{data.post.createdAt}</div>
	<div class="prose lg:prose-xl">
		{@html data.post.content}
	</div>
</article>
```

### Login Page

```svelte
<!-- src/routes/(public)/login/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';

	let loading = $state(false);
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content flex-col">
		<div class="card w-full max-w-sm shadow-2xl bg-base-100">
			<form
				method="POST"
				class="card-body"
				use:enhance={() => {
					loading = true;
					return async ({ update }) => {
						await update();
						loading = false;
					};
				}}
			>
				<h2 class="card-title text-2xl justify-center mb-4">Login</h2>

				<div class="form-control">
					<label class="label">
						<span class="label-text">Email</span>
					</label>
					<input
						type="email"
						name="email"
						value="admin@example.com"
						class="input input-bordered"
						required
					/>
				</div>

				<div class="form-control">
					<label class="label">
						<span class="label-text">Password</span>
					</label>
					<input
						type="password"
						name="password"
						value="password123"
						class="input input-bordered"
						required
					/>
					<label class="label">
						<span class="label-text-alt">Demo: password123</span>
					</label>
				</div>

				<div class="form-control mt-6">
					<button type="submit" class="btn btn-primary" disabled={loading}>
						{loading ? 'Logging in...' : 'Login'}
					</button>
				</div>
			</form>
		</div>
	</div>
</div>
```

```typescript
// src/routes/(public)/login/+server.ts
import { login } from "$lib/server/auth";
import { fail, redirect } from "@sveltejs/kit";
import type { Actions } from "./$types";

export const actions: Actions = {
  default: async ({ request, cookies, url }) => {
    const data = await request.formData();
    const email = data.get("email")?.toString();
    const password = data.get("password")?.toString();

    if (!email || !password) {
      return fail(400, { error: "Missing email or password" });
    }

    const result = await login(email, password);

    if (!result) {
      return fail(401, { error: "Invalid credentials" });
    }

    cookies.set("session", result.sessionId, {
      path: "/",
      httpOnly: true,
      sameSite: "strict",
      maxAge: 60 * 60 * 24 * 7, // 7 days
    });

    const redirectTo = url.searchParams.get("redirect") || "/admin";
    throw redirect(303, redirectTo);
  },
};
```

### Admin Dashboard

```typescript
// src/routes/(admin)/+layout.server.ts
import type { LayoutServerLoad } from "./$types";

export const load: LayoutServerLoad = async ({ locals }) => {
  return {
    user: locals.user,
  };
};
```

```svelte
<!-- src/routes/(admin)/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();
</script>

<div class="drawer lg:drawer-open">
	<input id="admin-drawer" type="checkbox" class="drawer-toggle" />
	<div class="drawer-content flex flex-col">
		<div class="navbar bg-base-300">
			<div class="flex-none lg:hidden">
				<label for="admin-drawer" class="btn btn-square btn-ghost">☰</label>
			</div>
			<div class="flex-1 px-2">
				<span class="text-lg font-bold">Admin Panel</span>
			</div>
			<div class="flex-none gap-2">
				<span class="text-sm">{data.user?.name}</span>
				<form action="/logout" method="POST">
					<button type="submit" class="btn btn-ghost btn-sm">Logout</button>
				</form>
			</div>
		</div>

		<main class="p-6">
			{@render children()}
		</main>
	</div>

	<div class="drawer-side">
		<label for="admin-drawer" class="drawer-overlay"></label>
		<ul class="menu p-4 w-80 min-h-full bg-base-200">
			<li><a href="/admin">Dashboard</a></li>
			<li><a href="/admin/posts">Posts</a></li>
			<li><a href="/">View Site</a></li>
		</ul>
	</div>
</div>
```

```typescript
// src/routes/(admin)/admin/+page.server.ts
import { db } from "$lib/server/db";
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async () => {
  // Run in parallel!
  const [totalPosts, publishedPosts, draftPosts] = await Promise.all([
    db.posts.count({}),
    db.posts.count({ where: { published: true } }),
    db.posts.count({ where: { published: false } }),
  ]);

  return {
    stats: {
      total: totalPosts,
      published: publishedPosts,
      drafts: draftPosts,
    },
  };
};
```

```svelte
<!-- src/routes/(admin)/admin/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<h1 class="text-3xl font-bold mb-6">Dashboard</h1>

<div class="grid md:grid-cols-3 gap-6 mb-8">
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Total Posts</div>
		<div class="stat-value text-primary">{data.stats.total}</div>
	</div>
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Published</div>
		<div class="stat-value text-success">{data.stats.published}</div>
	</div>
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Drafts</div>
		<div class="stat-value text-warning">{data.stats.drafts}</div>
	</div>
</div>

<div class="flex gap-4">
	<a href="/admin/posts/new" class="btn btn-primary">Create New Post</a>
	<a href="/admin/posts" class="btn btn-secondary">Manage Posts</a>
</div>
```

### API Endpoints

```typescript
// src/routes/api/posts/+server.ts
import { json } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import type { RequestHandler } from "./$types";

export const GET: RequestHandler = async ({ url }) => {
  const page = Number(url.searchParams.get("page") || "1");
  const limit = Number(url.searchParams.get("limit") || "10");
  const skip = (page - 1) * limit;

  const [posts, total] = await Promise.all([
    db.posts.findMany({ skip, take: limit, where: { published: true } }),
    db.posts.count({ where: { published: true } }),
  ]);

  return json({ posts, total });
};
```

```typescript
// src/routes/api/posts/[slug]/+server.ts
import { json, error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import type { RequestHandler } from "./$types";

export const GET: RequestHandler = async ({ params }) => {
  const post = await db.posts.findUnique({ where: { slug: params.slug } });

  if (!post) {
    throw error(404, "Post not found");
  }

  return json(post);
};
```

### Components

```svelte
<!-- src/lib/components/PostCard.svelte -->
<script lang="ts">
	import type { Post } from '$lib/server/db';

	let { post }: { post: Post } = $props();
</script>

<div class="card bg-base-100 shadow-xl hover:shadow-2xl transition-shadow">
	<div class="card-body">
		<h2 class="card-title">{post.title}</h2>
		<p class="text-base-content/70">{post.excerpt}</p>
		<div class="card-actions justify-between items-center mt-4">
			<span class="text-sm text-base-content/60">{post.createdAt}</span>
			<a href="/blog/{post.slug}" class="btn btn-primary btn-sm">Read More</a>
		</div>
	</div>
</div>
```

```svelte
<!-- src/lib/components/Pagination.svelte -->
<script lang="ts">
	let {
		currentPage,
		totalPages,
		baseUrl
	}: { currentPage: number; totalPages: number; baseUrl: string } = $props();
</script>

{#if totalPages > 1}
	<div class="flex justify-center mt-8">
		<div class="join">
			{#if currentPage > 1}
				<a href="{baseUrl}?page={currentPage - 1}" class="join-item btn">«</a>
			{/if}

			{#each Array(totalPages) as _, i}
				<a
					href="{baseUrl}?page={i + 1}"
					class="join-item btn"
					class:btn-active={currentPage === i + 1}
				>
					{i + 1}
				</a>
			{/each}

			{#if currentPage < totalPages}
				<a href="{baseUrl}?page={currentPage + 1}" class="join-item btn">»</a>
			{/if}
		</div>
	</div>
{/if}
```

### Error Page

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-6xl font-bold text-error">{$page.status}</h1>
			<p class="text-2xl font-semibold mt-4">{$page.error?.message || 'Something went wrong'}</p>
			<div class="mt-8 flex gap-4 justify-center">
				<a href="/" class="btn btn-primary">Go Home</a>
				<a href="/blog" class="btn btn-secondary">View Blog</a>
			</div>
		</div>
	</div>
</div>
```

### Navigation with Loading Bar

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
	import '../app.css';
	import { beforeNavigate, afterNavigate } from '$app/navigation';

	let loading = $state(false);
	let progress = $state(0);
	let interval: number;

	beforeNavigate(() => {
		loading = true;
		progress = 0;
		interval = setInterval(() => {
			progress = Math.min(progress + 10, 90);
		}, 100);
	});

	afterNavigate(() => {
		clearInterval(interval);
		progress = 100;
		setTimeout(() => {
			loading = false;
			progress = 0;
		}, 300);
	});
</script>

{#if loading}
	<div
		class="fixed top-0 left-0 h-1 bg-primary z-50 transition-all duration-200"
		style="width: {progress}%"
	></div>
{/if}

{@render children()}
```

> **Note:** Add `import type { Snippet } from 'svelte';` and `let { children }: { children: Snippet } = $props();` in the script section.

**Key Features Demonstrated:**

- ✅ **Authentication**: Hooks + locals + protected routes
- ✅ **Type Safety**: Generated types from `./$types`
- ✅ **Data Loading**: Server and universal load functions
- ✅ **API Endpoints**: RESTful API with GET/POST
- ✅ **Pagination**: URL search params
- ✅ **Error Handling**: Custom error pages
- ✅ **Loading States**: Progress bar with navigation
- ✅ **Layout Groups**: Public vs Admin
- ✅ **Parallel Requests**: Promise.all for performance
- ✅ **DaisyUI**: Full component library usage

---

## 📝 Key Takeaways

✅ Load functions fetch data before page renders  
✅ Use `+page.ts` for universal (client + server) loads  
✅ Use `+page.server.ts` for server-only loads (secrets, DB)  
✅ Layout load functions provide data to all children  
✅ `await parent()` accesses parent layout data  
✅ Create API endpoints with `+server.ts`  
✅ Handle hooks run on every request (auth, logging)  
✅ Use `locals` to pass data from hooks to routes  
✅ `error()` throws expected errors with custom messages  
✅ URL search params enable bookmarkable pagination  
✅ `Promise.all()` runs requests in parallel  
✅ Stream slow data with unwrapped promises  
✅ `beforeNavigate` / `afterNavigate` for loading indicators  
✅ Generated types provide full autocomplete

---

## 🚀 Next Steps

You've mastered data loading and API routes! Next up:

- **Section 12**: Forms & Actions - Progressive enhancement with form actions
- **Section 13**: Deployment - Ship your app to production
- Advanced patterns like optimistic UI and real-time updates
- Building production-ready full-stack applications
