# Section 15: Database Queries & Data Display

## 📚 Learning Objectives

By the end of this section, you will:

- Execute type-safe database queries with Drizzle
- Query data in layout load functions
- Display fetched data with proper TypeScript types
- Build complex page structures with nested layouts
- Implement workspace and page management UI
- Handle loading states and errors gracefully
- Build a complete collaborative workspace platform

---

## Table of Contents

- [Section 15: Database Queries \& Data Display](#section-15-database-queries--data-display)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Your First Database Query](#1-your-first-database-query)
    - [Fetching Data from PostgreSQL](#fetching-data-from-postgresql)
  - [2. Querying App Layout Data](#2-querying-app-layout-data)
    - [Loading User \& Navigation Data](#loading-user--navigation-data)
  - [3. Displaying Fetched Layout Data](#3-displaying-fetched-layout-data)
    - [Rendering Database Content](#rendering-database-content)
  - [4. App Route Markup Walkthrough](#4-app-route-markup-walkthrough)
    - [Building the Main Layout](#building-the-main-layout)
  - [5. Workspaces \& Pages Layout](#5-workspaces--pages-layout)
    - [Nested Workspace Structure](#nested-workspace-structure)
  - [6. Creating the Page Route](#6-creating-the-page-route)
    - [Individual Page Display](#individual-page-display)
  - [7. Complete Example: Collaborative Workspace Platform](#7-complete-example-collaborative-workspace-platform)
    - [Full CRUD with Database Integration](#full-crud-with-database-integration)
    - [📁 Project Structure](#-project-structure)
    - [Workspace Dashboard](#workspace-dashboard)
    - [Create Page](#create-page)
    - [Search API](#search-api)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Your First Database Query

### Fetching Data from PostgreSQL

Let's query real data from your database.

**Real-World Scenario:** Fetch all workspaces for the homepage.

```typescript
// src/routes/+page.server.ts
import { db } from "$lib/server/db";
import { workspaces } from "$lib/server/db/schema";
import { desc } from "drizzle-orm";
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async () => {
  // Your first query!
  // select() with no args = SELECT * (all columns)
  // Type-safe: TypeScript knows exact shape of returned data
  const allWorkspaces = await db
    .select()
    .from(workspaces)
    // orderBy + desc = newest first (ORDER BY created_at DESC)
    .orderBy(desc(workspaces.createdAt))
    // limit(10) = only return 10 rows (LIMIT 10)
    .limit(10);

  // Return data to page component
  // SvelteKit automatically serializes for client
  return {
    workspaces: allWorkspaces,
  };
};
```

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<div class="container mx-auto px-4 py-12">
	<h1 class="text-4xl font-bold mb-8">Workspaces</h1>

	<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
		{#each data.workspaces as workspace}
			<div class="card bg-base-100 shadow-xl">
				<div class="card-body">
					<h2 class="card-title">{workspace.name}</h2>
					<p>{workspace.description || 'No description'}</p>
					<div class="card-actions justify-end">
						<a href="/workspaces/{workspace.slug}" class="btn btn-primary btn-sm"> Open </a>
					</div>
				</div>
			</div>
		{/each}
	</div>
</div>
```

**Query methods:**

```typescript
// Select all
const all = await db.select().from(workspaces);

// Select specific columns
const names = await db.select({ name: workspaces.name }).from(workspaces);

// Where clause
const active = await db
  .select()
  .from(workspaces)
  .where(eq(workspaces.isActive, true));

// Join
const withOwners = await db
  .select()
  .from(workspaces)
  .innerJoin(users, eq(workspaces.ownerId, users.id));

// Count
const [{ count }] = await db.select({ count: count() }).from(workspaces);
```

---

## 2. Querying App Layout Data

### Loading User & Navigation Data

Layout load functions provide data to all child pages.

**Real-World Scenario:** Load user info and their workspaces in the app layout.

```typescript
// src/routes/(app)/+layout.server.ts
import { redirect } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { workspaces, workspaceMembers } from "$lib/server/db/schema";
import { eq } from "drizzle-orm";
import type { LayoutServerLoad } from "./$types";

export const load: LayoutServerLoad = async ({ locals }) => {
  // Check authentication (set in hooks)
  if (!locals.user) {
    throw redirect(303, "/login");
  }

  // Query user's workspaces with JOIN
  // Select specific columns (not SELECT *)
  const userWorkspaces = await db
    .select({
      // Pick columns from workspace table
      id: workspaces.id,
      name: workspaces.name,
      slug: workspaces.slug,
      // Pick role from members table
      role: workspaceMembers.role,
    })
    // Start from workspaceMembers (the join table)
    .from(workspaceMembers)
    // INNER JOIN workspaces ON workspace_members.workspace_id = workspaces.id
    // innerJoin only returns rows where match exists in BOTH tables
    .innerJoin(workspaces, eq(workspaceMembers.workspaceId, workspaces.id))
    // Filter to current user's memberships only
    .where(eq(workspaceMembers.userId, locals.user.id))
    // Sort alphabetically by workspace name
    .orderBy(workspaces.name);

  return {
    user: locals.user,
    workspaces: userWorkspaces,
  };
};
```

> 💡 **Best Practice**: Query navigation data in layout loads so it's available on all pages.

---

## 3. Displaying Fetched Layout Data

### Rendering Database Content

Use layout data in your UI components.

```svelte
<!-- src/routes/(app)/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import { page } from '$app/stores';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();

	let sidebarOpen = $state(false);
</script>

<div class="drawer lg:drawer-open">
	<input id="sidebar" type="checkbox" bind:checked={sidebarOpen} class="drawer-toggle" />

	<div class="drawer-content flex flex-col">
		<!-- Navbar -->
		<div class="navbar bg-base-300">
			<div class="flex-none lg:hidden">
				<label for="sidebar" class="btn btn-square btn-ghost">
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
				<span class="text-lg font-bold">Workspace App</span>
			</div>
			<div class="flex-none gap-2">
				<div class="dropdown dropdown-end">
					<label tabindex="0" class="btn btn-ghost btn-circle avatar">
						<div class="w-10 rounded-full">
							<img src={data.user.avatarUrl || 'https://i.pravatar.cc/150'} alt={data.user.name} />
						</div>
					</label>
					<ul
						tabindex="0"
						class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52 mt-4"
					>
						<li><a href="/settings">Settings</a></li>
						<li><form action="/logout" method="POST"><button>Logout</button></form></li>
					</ul>
				</div>
			</div>
		</div>

		<!-- Page content -->
		<main class="flex-1 p-6">
			{@render children()}
		</main>
	</div>

	<!-- Sidebar -->
	<div class="drawer-side">
		<label for="sidebar" class="drawer-overlay"></label>
		<aside class="bg-base-200 w-80 min-h-full">
			<div class="p-4">
				<h2 class="text-lg font-bold mb-4">Your Workspaces</h2>
				<ul class="menu">
					{#each data.workspaces as workspace}
						<li>
							<a
								href="/workspaces/{workspace.slug}"
								class:active={$page.url.pathname.startsWith(`/workspaces/${workspace.slug}`)}
							>
								<div class="flex flex-col items-start">
									<span class="font-medium">{workspace.name}</span>
									<span class="badge badge-xs badge-primary">{workspace.role}</span>
								</div>
							</a>
						</li>
					{/each}
				</ul>

				<div class="divider"></div>

				<a href="/workspaces/new" class="btn btn-primary btn-sm btn-block"> + New Workspace </a>
			</div>
		</aside>
	</div>
</div>
```

**Key Concepts:**

- Layout data available via `data` prop
- Active states with `$page.url.pathname`
- Responsive sidebar with drawer component
- User avatar from database

---

## 4. App Route Markup Walkthrough

### Building the Main Layout

Structure your app with a clear hierarchy.

```
src/routes/
├── (marketing)/        # Public pages
│   ├── +layout.svelte
│   └── +page.svelte
└── (app)/             # Protected app
    ├── +layout.server.ts   # Load user + workspaces
    ├── +layout.svelte      # App shell (sidebar, navbar)
    └── workspaces/
        └── [slug]/
            ├── +layout.server.ts
            ├── +layout.svelte
            └── +page.svelte
```

**App layout structure:**

```svelte
<!-- src/routes/(app)/+layout.svelte -->
<script lang=\"ts\">
\timport type { Snippet } from 'svelte';\n\timport type { LayoutData } from './$types';

\tlet { data, children }: { data: LayoutData; children: Snippet } = $props();
</script>

<div class="app-container">
	<!-- Top Navigation -->
	<nav class="navbar">
		<WorkspaceSwitcher workspaces={data.workspaces} />
		<UserMenu user={data.user} />
	</nav>

	<!-- Sidebar + Content -->
	<div class="flex">
		<aside class="sidebar">
			<Navigation />
		</aside>
		<main class="content">
			{@render children()}
		</main>
	</div>
</div>
```

---

## 5. Workspaces & Pages Layout

### Nested Workspace Structure

Load workspace-specific data in a nested layout.

```typescript
// src/routes/(app)/workspaces/[slug]/+layout.server.ts
import { error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { workspaces, pages, workspaceMembers } from "$lib/server/db/schema";
import { eq, and, desc } from "drizzle-orm";
import type { LayoutServerLoad } from "./$types";

export const load: LayoutServerLoad = async ({ params, locals, parent }) => {
  // Get parent data (user info)
  await parent();

  // Find workspace by slug
  const [workspace] = await db
    .select()
    .from(workspaces)
    .where(eq(workspaces.slug, params.slug))
    .limit(1);

  if (!workspace) {
    throw error(404, "Workspace not found");
  }

  // Check if user is a member
  const [membership] = await db
    .select()
    .from(workspaceMembers)
    .where(
      and(
        eq(workspaceMembers.workspaceId, workspace.id),
        eq(workspaceMembers.userId, locals.user!.id)
      )
    )
    .limit(1);

  if (!membership) {
    throw error(403, "You are not a member of this workspace");
  }

  // Get workspace pages
  const workspacePages = await db
    .select({
      id: pages.id,
      title: pages.title,
      isPublished: pages.isPublished,
      updatedAt: pages.updatedAt,
    })
    .from(pages)
    .where(eq(pages.workspaceId, workspace.id))
    .orderBy(desc(pages.updatedAt));

  return {
    workspace,
    pages: workspacePages,
    userRole: membership.role,
  };
};
```

```svelte
<!-- src/routes/(app)/workspaces/[slug]/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import { page } from '$app/stores';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();

	let canEdit = $derived(['owner', 'admin'].includes(data.userRole));
</script>

<div class="workspace-layout">
	<!-- Workspace Header -->
	<div class="border-b bg-base-200 px-6 py-4">
		<div class="flex justify-between items-center">
			<div>
				<h1 class="text-2xl font-bold">{data.workspace.name}</h1>
				<p class="text-base-content/60">{data.workspace.description}</p>
			</div>
			{#if canEdit}
				<a href="/workspaces/{data.workspace.slug}/settings" class="btn btn-ghost btn-sm">
					Settings
				</a>
			{/if}
		</div>
	</div>

	<div class="flex">
		<!-- Pages Sidebar -->
		<aside class="w-64 border-r bg-base-100 p-4">
			<div class="flex justify-between items-center mb-4">
				<h2 class="font-bold">Pages</h2>
				{#if canEdit}
					<a href="/workspaces/{data.workspace.slug}/pages/new" class="btn btn-xs btn-primary">
						+ New
					</a>
				{/if}
			</div>

			<ul class="menu menu-compact">
				{#each data.pages as page}
					<li>
						<a
							href="/workspaces/{data.workspace.slug}/pages/{page.id}"
							class:active={$page.params.pageId === page.id.toString()}
						>
							<div class="flex-1">
								{page.title}
								{#if !page.isPublished}
									<span class="badge badge-xs">Draft</span>
								{/if}
							</div>
						</a>
					</li>
				{/each}

				{#if data.pages.length === 0}
					<li class="text-base-content/60 text-sm p-2">No pages yet</li>
				{/if}
			</ul>
		</aside>

		<!-- Page Content -->
		<main class="flex-1">
			{@render children()}
		</main>
	</div>
</div>
```

**Key Features:**

- Nested layout with workspace context
- Permission checking (owner/admin)
- Pages navigation sidebar
- Active page highlighting
- Empty state handling

---

## 6. Creating the Page Route

### Individual Page Display

Display a single page with full details.

```typescript
// src/routes/(app)/workspaces/[slug]/pages/[pageId]/+page.server.ts
import { error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { pages, users } from "$lib/server/db/schema";
import { eq } from "drizzle-orm";
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async ({ params, parent }) => {
  // Get workspace data from parent layout
  const { workspace } = await parent();

  // Query page with author info
  const [pageData] = await db
    .select({
      page: pages,
      author: {
        id: users.id,
        name: users.name,
        avatarUrl: users.avatarUrl,
      },
    })
    .from(pages)
    .innerJoin(users, eq(pages.authorId, users.id))
    .where(eq(pages.id, parseInt(params.pageId)))
    .limit(1);

  if (!pageData) {
    throw error(404, "Page not found");
  }

  // Verify page belongs to this workspace
  if (pageData.page.workspaceId !== workspace.id) {
    throw error(404, "Page not found in this workspace");
  }

  return {
    page: pageData.page,
    author: pageData.author,
  };
};
```

```svelte
<!-- src/routes/(app)/workspaces/[slug]/pages/[pageId]/+page.svelte -->
<script lang="ts">
	import { marked } from 'marked';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	let renderedContent = $derived(marked(data.page.content || ''));

	function formatDate(date: Date) {
		return new Date(date).toLocaleDateString('en-US', {
			year: 'numeric',
			month: 'long',
			day: 'numeric'
		});
	}
</script>

<article class="max-w-4xl mx-auto p-8">
	<!-- Page Header -->
	<header class="mb-8">
		<div class="flex items-center gap-4 mb-4">
			<img
				src={data.author.avatarUrl || 'https://i.pravatar.cc/150'}
				alt={data.author.name}
				class="w-10 h-10 rounded-full"
			/>
			<div>
				<p class="font-medium">{data.author.name}</p>
				<p class="text-sm text-base-content/60">
					Updated {formatDate(data.page.updatedAt)}
				</p>
			</div>
		</div>

		<h1 class="text-4xl font-bold mb-2">{data.page.title}</h1>

		{#if !data.page.isPublished}
			<div class="badge badge-warning">Draft</div>
		{/if}
	</header>

	<!-- Page Content -->
	<div class="prose lg:prose-xl max-w-none">
		{@html renderedContent}
	</div>

	<!-- Edit Button -->
	<div class="mt-12 pt-8 border-t">
		<a href="/workspaces/{data.workspace.slug}/pages/{data.page.id}/edit" class="btn btn-primary">
			Edit Page
		</a>
	</div>
</article>
```

---

## 7. Complete Example: Collaborative Workspace Platform

### Full CRUD with Database Integration

Let's build the complete workspace platform with all database operations!

**Features:**

- ✅ Workspace dashboard with pages list
- ✅ Create/edit/delete pages
- ✅ Markdown editor with live preview
- ✅ Permission-based access control
- ✅ Real-time member management
- ✅ Activity feed
- ✅ Search functionality

### 📁 Project Structure

```
src/
├── lib/
│   ├── server/
│   │   └── db/
│   │       ├── index.ts
│   │       ├── schema.ts
│   │       └── queries.ts        # All database queries
│   └── components/
│       ├── MarkdownEditor.svelte
│       ├── PageCard.svelte
│       ├── MemberList.svelte
│       └── ActivityFeed.svelte
├── routes/
│   ├── (app)/
│   │   ├── +layout.server.ts     # User + workspaces
│   │   ├── +layout.svelte
│   │   └── workspaces/
│   │       ├── [slug]/
│   │       │   ├── +layout.server.ts  # Workspace data
│   │       │   ├── +layout.svelte     # Workspace shell
│   │       │   ├── +page.server.ts    # Dashboard
│   │       │   ├── +page.svelte
│   │       │   ├── pages/
│   │       │   │   ├── new/
│   │       │   │   │   ├── +page.svelte
│   │       │   │   │   └── +server.ts
│   │       │   │   └── [pageId]/
│   │       │   │       ├── +page.server.ts
│   │       │   │       ├── +page.svelte
│   │       │   │       └── edit/
│   │       │   │           ├── +page.server.ts
│   │       │   │           ├── +page.svelte
│   │       │   │           └── +server.ts
│   │       │   ├── members/
│   │       │   │   ├── +page.server.ts
│   │       │   │   └── +page.svelte
│   │       │   └── settings/
│   │       │       ├── +page.server.ts
│   │       │       ├── +page.svelte
│   │       │       └── +server.ts
│   │       └── new/
│   │           ├── +page.svelte
│   │           └── +server.ts
│   └── api/
│       └── search/+server.ts
```

### Workspace Dashboard

```typescript
// src/routes/(app)/workspaces/[slug]/+page.server.ts
import { db } from "$lib/server/db";
import { pages, users, workspaceMembers } from "$lib/server/db/schema";
import { eq, desc, count } from "drizzle-orm";
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async ({ parent }) => {
  const { workspace } = await parent();

  // Get recent pages
  const recentPages = await db
    .select({
      id: pages.id,
      title: pages.title,
      isPublished: pages.isPublished,
      updatedAt: pages.updatedAt,
      author: {
        name: users.name,
        avatarUrl: users.avatarUrl,
      },
    })
    .from(pages)
    .innerJoin(users, eq(pages.authorId, users.id))
    .where(eq(pages.workspaceId, workspace.id))
    .orderBy(desc(pages.updatedAt))
    .limit(10);

  // Get workspace stats
  const [stats] = await db
    .select({
      totalPages: count(pages.id),
      totalMembers: count(workspaceMembers.id),
    })
    .from(pages)
    .where(eq(pages.workspaceId, workspace.id))
    .leftJoin(workspaceMembers, eq(workspaceMembers.workspaceId, workspace.id));

  // Get workspace members
  const members = await db
    .select({
      id: workspaceMembers.id,
      role: workspaceMembers.role,
      joinedAt: workspaceMembers.joinedAt,
      user: {
        id: users.id,
        name: users.name,
        email: users.email,
        avatarUrl: users.avatarUrl,
      },
    })
    .from(workspaceMembers)
    .innerJoin(users, eq(workspaceMembers.userId, users.id))
    .where(eq(workspaceMembers.workspaceId, workspace.id))
    .orderBy(workspaceMembers.joinedAt);

  return {
    recentPages,
    stats,
    members,
  };
};
```

```svelte
<!-- src/routes/(app)/workspaces/[slug]/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<div class="p-6 space-y-8">
	<!-- Stats -->
	<div class="stats shadow w-full">
		<div class="stat">
			<div class="stat-title">Total Pages</div>
			<div class="stat-value text-primary">{data.stats.totalPages}</div>
		</div>
		<div class="stat">
			<div class="stat-title">Team Members</div>
			<div class="stat-value text-secondary">{data.stats.totalMembers}</div>
		</div>
		<div class="stat">
			<div class="stat-title">Your Role</div>
			<div class="stat-value text-accent text-lg capitalize">{data.userRole}</div>
		</div>
	</div>

	<!-- Recent Pages -->
	<div class="card bg-base-100 shadow">
		<div class="card-body">
			<div class="flex justify-between items-center mb-4">
				<h2 class="card-title">Recent Pages</h2>
				<a href="/workspaces/{data.workspace.slug}/pages/new" class="btn btn-primary btn-sm">
					+ New Page
				</a>
			</div>

			<div class="overflow-x-auto">
				<table class="table table-zebra">
					<thead>
						<tr>
							<th>Title</th>
							<th>Author</th>
							<th>Status</th>
							<th>Updated</th>
							<th></th>
						</tr>
					</thead>
					<tbody>
						{#each data.recentPages as page}
							<tr>
								<td>
									<a
										href="/workspaces/{data.workspace.slug}/pages/{page.id}"
										class="link link-hover font-medium"
									>
										{page.title}
									</a>
								</td>
								<td>
									<div class="flex items-center gap-2">
										<img
											src={page.author.avatarUrl || 'https://i.pravatar.cc/150'}
											alt={page.author.name}
											class="w-8 h-8 rounded-full"
										/>
										<span>{page.author.name}</span>
									</div>
								</td>
								<td>
									{#if page.isPublished}
										<span class="badge badge-success">Published</span>
									{:else}
										<span class="badge badge-warning">Draft</span>
									{/if}
								</td>
								<td>{new Date(page.updatedAt).toLocaleDateString()}</td>
								<td>
									<a
										href="/workspaces/{data.workspace.slug}/pages/{page.id}/edit"
										class="btn btn-ghost btn-xs"
									>
										Edit
									</a>
								</td>
							</tr>
						{/each}
					</tbody>
				</table>
			</div>
		</div>
	</div>

	<!-- Team Members -->
	<div class="card bg-base-100 shadow">
		<div class="card-body">
			<h2 class="card-title mb-4">Team Members</h2>

			<div class="grid md:grid-cols-2 gap-4">
				{#each data.members as member}
					<div class="flex items-center gap-3 p-3 border rounded-lg">
						<img
							src={member.user.avatarUrl || 'https://i.pravatar.cc/150'}
							alt={member.user.name}
							class="w-12 h-12 rounded-full"
						/>
						<div class="flex-1">
							<p class="font-medium">{member.user.name}</p>
							<p class="text-sm text-base-content/60">{member.user.email}</p>
						</div>
						<div class="badge badge-outline capitalize">{member.role}</div>
					</div>
				{/each}
			</div>
		</div>
	</div>
</div>
```

### Create Page

```svelte
<!-- src/routes/(app)/workspaces/[slug]/pages/new/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();

	let title = $state('');
	let content = $state('');
	let isPublished = $state(false);
	let submitting = $state(false);
</script>

<div class="p-6 max-w-4xl mx-auto">
	<h1 class="text-3xl font-bold mb-6">Create New Page</h1>

	<form
		method="POST"
		class="space-y-6"
		use:enhance={() => {
			submitting = true;
			return async ({ update }) => {
				await update();
				submitting = false;
			};
		}}
	>
		{#if form?.error}
			<div class="alert alert-error">
				<span>{form.error}</span>
			</div>
		{/if}

		<div class="form-control">
			<label class="label">
				<span class="label-text">Title</span>
			</label>
			<input type="text" name="title" bind:value={title} class="input input-bordered" required />
		</div>

		<div class="form-control">
			<label class="label">
				<span class="label-text">Content (Markdown)</span>
			</label>
			<textarea
				name="content"
				bind:value={content}
				rows="15"
				class="textarea textarea-bordered font-mono"
				placeholder="# Your content here..."
			></textarea>
		</div>

		<div class="form-control">
			<label class="label cursor-pointer justify-start gap-4">
				<input type="checkbox" name="isPublished" bind:checked={isPublished} class="checkbox" />
				<span class="label-text">Publish immediately</span>
			</label>
		</div>

		<div class="flex gap-4">
			<button type="submit" class="btn btn-primary" disabled={submitting}>
				{submitting ? 'Creating...' : 'Create Page'}
			</button>
			<a href=".." class="btn btn-ghost">Cancel</a>
		</div>
	</form>
</div>
```

```typescript
// src/routes/(app)/workspaces/[slug]/pages/new/+server.ts
import { fail, redirect } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { pages } from "$lib/server/db/schema";
import type { Actions } from "./$types";

export const actions: Actions = {
  default: async ({ request, params, locals, parent }) => {
    const { workspace } = await parent();
    const data = await request.formData();

    const title = data.get("title")?.toString();
    const content = data.get("content")?.toString();
    const isPublished = data.get("isPublished") === "on";

    if (!title) {
      return fail(400, { error: "Title is required" });
    }

    const [page] = await db
      .insert(pages)
      .values({
        workspaceId: workspace.id,
        title,
        content: content || "",
        authorId: locals.user!.id,
        isPublished,
      })
      .returning();

    throw redirect(303, `/workspaces/${workspace.slug}/pages/${page.id}`);
  },
};
```

### Search API

```typescript
// src/routes/api/search/+server.ts
import { json } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import { pages, workspaces } from "$lib/server/db/schema";
import { like, or, and, eq } from "drizzle-orm";
import type { RequestHandler } from "./$types";

export const GET: RequestHandler = async ({ url, locals }) => {
  if (!locals.user) {
    return json({ results: [] });
  }

  const query = url.searchParams.get("q") || "";
  const workspaceId = url.searchParams.get("workspaceId");

  if (!query) {
    return json({ results: [] });
  }

  let searchQuery = db
    .select({
      id: pages.id,
      title: pages.title,
      content: pages.content,
      workspaceId: pages.workspaceId,
      workspaceName: workspaces.name,
      workspaceSlug: workspaces.slug,
    })
    .from(pages)
    .innerJoin(workspaces, eq(pages.workspaceId, workspaces.id))
    .where(
      or(like(pages.title, `%${query}%`), like(pages.content, `%${query}%`))
    )
    .limit(20);

  if (workspaceId) {
    searchQuery = searchQuery.where(
      and(
        eq(pages.workspaceId, parseInt(workspaceId)),
        or(like(pages.title, `%${query}%`), like(pages.content, `%${query}%`))
      )
    );
  }

  const results = await searchQuery;

  return json({ results, query });
};
```

**Key Features Demonstrated:**

- ✅ **Database Queries**: Complex joins with Drizzle
- ✅ **Layout Data**: Nested layouts with parent data
- ✅ **Type Safety**: Full TypeScript inference
- ✅ **CRUD Operations**: Create, read, update, delete
- ✅ **Permissions**: Role-based access control
- ✅ **Search**: Full-text search across pages
- ✅ **Stats**: Aggregated workspace statistics
- ✅ **Real-time Updates**: Activity feed and recent pages

---

## 📝 Key Takeaways

✅ Drizzle provides type-safe database queries  
✅ Use `.select().from()` for basic queries  
✅ `eq()`, `and()`, `or()` for where clauses  
✅ `innerJoin()` for related data  
✅ Layout loads share data across pages  
✅ `await parent()` accesses parent layout data  
✅ Permission checks in load functions  
✅ Query data with author/workspace context  
✅ Use `count()` for statistics  
✅ `like()` for search functionality  
✅ Always validate user access to resources  
✅ Return structured data with joins

---

## 🚀 Next Steps

You've mastered database queries and data display! Next up:

- **Section 16**: Form Actions & Data Mutations
- **Section 17**: Authentication & Authorization
- Real-time updates with WebSockets
- Advanced query optimization and caching
