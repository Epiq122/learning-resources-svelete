# Section 13: Load Function Dependencies & Invalidation

## 📚 Learning Objectives

By the end of this section, you will:

- Understand load function dependencies and re-execution
- Manually invalidate load functions with `invalidate()`
- Handle errors that trigger load re-runs
- Use `preloadData()` and `preloadCode()` for performance
- Configure link preloading behavior
- Build a real-time collaborative task manager

---

## Table of Contents

- [Section 13: Load Function Dependencies \& Invalidation](#section-13-load-function-dependencies--invalidation)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Adding Layouts to Child Routes](#1-adding-layouts-to-child-routes)
    - [Nested Layout Patterns](#nested-layout-patterns)
  - [2. Load Function Dependencies](#2-load-function-dependencies)
    - [What Triggers Load Re-runs?](#what-triggers-load-re-runs)
  - [3. Invalidating Load Functions](#3-invalidating-load-functions)
    - [Manual Cache Busting](#manual-cache-busting)
  - [4. Re-running Load Functions on Error](#4-re-running-load-functions-on-error)
    - [Retry Failed Requests](#retry-failed-requests)
  - [5. Link Preloading Options](#5-link-preloading-options)
    - [Preloading Code \& Data](#preloading-code--data)
    - [Default Behavior](#default-behavior)
    - [Preloading Strategies](#preloading-strategies)
    - [Programmatic Preloading](#programmatic-preloading)
  - [6. Complete Example: Real-Time Task Manager](#6-complete-example-real-time-task-manager)
    - [Collaborative Tasks with Live Updates](#collaborative-tasks-with-live-updates)
    - [📁 Project Structure](#-project-structure)
    - [Database](#database)
    - [Task List Load Function](#task-list-load-function)
    - [Task List Page](#task-list-page)
    - [Task Detail Layout](#task-detail-layout)
    - [API Endpoints](#api-endpoints)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Adding Layouts to Child Routes

### Nested Layout Patterns

You can add layouts at any level of your routing hierarchy.

**Real-World Scenario:** Blog post pages need a different layout than the blog list.

```
src/routes/
├── blog/
│   ├── +page.svelte           # Blog list
│   └── [slug]/
│       ├── +layout.svelte     # Layout for single post
│       └── +page.svelte       # Post content
```

```svelte
<!-- src/routes/blog/[slug]/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();
</script>

<div class="container mx-auto px-4 py-8">
	<!-- Breadcrumbs -->
	<div class="breadcrumbs text-sm mb-6">
		<ul>
			<li><a href="/">Home</a></li>
			<li><a href="/blog">Blog</a></li>
			<li>{data.post.title}</li>
		</ul>
	</div>

	<!-- Post metadata -->
	<div class="mb-8">
		<h1 class="text-4xl font-bold mb-2">{data.post.title}</h1>
		<div class="flex gap-4 text-sm text-base-content/60">
			<span>By {data.post.author}</span>
			<span>{data.post.publishedAt}</span>
			<span>{data.post.readTime} min read</span>
		</div>
	</div>

	<!-- Post content -->
	{@render children()}

	<!-- Related posts -->
	<div class="mt-12 border-t pt-8">
		<h3 class="text-2xl font-bold mb-4">Related Posts</h3>
		<div class="grid md:grid-cols-3 gap-6">
			{#each data.relatedPosts as post}
				<a href="/blog/{post.slug}" class="card bg-base-100 shadow hover:shadow-xl">
					<div class="card-body">
						<h4 class="card-title">{post.title}</h4>
					</div>
				</a>
			{/each}
		</div>
	</div>
</div>
```

```typescript
// src/routes/blog/[slug]/+layout.server.ts
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async ({ params }) => {
	const [post, relatedPosts] = await Promise.all([
		db.posts.findUnique({ where: { slug: params.slug } }),
		db.posts.findRelated(params.slug, 3)
	]);

	return { post, relatedPosts };
};
```

> 💡 **Best Practice**: Use nested layouts to share data and UI between related pages.

---

## 2. Load Function Dependencies

### What Triggers Load Re-runs?

Load functions automatically re-run when their **dependencies** change.

**Dependencies:**

- URL parameters (`params`)
- URL search params (`url.searchParams`)
- Data fetched with `fetch()` or `depends()`

**Real-World Scenario:** Search results update when query changes.

```typescript
// src/routes/search/+page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ url, fetch }) => {
	const query = url.searchParams.get('q') || '';
	const category = url.searchParams.get('category') || 'all';

	// This load function depends on 'q' and 'category' params
	// It will re-run whenever they change!
	const response = await fetch(`/api/search?q=${query}&category=${category}`);
	const results = await response.json();

	return { results, query, category };
};
```

```svelte
<!-- src/routes/search/+page.svelte -->
<script lang="ts">
	import { goto } from '$app/navigation';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
	let searchQuery = $state(data.query);

	function handleSearch() {
		goto(`/search?q=${searchQuery}`);
		// Load function automatically re-runs!
	}
</script>

<form
	onsubmit={(e) => {
		e.preventDefault();
		handleSearch();
	}}
>
	<input type="search" bind:value={searchQuery} class="input input-bordered" />
	<button type="submit" class="btn btn-primary">Search</button>
</form>

<div class="mt-6">
	<h2 class="text-2xl font-bold mb-4">Results for "{data.query}"</h2>
	{#each data.results as result}
		<div class="card bg-base-100 shadow mb-4">
			<div class="card-body">
				<h3 class="card-title">{result.title}</h3>
				<p>{result.description}</p>
			</div>
		</div>
	{/each}
</div>
```

**Automatic dependencies:**

- ✅ URL pathname changes
- ✅ URL search params change
- ✅ `params` object changes
- ✅ Fetched URLs change

---

## 3. Invalidating Load Functions

### Manual Cache Busting

Use `invalidate()` to manually re-run load functions.

**Real-World Scenario:** Refresh data after a mutation without full page reload.

```typescript
// src/routes/tasks/+page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch, depends }) => {
	// Register custom dependency
	depends('tasks:list');

	const response = await fetch('/api/tasks');
	const tasks = await response.json();

	return { tasks };
};
```

```svelte
<!-- src/routes/tasks/+page.svelte -->
<script lang="ts">
	import { invalidate } from '$app/navigation';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	async function createTask(title: string) {
		await fetch('/api/tasks', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({ title })
		});

		// Manually invalidate to refresh tasks!
		await invalidate('tasks:list');
	}

	async function refreshAll() {
		// Invalidate all load functions on this page
		await invalidate(() => true);
	}
</script>

<button onclick={() => createTask('New Task')} class="btn btn-primary"> Add Task </button>

<button onclick={refreshAll} class="btn btn-secondary"> Refresh All </button>

<div class="mt-6 space-y-2">
	{#each data.tasks as task}
		<div class="card bg-base-100 shadow">
			<div class="card-body">
				<h3>{task.title}</h3>
			</div>
		</div>
	{/each}
</div>
```

**invalidate() strategies:**

```typescript
// Invalidate specific dependency
await invalidate('tasks:list');

// Invalidate specific URL
await invalidate('/api/tasks');

// Invalidate multiple
await invalidate(['tasks:list', 'user:profile']);

// Invalidate all
await invalidate(() => true);

// Invalidate by pattern
await invalidate((url) => url.pathname.startsWith('/api/'));
```

> 💡 **Best Practice**: Use custom dependencies with `depends()` for semantic invalidation.

---

## 4. Re-running Load Functions on Error

### Retry Failed Requests

When a load function throws an error, you can retry it.

**Real-World Scenario:** Network failed - let user retry.

```typescript
// src/routes/posts/+page.ts
import { error } from '@sveltejs/kit';
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ fetch }) => {
	try {
		const response = await fetch('/api/posts');

		if (!response.ok) {
			throw error(response.status, 'Failed to load posts');
		}

		const posts = await response.json();
		return { posts };
	} catch (err) {
		throw error(500, 'Network error - please try again');
	}
};
```

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
	import { invalidate } from '$app/navigation';

	async function retry() {
		// Re-run all load functions on the current page!
		await invalidate(() => true);
	}
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-6xl font-bold text-error">{$page.status}</h1>
			<p class="py-6">{$page.error?.message}</p>

			{#if $page.status >= 500}
				<button onclick={retry} class="btn btn-primary"> Retry </button>
			{:else}
				<a href="/" class="btn btn-primary">Go Home</a>
			{/if}
		</div>
	</div>
</div>
```

**Key Concepts:**

- Errors throw and show error page
- `invalidate(() => true)` re-runs load functions
- Users can retry without losing their place

---

## 5. Link Preloading Options

### Preloading Code & Data

SvelteKit can preload routes before navigation for instant page loads.

**Real-World Scenario:** Preload next page in a multi-step form.

### Default Behavior

```svelte
<!-- Default: hover preloads -->
<a href="/dashboard">Dashboard</a>
```

### Preloading Strategies

```svelte
<!-- 1. No preloading -->
<a href="/large-page" data-sveltekit-preload-data="off"> Heavy Page </a>

<!-- 2. Preload on hover (default) -->
<a href="/dashboard" data-sveltekit-preload-data="hover"> Dashboard </a>

<!-- 3. Preload on tap (mobile-friendly) -->
<a href="/settings" data-sveltekit-preload-data="tap"> Settings </a>

<!-- 4. Eager preload (as soon as link is visible) -->
<a href="/next-step" data-sveltekit-preload-data="viewport"> Continue </a>

<!-- 5. Preload code only (not data) -->
<a href="/blog" data-sveltekit-preload-code="hover"> Blog </a>

<!-- 6. Preload both code and data -->
<a href="/profile" data-sveltekit-preload-data="hover" data-sveltekit-preload-code="eager">
	Profile
</a>
```

### Programmatic Preloading

```svelte
<script lang="ts">
	import { preloadData, preloadCode } from '$app/navigation';
	import { onMount } from 'svelte';

	onMount(() => {
		// Preload next step in background
		preloadData('/checkout/step-2');

		// Preload multiple routes
		preloadCode('/dashboard');
		preloadCode('/settings');
	});
</script>
```

**Preloading options:**

| Option     | When to Use                           |
| ---------- | ------------------------------------- |
| `off`      | Heavy pages, authenticated routes     |
| `hover`    | Default, good for desktop             |
| `tap`      | Mobile-optimized                      |
| `viewport` | Critical next steps (forms, checkout) |
| Code only  | When data is user-specific            |

> ⚡ **Performance Tip**: Preload critical paths in multi-step flows for instant navigation.

---

## 6. Complete Example: Real-Time Task Manager

### Collaborative Tasks with Live Updates

Let's build a collaborative task manager with real-time updates, optimistic UI, and smart invalidation!

**Features:**

- ✅ Real-time updates with polling and manual refresh
- ✅ Optimistic UI for instant feedback
- ✅ Task filtering with URL params (auto-invalidation)
- ✅ Smart preloading for task details
- ✅ Error handling with retry
- ✅ Nested layouts for task details
- ✅ Custom dependencies for granular invalidation

### 📁 Project Structure

```
src/
├── lib/
│   ├── server/
│   │   └── db.ts                   # Task database
│   └── components/
│       ├── TaskList.svelte
│       ├── TaskItem.svelte
│       ├── TaskForm.svelte
│       └── FilterBar.svelte
├── routes/
│   ├── tasks/
│   │   ├── +page.ts               # Task list with filters
│   │   ├── +page.svelte
│   │   └── [id]/
│   │       ├── +layout.ts         # Task details layout
│   │       ├── +layout.svelte
│   │       ├── +page.svelte       # Task view
│   │       └── edit/
│   │           └── +page.svelte   # Task edit
│   └── api/
│       └── tasks/
│           ├── +server.ts         # List & create
│           └── [id]/
│               └── +server.ts     # Update & delete
└── hooks.server.ts
```

### Database

```typescript
// src/lib/server/db.ts
export interface Task {
	id: string;
	title: string;
	description: string;
	status: 'todo' | 'in-progress' | 'done';
	priority: 'low' | 'medium' | 'high';
	assignee: string;
	createdAt: string;
	updatedAt: string;
}

let tasks: Task[] = [
	{
		id: '1',
		title: 'Design landing page',
		description: 'Create mockups for new landing page',
		status: 'in-progress',
		priority: 'high',
		assignee: 'Alice',
		createdAt: '2024-01-01T10:00:00Z',
		updatedAt: '2024-01-01T10:00:00Z'
	},
	{
		id: '2',
		title: 'Write API docs',
		description: 'Document all API endpoints',
		status: 'todo',
		priority: 'medium',
		assignee: 'Bob',
		createdAt: '2024-01-02T10:00:00Z',
		updatedAt: '2024-01-02T10:00:00Z'
	}
];

export const db = {
	tasks: {
		findAll: async (filters?: { status?: string; priority?: string; search?: string }) => {
			let filtered = tasks;

			if (filters?.status) {
				filtered = filtered.filter((t) => t.status === filters.status);
			}
			if (filters?.priority) {
				filtered = filtered.filter((t) => t.priority === filters.priority);
			}
			if (filters?.search) {
				const search = filters.search.toLowerCase();
				filtered = filtered.filter(
					(t) =>
						t.title.toLowerCase().includes(search) || t.description.toLowerCase().includes(search)
				);
			}

			return filtered;
		},

		findById: async (id: string) => {
			return tasks.find((t) => t.id === id);
		},

		create: async (data: Omit<Task, 'id' | 'createdAt' | 'updatedAt'>) => {
			const task: Task = {
				...data,
				id: crypto.randomUUID(),
				createdAt: new Date().toISOString(),
				updatedAt: new Date().toISOString()
			};
			tasks.push(task);
			return task;
		},

		update: async (id: string, data: Partial<Task>) => {
			const index = tasks.findIndex((t) => t.id === id);
			if (index === -1) return null;

			tasks[index] = {
				...tasks[index],
				...data,
				updatedAt: new Date().toISOString()
			};
			return tasks[index];
		},

		delete: async (id: string) => {
			const index = tasks.findIndex((t) => t.id === id);
			if (index === -1) return false;
			tasks.splice(index, 1);
			return true;
		}
	}
};
```

### Task List Load Function

```typescript
// src/routes/tasks/+page.ts
import type { PageLoad } from './$types';

export const load: PageLoad = async ({ url, fetch, depends }) => {
	// Register custom dependencies for granular invalidation
	depends('tasks:list');
	depends('tasks:filters');

	// Extract filters from URL (automatic dependency!)
	const status = url.searchParams.get('status') || undefined;
	const priority = url.searchParams.get('priority') || undefined;
	const search = url.searchParams.get('search') || undefined;

	// Build query string
	const params = new URLSearchParams();
	if (status) params.set('status', status);
	if (priority) params.set('priority', priority);
	if (search) params.set('search', search);

	// Fetch tasks (URL dependency!)
	const response = await fetch(`/api/tasks?${params}`);
	const tasks = await response.json();

	return {
		tasks,
		filters: { status, priority, search }
	};
};
```

### Task List Page

```svelte
<!-- src/routes/tasks/+page.svelte -->
<script lang="ts">
	import { goto, invalidate, preloadData } from '$app/navigation';
	import { page } from '$app/stores';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	// Filters
	let status = $state(data.filters.status || '');
	let priority = $state(data.filters.priority || '');
	let search = $state(data.filters.search || '');

	// Optimistic tasks
	let optimisticTasks = $state(data.tasks);
	$effect(() => {
		optimisticTasks = data.tasks;
	});

	// Auto-refresh every 30 seconds
	let refreshInterval: number;
	$effect(() => {
		refreshInterval = setInterval(() => {
			invalidate('tasks:list');
		}, 30000);

		return () => clearInterval(refreshInterval);
	});

	function applyFilters() {
		const params = new URLSearchParams();
		if (status) params.set('status', status);
		if (priority) params.set('priority', priority);
		if (search) params.set('search', search);

		// URL change triggers automatic re-run!
		goto(`/tasks?${params}`, { keepFocus: true });
	}

	async function createTask(title: string) {
		// Optimistic update
		const optimisticTask = {
			id: 'temp-' + Date.now(),
			title,
			description: '',
			status: 'todo' as const,
			priority: 'medium' as const,
			assignee: 'You',
			createdAt: new Date().toISOString(),
			updatedAt: new Date().toISOString()
		};
		optimisticTasks = [optimisticTask, ...optimisticTasks];

		try {
			await fetch('/api/tasks', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({
					title,
					status: 'todo',
					priority: 'medium',
					assignee: 'You',
					description: ''
				})
			});

			// Refresh from server
			await invalidate('tasks:list');
		} catch (err) {
			// Rollback on error
			optimisticTasks = data.tasks;
			alert('Failed to create task');
		}
	}

	async function deleteTask(id: string) {
		// Optimistic delete
		optimisticTasks = optimisticTasks.filter((t) => t.id !== id);

		try {
			await fetch(`/api/tasks/${id}`, { method: 'DELETE' });
			await invalidate('tasks:list');
		} catch (err) {
			optimisticTasks = data.tasks;
			alert('Failed to delete task');
		}
	}

	async function refreshNow() {
		await invalidate('tasks:list');
	}

	// Preload task details on hover
	function handleTaskHover(id: string) {
		preloadData(`/tasks/${id}`);
	}
</script>

<div class="container mx-auto px-4 py-8">
	<div class="flex justify-between items-center mb-8">
		<h1 class="text-4xl font-bold">Tasks</h1>
		<button onclick={refreshNow} class="btn btn-circle btn-ghost" title="Refresh"> 🔄 </button>
	</div>

	<!-- Filters -->
	<div class="card bg-base-100 shadow mb-6">
		<div class="card-body">
			<div class="grid md:grid-cols-3 gap-4">
				<div class="form-control">
					<label class="label">
						<span class="label-text">Status</span>
					</label>
					<select bind:value={status} onchange={applyFilters} class="select select-bordered">
						<option value="">All</option>
						<option value="todo">To Do</option>
						<option value="in-progress">In Progress</option>
						<option value="done">Done</option>
					</select>
				</div>

				<div class="form-control">
					<label class="label">
						<span class="label-text">Priority</span>
					</label>
					<select bind:value={priority} onchange={applyFilters} class="select select-bordered">
						<option value="">All</option>
						<option value="low">Low</option>
						<option value="medium">Medium</option>
						<option value="high">High</option>
					</select>
				</div>

				<div class="form-control">
					<label class="label">
						<span class="label-text">Search</span>
					</label>
					<input
						type="search"
						bind:value={search}
						onkeyup={(e) => e.key === 'Enter' && applyFilters()}
						placeholder="Search tasks..."
						class="input input-bordered"
					/>
				</div>
			</div>
		</div>
	</div>

	<!-- Quick add -->
	<form
		onsubmit={(e) => {
			e.preventDefault();
			const input = e.target.querySelector('input');
			if (input.value) {
				createTask(input.value);
				input.value = '';
			}
		}}
		class="mb-6"
	>
		<div class="join w-full">
			<input
				type="text"
				placeholder="Add a task..."
				class="input input-bordered join-item flex-1"
			/>
			<button type="submit" class="btn btn-primary join-item">Add</button>
		</div>
	</form>

	<!-- Task list -->
	<div class="space-y-3">
		{#if optimisticTasks.length === 0}
			<div class="alert">
				<span>No tasks found. Try adjusting your filters.</span>
			</div>
		{/if}

		{#each optimisticTasks as task (task.id)}
			<div
				class="card bg-base-100 shadow hover:shadow-lg transition-shadow"
				onmouseenter={() => handleTaskHover(task.id)}
			>
				<div class="card-body">
					<div class="flex justify-between items-start">
						<div class="flex-1">
							<a href="/tasks/{task.id}" class="card-title hover:text-primary">
								{task.title}
							</a>
							<p class="text-base-content/60 text-sm mt-1">
								{task.description || 'No description'}
							</p>
							<div class="flex gap-2 mt-3">
								<div class="badge badge-outline">{task.status}</div>
								<div
									class="badge badge-{task.priority === 'high'
										? 'error'
										: task.priority === 'medium'
											? 'warning'
											: 'info'}"
								>
									{task.priority}
								</div>
								<div class="badge badge-ghost">{task.assignee}</div>
							</div>
						</div>
						<div class="dropdown dropdown-end">
							<label tabindex="0" class="btn btn-ghost btn-sm">⋮</label>
							<ul
								tabindex="0"
								class="dropdown-content menu p-2 shadow bg-base-100 rounded-box w-52"
							>
								<li><a href="/tasks/{task.id}/edit">Edit</a></li>
								<li>
									<button onclick={() => deleteTask(task.id)} class="text-error">Delete</button>
								</li>
							</ul>
						</div>
					</div>
				</div>
			</div>
		{/each}
	</div>
</div>
```

### Task Detail Layout

```typescript
// src/routes/tasks/[id]/+layout.ts
import { error } from '@sveltejs/kit';
import type { LayoutLoad } from './$types';

export const load: LayoutLoad = async ({ params, fetch, depends }) => {
	depends('task:detail');

	const response = await fetch(`/api/tasks/${params.id}`);

	if (!response.ok) {
		throw error(404, 'Task not found');
	}

	const task = await response.json();
	return { task };
};
```

```svelte
<!-- src/routes/tasks/[id]/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import { invalidate } from '$app/navigation';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();

	async function refresh() {
		await invalidate('task:detail');
	}
</script>

<div class="container mx-auto px-4 py-8">
	<div class="breadcrumbs text-sm mb-6">
		<ul>
			<li><a href="/">Home</a></li>
			<li><a href="/tasks">Tasks</a></li>
			<li>{data.task.title}</li>
		</ul>
	</div>

	<div class="flex justify-between items-center mb-6">
		<h1 class="text-3xl font-bold">{data.task.title}</h1>
		<button onclick={refresh} class="btn btn-circle btn-ghost">🔄</button>
	</div>

	{@render children()}
</div>
```

### API Endpoints

```typescript
// src/routes/api/tasks/+server.ts
import { json } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ url }) => {
	const status = url.searchParams.get('status') || undefined;
	const priority = url.searchParams.get('priority') || undefined;
	const search = url.searchParams.get('search') || undefined;

	const tasks = await db.tasks.findAll({ status, priority, search });
	return json(tasks);
};

export const POST: RequestHandler = async ({ request }) => {
	const data = await request.json();
	const task = await db.tasks.create(data);
	return json(task, { status: 201 });
};
```

```typescript
// src/routes/api/tasks/[id]/+server.ts
import { json, error } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ params }) => {
	const task = await db.tasks.findById(params.id);
	if (!task) throw error(404, 'Task not found');
	return json(task);
};

export const PUT: RequestHandler = async ({ params, request }) => {
	const data = await request.json();
	const task = await db.tasks.update(params.id, data);
	if (!task) throw error(404, 'Task not found');
	return json(task);
};

export const DELETE: RequestHandler = async ({ params }) => {
	const deleted = await db.tasks.delete(params.id);
	if (!deleted) throw error(404, 'Task not found');
	return json({ success: true });
};
```

**Key Features Demonstrated:**

- ✅ **Automatic Dependencies**: URL params trigger re-runs
- ✅ **Custom Dependencies**: `depends()` for semantic invalidation
- ✅ **Manual Invalidation**: `invalidate()` for refresh
- ✅ **Optimistic UI**: Instant feedback, rollback on error
- ✅ **Real-Time Updates**: Polling with `setInterval`
- ✅ **Smart Preloading**: `preloadData()` on hover
- ✅ **Nested Layouts**: Shared task detail UI
- ✅ **Error Recovery**: Retry with `invalidate(() => true)`
- ✅ **Filter Persistence**: URL params maintain state

---

## 📝 Key Takeaways

✅ Nested layouts share UI and data between related pages  
✅ Load functions re-run when dependencies change  
✅ Dependencies: params, search params, fetched URLs  
✅ `depends()` registers custom dependencies  
✅ `invalidate()` manually triggers re-runs  
✅ `invalidate(() => true)` re-runs all loads  
✅ Error pages can retry with invalidation  
✅ `preloadData()` and `preloadCode()` improve performance  
✅ Link preloading: `off`, `hover`, `tap`, `viewport`  
✅ Use `keepFocus: true` to maintain scroll position  
✅ Optimistic UI with rollback on error

---

## 🚀 Next Steps

You've mastered load function dependencies and invalidation! Next up:

- **Section 14**: Form Actions & Progressive Enhancement
- **Section 15**: Advanced Data Patterns (Subscriptions, WebSockets)
- Building real-time collaborative applications
