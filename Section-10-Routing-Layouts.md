# Section 10: Routing & Layouts

## 📚 Learning Objectives

By the end of this section, you will:

- Master SvelteKit's file-based routing system
- Create and navigate between routes programmatically
- Use dynamic route parameters for flexible URLs
- Build nested layouts that share UI across pages
- Conditionally modify layouts based on route data
- Organize routes with layout groups
- Break out of layout hierarchies when needed
- Handle rest and optional parameters
- Create custom parameter matchers for type-safe routing

---

## Table of Contents

- [Section 10: Routing \& Layouts](#section-10-routing--layouts)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Pages \& Layouts Overview](#1-pages--layouts-overview)
    - [Understanding the Routing System](#understanding-the-routing-system)
  - [2. Creating Routes \& Navigating Between Routes](#2-creating-routes--navigating-between-routes)
    - [File-Based Routing](#file-based-routing)
    - [📁 Basic Routes](#-basic-routes)
    - [Navigation Methods](#navigation-methods)
    - [Quick Example: Navigation Bar](#quick-example-navigation-bar)
  - [3. Dynamic Route Parameters](#3-dynamic-route-parameters)
    - [Building Dynamic Routes](#building-dynamic-routes)
    - [📁 Dynamic Route Structure](#-dynamic-route-structure)
    - [Accessing Parameters](#accessing-parameters)
    - [Loading Data for Dynamic Routes](#loading-data-for-dynamic-routes)
  - [4. Nesting Pages \& Layouts](#4-nesting-pages--layouts)
    - [Creating Nested Layouts](#creating-nested-layouts)
    - [📁 Nested Layout Example](#-nested-layout-example)
    - [Layout Example](#layout-example)
  - [5. Modifying Layouts Conditionally](#5-modifying-layouts-conditionally)
    - [Dynamic Layout Behavior](#dynamic-layout-behavior)
  - [6. Layout Groups](#6-layout-groups)
    - [Organizing Routes with Groups](#organizing-routes-with-groups)
    - [📁 Layout Groups](#-layout-groups)
  - [7. Breaking Out of Layouts Hierarchy](#7-breaking-out-of-layouts-hierarchy)
    - [Resetting Layout Inheritance](#resetting-layout-inheritance)
  - [8. Rest Parameters](#8-rest-parameters)
    - [Catch-All Routes](#catch-all-routes)
  - [9. Optional Parameters](#9-optional-parameters)
    - [Making Parameters Optional](#making-parameters-optional)
  - [10. Parameter Matchers](#10-parameter-matchers)
    - [Type-Safe Route Parameters](#type-safe-route-parameters)
    - [📁 Create a Matcher](#-create-a-matcher)
    - [Use the Matcher](#use-the-matcher)
  - [11. Complete Example: Marketing Site with App Dashboard](#11-complete-example-marketing-site-with-app-dashboard)
    - [Putting It All Together](#putting-it-all-together)
    - [📁 Complete Folder Structure](#-complete-folder-structure)
    - [Root Layout](#root-layout)
    - [Marketing Layout](#marketing-layout)
    - [Marketing Navigation](#marketing-navigation)
    - [Dashboard Layout](#dashboard-layout)
    - [Example Pages](#example-pages)
    - [Parameter Matchers](#parameter-matchers)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Pages & Layouts Overview

### Understanding the Routing System

SvelteKit uses **file-based routing** where the folder structure in `src/routes/` directly maps to URLs.

**Real-World Scenario:** You're building a SaaS application with public marketing pages and a protected dashboard area.

**What it does:** Explains how pages and layouts work together.

**Key Files:**

- `+page.svelte` - A page component (renders at that route)
- `+layout.svelte` - A layout that wraps pages
- `+page.ts` / `+page.server.ts` - Load data for pages
- `+layout.ts` / `+layout.server.ts` - Load data for layouts

**How Routing Works:**

```
src/routes/
├── +layout.svelte          → Wraps all pages
├── +page.svelte            → / (home)
├── about/
│   └── +page.svelte        → /about
└── blog/
    ├── +layout.svelte      → Wraps blog pages
    ├── +page.svelte        → /blog
    └── [slug]/
        └── +page.svelte    → /blog/my-post
```

**Layout Inheritance:**

Layouts automatically wrap all child routes. Each route gets:

1. Root `+layout.svelte` (always applies)
2. Any nested layouts in parent folders
3. The page itself (`+page.svelte`)

> 💡 **Best Practice**: Use layouts for shared UI like headers, navigation, and footers. Keep page-specific content in `+page.svelte`.

**⚠️ Common Mistakes:**

- Don't create a layout for every page - only when you need shared UI
- Don't forget `<slot />` in layouts - it's where child content renders
- Don't put business logic in layouts - use page/layout load functions

---

## 2. Creating Routes & Navigating Between Routes

### File-Based Routing

Create pages by adding files to `src/routes/`.

**Real-World Scenario:** Building a website with home, about, contact, and blog pages.

### 📁 Basic Routes

```
src/routes/
├── +page.svelte           → /
├── about/
│   └── +page.svelte       → /about
├── contact/
│   └── +page.svelte       → /contact
└── pricing/
    └── +page.svelte       → /pricing
```

### Navigation Methods

**1. Using `<a>` tags (recommended for accessibility):**

```svelte
<a href="/" class="btn btn-ghost">Home</a>
<a href="/about" class="btn btn-ghost">About</a>
<a href="/pricing" class="btn btn-ghost">Pricing</a>
```

**2. Using `goto()` for programmatic navigation:**

```svelte
<script lang="ts">
	import { goto } from '$app/navigation';

	function handleSubmit() {
		// Process form...
		goto('/success');
	}

	function goBack() {
		goto(-1); // Go back one page
	}
</script>

<button onclick={handleSubmit} class="btn btn-primary">Submit</button>
<button onclick={goBack} class="btn btn-ghost">Back</button>
```

**3. Using `data-sveltekit-preload-data` for faster navigation:**

```svelte
<a href="/blog" data-sveltekit-preload-data="hover" class="btn"> Blog (preloads on hover) </a>
```

### Quick Example: Navigation Bar

```svelte
<!-- src/lib/components/Navbar.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	const isActive = (path: string) => $page.url.pathname === path;
</script>

<div class="navbar bg-base-100 shadow-lg">
	<div class="navbar-start">
		<a href="/" class="btn btn-ghost text-xl">MySite</a>
	</div>
	<div class="navbar-center hidden lg:flex">
		<ul class="menu menu-horizontal px-1">
			<li><a href="/" class:active={isActive('/')}>Home</a></li>
			<li><a href="/about" class:active={isActive('/about')}>About</a></li>
			<li><a href="/pricing" class:active={isActive('/pricing')}>Pricing</a></li>
			<li><a href="/contact" class:active={isActive('/contact')}>Contact</a></li>
		</ul>
	</div>
</div>
```

> 💡 **Best Practice**: Use `<a>` tags for navigation when possible. SvelteKit automatically handles them as client-side navigation without full page reloads.

**⚠️ Common Mistakes:**

- Don't use `<Link>` components (this isn't React!)
- Don't use `onClick` handlers for basic navigation - use `<a>` tags
- Don't forget to import `goto` from `$app/navigation` (not `$app/stores`)

---

## 3. Dynamic Route Parameters

### Building Dynamic Routes

Use `[brackets]` in folder names to create dynamic segments.

**Real-World Scenario:** Blog posts, user profiles, product pages - anything with dynamic IDs.

**What it does:** Captures URL segments as parameters.

### 📁 Dynamic Route Structure

```
src/routes/
├── blog/
│   └── [slug]/
│       └── +page.svelte        → /blog/my-post
├── users/
│   └── [userId]/
│       ├── +page.svelte        → /users/123
│       └── posts/
│           └── [postId]/
│               └── +page.svelte → /users/123/posts/456
└── products/
    └── [id]/
        └── +page.svelte        → /products/abc-123
```

### Accessing Parameters

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	// Access the slug parameter
	const slug = $page.params.slug;
</script>

<div class="container mx-auto p-8">
	<article class="prose">
		<h1>Blog Post: {slug}</h1>
		<p>This is the content for {slug}</p>
	</article>
</div>
```

### Loading Data for Dynamic Routes

```typescript
// src/routes/blog/[slug]/+page.ts
export const load = ({ params }) => {
	return {
		post: {
			slug: params.slug,
			title: `Post about ${params.slug}`,
			content: 'This would come from your database...'
		}
	};
};
```

```svelte
<!-- src/routes/blog/[slug]/+page.svelte -->
<script lang="ts">
	let { data } = $props();
</script>

<article class="prose max-w-none">
	<h1>{data.post.title}</h1>
	<div>{data.post.content}</div>
</article>
```

> 💡 **Best Practice**: Use `+page.ts` for client-side data loading and `+page.server.ts` for server-only operations (database queries, API calls with secrets).

---

## 4. Nesting Pages & Layouts

### Creating Nested Layouts

Layouts can be nested to create hierarchies of shared UI.

**Real-World Scenario:** Your app has a public site layout and a different dashboard layout.

### 📁 Nested Layout Example

```
src/routes/
├── +layout.svelte              → Root layout (all pages)
├── +page.svelte                → Home
├── (marketing)/                → Layout group
│   ├── +layout.svelte          → Marketing layout
│   ├── about/
│   │   └── +page.svelte        → /about
│   └── pricing/
│       └── +page.svelte        → /pricing
└── dashboard/
    ├── +layout.svelte          → Dashboard layout
    ├── +page.svelte            → /dashboard
    ├── settings/
    │   └── +page.svelte        → /dashboard/settings
    └── analytics/
        └── +page.svelte        → /dashboard/analytics
```

### Layout Example

```svelte
<!-- src/routes/dashboard/+layout.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

<div class="drawer lg:drawer-open">
	<input id="drawer" type="checkbox" class="drawer-toggle" />
	<div class="drawer-content flex flex-col">
		<!-- Navbar -->
		<div class="navbar bg-base-300 w-full">
			<div class="flex-none lg:hidden">
				<label for="drawer" class="btn btn-square btn-ghost">
					<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 6h16M4 12h16M4 18h16"
						/>
					</svg>
				</label>
			</div>
			<div class="flex-1 px-2 mx-2">Dashboard</div>
		</div>

		<!-- Page content -->
		<div class="p-4">
			<slot />
		</div>
	</div>

	<!-- Sidebar -->
	<div class="drawer-side">
		<label for="drawer" class="drawer-overlay"></label>
		<ul class="menu p-4 w-80 min-h-full bg-base-200">
			<li><a href="/dashboard" class:active={$page.url.pathname === '/dashboard'}>Overview</a></li>
			<li>
				<a href="/dashboard/analytics" class:active={$page.url.pathname === '/dashboard/analytics'}
					>Analytics</a
				>
			</li>
			<li>
				<a href="/dashboard/settings" class:active={$page.url.pathname === '/dashboard/settings'}
					>Settings</a
				>
			</li>
		</ul>
	</div>
</div>
```

---

## 5. Modifying Layouts Conditionally

### Dynamic Layout Behavior

Change layout behavior based on route data or user state.

**Real-World Scenario:** Hide the sidebar when viewing certain pages or for certain user roles.

```svelte
<!-- src/routes/dashboard/+layout.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	let { data } = $props();

	// Conditionally show sidebar based on route or user
	const showSidebar = $derived(!$page.url.pathname.includes('/fullscreen'));
</script>

<div class="drawer" class:lg:drawer-open={showSidebar}>
	{#if showSidebar}
		<div class="drawer-side">
			<!-- Sidebar content -->
		</div>
	{/if}

	<div class="drawer-content">
		<slot />
	</div>
</div>
```

---

## 6. Layout Groups

### Organizing Routes with Groups

Use `(parentheses)` to group routes without affecting the URL.

**Real-World Scenario:** Organize marketing pages separately from app pages without adding URL segments.

### 📁 Layout Groups

```
src/routes/
├── (marketing)/           → Group name doesn't appear in URL
│   ├── +layout.svelte     → Shared marketing layout
│   ├── +page.svelte       → /  (home)
│   ├── about/
│   │   └── +page.svelte   → /about
│   └── pricing/
│       └── +page.svelte   → /pricing
└── (app)/                 → Different group
    ├── +layout.svelte     → Shared app layout
    ├── dashboard/
    │   └── +page.svelte   → /dashboard
    └── settings/
        └── +page.svelte   → /settings
```

**Benefits:**

- ✅ Organize code logically
- ✅ Apply different layouts without URL nesting
- ✅ Keep URLs clean
- ✅ Separate concerns (marketing vs app)

---

## 7. Breaking Out of Layouts Hierarchy

### Resetting Layout Inheritance

Use `+layout@.svelte` to break out of parent layouts.

**Real-World Scenario:** A login page that shouldn't have the dashboard sidebar.

```
src/routes/
├── +layout.svelte                    → Root layout
└── dashboard/
    ├── +layout.svelte                → Dashboard layout with sidebar
    ├── +page.svelte                  → Uses dashboard layout
    └── login/
        ├── +layout@.svelte           → Breaks out, only uses root layout
        └── +page.svelte              → /dashboard/login (no sidebar)
```

```svelte
<!-- src/routes/dashboard/login/+layout@.svelte -->
<div class="min-h-screen flex items-center justify-center bg-base-200">
	<slot />
</div>
```

---

## 8. Rest Parameters

### Catch-All Routes

Use `[...rest]` to match multiple path segments.

**Real-World Scenario:** Documentation site with deep nesting, file browsers, or catch-all 404 pages.

```
src/routes/
└── docs/
    └── [...path]/
        └── +page.svelte   → /docs/any/path/here
```

```svelte
<!-- src/routes/docs/[...path]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	const pathSegments = $page.params.path?.split('/') || [];
</script>

<div class="breadcrumbs">
	<ul>
		<li><a href="/docs">Docs</a></li>
		{#each pathSegments as segment}
			<li>{segment}</li>
		{/each}
	</ul>
</div>
```

---

## 9. Optional Parameters

### Making Parameters Optional

Use `[[double-brackets]]` for optional parameters.

**Real-World Scenario:** Language selection, pagination, or optional filters.

```
src/routes/
└── [[lang]]/
    └── +page.svelte   → / or /en or /es
```

```typescript
// src/routes/[[lang]]/+page.ts
export const load = ({ params }) => {
	const lang = params.lang || 'en'; // Default to 'en'

	return {
		lang,
		greeting: lang === 'es' ? 'Hola' : 'Hello'
	};
};
```

---

## 10. Parameter Matchers

### Type-Safe Route Parameters

Create custom matchers to validate route parameters.

**Real-World Scenario:** Ensure IDs are numeric or slugs match a specific pattern.

### 📁 Create a Matcher

```typescript
// src/params/integer.ts
import type { ParamMatcher } from '@sveltejs/kit';

export const match: ParamMatcher = (param) => {
	return /^\d+$/.test(param);
};
```

### Use the Matcher

```
src/routes/
└── users/
    └── [id=integer]/
        └── +page.svelte   → Only matches /users/123, not /users/abc
```

```svelte
<!-- src/routes/users/[id=integer]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	// id is guaranteed to be a number string
	const userId = $page.params.id;
</script>

<div class="card">
	<h2>User #{userId}</h2>
</div>
```

**Common Matchers:**

```typescript
// src/params/uuid.ts
export const match = (param) => {
	return /^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i.test(param);
};

// src/params/slug.ts
export const match = (param) => {
	return /^[a-z0-9-]+$/.test(param);
};
```

---

## 11. Complete Example: Marketing Site with App Dashboard

### Putting It All Together

Let's build a complete example combining all routing concepts!

**Real-World Scenario:** A SaaS product with marketing pages, blog, docs, and an authenticated dashboard.

### 📁 Complete Folder Structure

```
src/
├── params/
│   ├── integer.ts
│   └── slug.ts
├── lib/
│   └── components/
│       ├── MarketingNav.svelte
│       ├── DashboardNav.svelte
│       └── Footer.svelte
└── routes/
    ├── +layout.svelte                        → Root layout
    ├── (marketing)/                          → Marketing group
    │   ├── +layout.svelte                    → Marketing layout
    │   ├── +page.svelte                      → / (home)
    │   ├── about/
    │   │   └── +page.svelte                  → /about
    │   ├── pricing/
    │   │   └── +page.svelte                  → /pricing
    │   ├── blog/
    │   │   ├── +page.svelte                  → /blog (list)
    │   │   └── [slug=slug]/
    │   │       └── +page.svelte              → /blog/my-post
    │   └── docs/
    │       └── [...path]/
    │           └── +page.svelte              → /docs/guide/intro
    ├── (app)/                                → App group
    │   ├── +layout.svelte                    → App layout (auth check)
    │   ├── dashboard/
    │   │   ├── +layout.svelte                → Dashboard layout (sidebar)
    │   │   ├── +page.svelte                  → /dashboard
    │   │   ├── projects/
    │   │   │   ├── +page.svelte              → /dashboard/projects
    │   │   │   └── [id=integer]/
    │   │   │       └── +page.svelte          → /dashboard/projects/123
    │   │   ├── settings/
    │   │   │   └── [[tab]]/
    │   │   │       └── +page.svelte          → /dashboard/settings or /dashboard/settings/billing
    │   │   └── analytics/
    │   │       └── +page.svelte              → /dashboard/analytics
    │   └── login/
    │       ├── +layout@.svelte               → Break out of app layout
    │       └── +page.svelte                  → /login (no sidebar)
    └── +error.svelte                         → Error page
```

### Root Layout

```svelte
<!-- src/routes/+layout.svelte -->
<script lang="ts">
	import '../app.css';
</script>

<slot />
```

### Marketing Layout

```svelte
<!-- src/routes/(marketing)/+layout.svelte -->
<script lang="ts">
	import MarketingNav from '$lib/components/MarketingNav.svelte';
	import Footer from '$lib/components/Footer.svelte';
</script>

<div class="min-h-screen flex flex-col">
	<MarketingNav />
	<main class="flex-1">
		<slot />
	</main>
	<Footer />
</div>
```

### Marketing Navigation

```svelte
<!-- src/lib/components/MarketingNav.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	const isActive = (path: string) => $page.url.pathname === path;
</script>

<div class="navbar bg-base-100 shadow-lg">
	<div class="navbar-start">
		<a href="/" class="btn btn-ghost text-xl">🚀 SaaSKit</a>
	</div>
	<div class="navbar-center hidden lg:flex">
		<ul class="menu menu-horizontal px-1">
			<li><a href="/" class:active={isActive('/')}>Home</a></li>
			<li><a href="/about" class:active={isActive('/about')}>About</a></li>
			<li><a href="/pricing" class:active={isActive('/pricing')}>Pricing</a></li>
			<li><a href="/blog" class:active={isActive('/blog')}>Blog</a></li>
			<li><a href="/docs" class:active={$page.url.pathname.startsWith('/docs')}>Docs</a></li>
		</ul>
	</div>
	<div class="navbar-end gap-2">
		<a href="/login" class="btn btn-ghost">Login</a>
		<a href="/dashboard" class="btn btn-primary">Dashboard</a>
	</div>
</div>
```

### Dashboard Layout

```svelte
<!-- src/routes/(app)/dashboard/+layout.svelte -->
<script lang="ts">
	import DashboardNav from '$lib/components/DashboardNav.svelte';
	import { page } from '$app/stores';
</script>

<div class="drawer lg:drawer-open">
	<input id="drawer" type="checkbox" class="drawer-toggle" />
	<div class="drawer-content flex flex-col">
		<!-- Top navbar -->
		<div class="navbar bg-base-300 w-full">
			<div class="flex-none lg:hidden">
				<label for="drawer" class="btn btn-square btn-ghost">
					<svg class="w-6 h-6" fill="none" stroke="currentColor" viewBox="0 0 24 24">
						<path
							stroke-linecap="round"
							stroke-linejoin="round"
							stroke-width="2"
							d="M4 6h16M4 12h16M4 18h16"
						/>
					</svg>
				</label>
			</div>
			<div class="flex-1 px-2">
				<h1 class="text-xl font-bold">Dashboard</h1>
			</div>
			<div class="flex-none">
				<div class="dropdown dropdown-end">
					<label tabindex="0" class="btn btn-ghost btn-circle avatar placeholder">
						<div class="bg-neutral text-neutral-content rounded-full w-10">
							<span>JD</span>
						</div>
					</label>
					<ul class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52 mt-4">
						<li><a href="/dashboard/settings">Settings</a></li>
						<li><a href="/logout">Logout</a></li>
					</ul>
				</div>
			</div>
		</div>

		<!-- Page content -->
		<main class="flex-1 p-6 bg-base-200">
			<slot />
		</main>
	</div>

	<!-- Sidebar -->
	<div class="drawer-side">
		<label for="drawer" class="drawer-overlay"></label>
		<aside class="menu p-4 w-80 min-h-full bg-base-100">
			<ul>
				<li>
					<a href="/dashboard" class:active={$page.url.pathname === '/dashboard'}>
						<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M3 12l2-2m0 0l7-7 7 7M5 10v10a1 1 0 001 1h3m10-11l2 2m-2-2v10a1 1 0 01-1 1h-3m-6 0a1 1 0 001-1v-4a1 1 0 011-1h2a1 1 0 011 1v4a1 1 0 001 1m-6 0h6"
							/>
						</svg>
						Overview
					</a>
				</li>
				<li>
					<a
						href="/dashboard/projects"
						class:active={$page.url.pathname.startsWith('/dashboard/projects')}
					>
						<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M3 7v10a2 2 0 002 2h14a2 2 0 002-2V9a2 2 0 00-2-2h-6l-2-2H5a2 2 0 00-2 2z"
							/>
						</svg>
						Projects
					</a>
				</li>
				<li>
					<a
						href="/dashboard/analytics"
						class:active={$page.url.pathname === '/dashboard/analytics'}
					>
						<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"
							/>
						</svg>
						Analytics
					</a>
				</li>
				<li>
					<a
						href="/dashboard/settings"
						class:active={$page.url.pathname.startsWith('/dashboard/settings')}
					>
						<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M10.325 4.317c.426-1.756 2.924-1.756 3.35 0a1.724 1.724 0 002.573 1.066c1.543-.94 3.31.826 2.37 2.37a1.724 1.724 0 001.065 2.572c1.756.426 1.756 2.924 0 3.35a1.724 1.724 0 00-1.066 2.573c.94 1.543-.826 3.31-2.37 2.37a1.724 1.724 0 00-2.572 1.065c-.426 1.756-2.924 1.756-3.35 0a1.724 1.724 0 00-2.573-1.066c-1.543.94-3.31-.826-2.37-2.37a1.724 1.724 0 00-1.065-2.572c-1.756-.426-1.756-2.924 0-3.35a1.724 1.724 0 001.066-2.573c-.94-1.543.826-3.31 2.37-2.37.996.608 2.296.07 2.572-1.065z"
							/>
							<path
								stroke-linecap="round"
								stroke-linejoin="round"
								stroke-width="2"
								d="M15 12a3 3 0 11-6 0 3 3 0 016 0z"
							/>
						</svg>
						Settings
					</a>
				</li>
			</ul>
		</aside>
	</div>
</div>
```

### Example Pages

```svelte
<!-- src/routes/(marketing)/+page.svelte -->
<div
	class="hero min-h-screen"
	style="background-image: url(https://images.unsplash.com/photo-1451187580459-43490279c0fa?w=1920);"
>
	<div class="hero-overlay bg-opacity-60"></div>
	<div class="hero-content text-center text-neutral-content">
		<div class="max-w-md">
			<h1 class="mb-5 text-5xl font-bold">Welcome to SaaSKit</h1>
			<p class="mb-5">The fastest way to build and ship your next SaaS product.</p>
			<a href="/dashboard" class="btn btn-primary btn-lg">Get Started</a>
		</div>
	</div>
</div>
```

```svelte
<!-- src/routes/(marketing)/blog/[slug=slug]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	const slug = $page.params.slug;
</script>

<div class="container mx-auto px-4 py-12">
	<article class="prose lg:prose-xl mx-auto">
		<h1>{slug.replace(/-/g, ' ')}</h1>
		<div class="text-sm text-base-content/60 mb-8">
			Published on {new Date().toLocaleDateString()}
		</div>
		<p>
			This is a blog post about {slug}. In a real app, this content would come from a CMS or
			database.
		</p>
	</article>
</div>
```

```svelte
<!-- src/routes/(app)/dashboard/+page.svelte -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6">
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Total Views</div>
		<div class="stat-value text-primary">89,400</div>
		<div class="stat-desc">21% more than last month</div>
	</div>

	<div class="stat bg-base-100 shadow">
		<div class="stat-title">New Users</div>
		<div class="stat-value text-secondary">2,100</div>
		<div class="stat-desc">↗︎ 40 (2%)</div>
	</div>

	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Active Projects</div>
		<div class="stat-value">12</div>
		<div class="stat-desc">↘︎ 2 (14%)</div>
	</div>

	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Revenue</div>
		<div class="stat-value text-accent">$12,845</div>
		<div class="stat-desc">↗︎ $145 (1.2%)</div>
	</div>
</div>

<div class="mt-8">
	<h2 class="text-2xl font-bold mb-4">Recent Activity</h2>
	<div class="bg-base-100 shadow rounded-lg p-6">
		<p class="text-base-content/60">Your recent activity will appear here...</p>
	</div>
</div>
```

```svelte
<!-- src/routes/(app)/dashboard/projects/[id=integer]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
	import { goto } from '$app/navigation';

	const projectId = $page.params.id;
</script>

<div class="flex items-center gap-4 mb-6">
	<button onclick={() => goto('/dashboard/projects')} class="btn btn-ghost btn-sm">
		← Back to Projects
	</button>
</div>

<div class="card bg-base-100 shadow-xl">
	<div class="card-body">
		<h2 class="card-title text-3xl">Project #{projectId}</h2>
		<div class="divider"></div>
		<div class="grid grid-cols-2 gap-4">
			<div>
				<h3 class="font-bold mb-2">Details</h3>
				<p class="text-sm">Created: {new Date().toLocaleDateString()}</p>
				<p class="text-sm">Status: Active</p>
			</div>
			<div>
				<h3 class="font-bold mb-2">Actions</h3>
				<button class="btn btn-primary btn-sm">Edit</button>
				<button class="btn btn-error btn-sm ml-2">Delete</button>
			</div>
		</div>
	</div>
</div>
```

```svelte
<!-- src/routes/(app)/dashboard/settings/[[tab]]/+page.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	const tab = $page.params.tab || 'profile';
	const tabs = ['profile', 'billing', 'notifications', 'security'];
</script>

<div class="tabs tabs-boxed mb-6">
	{#each tabs as t}
		<a href="/dashboard/settings/{t}" class="tab" class:tab-active={tab === t}>
			{t.charAt(0).toUpperCase() + t.slice(1)}
		</a>
	{/each}
</div>

<div class="card bg-base-100 shadow-xl">
	<div class="card-body">
		<h2 class="card-title capitalize">{tab} Settings</h2>
		<div class="divider"></div>
		<p>Settings for {tab} would appear here...</p>
	</div>
</div>
```

```svelte
<!-- src/routes/(app)/login/+page.svelte -->
<div class="card w-96 bg-base-100 shadow-2xl">
	<div class="card-body">
		<h2 class="card-title text-2xl justify-center mb-4">Login</h2>
		<div class="form-control">
			<label class="label">
				<span class="label-text">Email</span>
			</label>
			<input type="email" placeholder="email@example.com" class="input input-bordered" />
		</div>
		<div class="form-control">
			<label class="label">
				<span class="label-text">Password</span>
			</label>
			<input type="password" placeholder="••••••••" class="input input-bordered" />
			<label class="label">
				<a href="/forgot-password" class="label-text-alt link link-hover">Forgot password?</a>
			</label>
		</div>
		<div class="form-control mt-6">
			<button class="btn btn-primary">Login</button>
		</div>
		<div class="divider">OR</div>
		<p class="text-center text-sm">
			Don't have an account? <a href="/signup" class="link link-primary">Sign up</a>
		</p>
	</div>
</div>
```

### Parameter Matchers

```typescript
// src/params/integer.ts
import type { ParamMatcher } from '@sveltejs/kit';

export const match: ParamMatcher = (param) => {
	return /^\d+$/.test(param);
};
```

```typescript
// src/params/slug.ts
import type { ParamMatcher } from '@sveltejs/kit';

export const match: ParamMatcher = (param) => {
	return /^[a-z0-9-]+$/.test(param);
};
```

**Key Features Demonstrated:**

- ✅ Layout groups `(marketing)` and `(app)`
- ✅ Nested layouts (dashboard with sidebar)
- ✅ Dynamic routes `[slug]` and `[id]`
- ✅ Parameter matchers `[slug=slug]` and `[id=integer]`
- ✅ Optional parameters `[[tab]]`
- ✅ Breaking out of layouts `+layout@.svelte` for login
- ✅ Navigation with active states
- ✅ DaisyUI components throughout
- ✅ Responsive design with drawer sidebar

> 💡 **Best Practice**: This structure scales to enterprise applications. Add more groups as needed (e.g., `(admin)`, `(api)`).

---

## 📝 Key Takeaways

✅ File-based routing maps folders to URLs automatically  
✅ Use `+page.svelte` for pages and `+layout.svelte` for shared UI  
✅ `[brackets]` create dynamic route parameters  
✅ `(parentheses)` create layout groups without affecting URLs  
✅ `+layout@.svelte` breaks out of parent layouts  
✅ `[...rest]` matches multiple path segments  
✅ `[[optional]]` makes parameters optional  
✅ Parameter matchers validate route parameters  
✅ Nested layouts create UI hierarchies  
✅ The `$page` store provides route information  
✅ Use `<a>` tags for navigation (SvelteKit handles them)  
✅ `goto()` enables programmatic navigation

---

## 🚀 Next Steps

Now that you've mastered SvelteKit routing and layouts, you're ready for:

- **Section 11**: Loading Data - Learn about `+page.ts`, `+page.server.ts`, and data flow
- **Section 12**: Form Actions - Handle form submissions server-side
- **Section 13**: API Routes - Build RESTful APIs with `+server.ts`
- Building full-stack applications with authentication
- Implementing protected routes and role-based access control
