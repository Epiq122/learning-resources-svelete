# Section 16: Form Actions & Data Mutations

## 📚 Learning Objectives

By the end of this section, you will:

- Create and handle form actions in SvelteKit
- Validate form data on the server
- Return success and error messages
- Insert, update, and delete database records
- Progressively enhance forms with `use:enhance`
- Customize form submission behavior
- Use SuperForms and Formsnap for advanced forms
- Implement shallow routing with modals
- Build a complete workspace management system

---

## Table of Contents

- [Section 16: Form Actions \& Data Mutations](#section-16-form-actions--data-mutations)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction to Form Actions](#1-introduction-to-form-actions)
    - [Server-Side Form Handling](#server-side-form-handling)
  - [2. Form Validation, Success \& Error Messages](#2-form-validation-success--error-messages)
    - [Handling Errors Gracefully](#handling-errors-gracefully)
  - [3. Inserting Data Into the Database](#3-inserting-data-into-the-database)
    - [Creating Workspace Records](#creating-workspace-records)
  - [4. Progressive Enhancement with use:enhance](#4-progressive-enhancement-with-useenhance)
    - [Making Forms Work Without JavaScript](#making-forms-work-without-javascript)
  - [5. Customizing use:enhance Behavior](#5-customizing-useenhance-behavior)
    - [Control Form Submission](#control-form-submission)
  - [6. SuperForms \& Formsnap](#6-superforms--formsnap)
    - [Advanced Form Management](#advanced-form-management)
  - [7. Refactoring with SuperForms](#7-refactoring-with-superforms)
    - [Clean Form Code](#clean-form-code)
  - [8. Editing \& Deleting Resources](#8-editing--deleting-resources)
    - [Update and Delete Actions](#update-and-delete-actions)
  - [9. Shallow Routing with Modals](#9-shallow-routing-with-modals)
    - [Modal Routes Without Full Navigation](#modal-routes-without-full-navigation)
  - [10. Complete Example: Workspace Management Platform](#10-complete-example-workspace-management-platform)
    - [Full CRUD with Forms \& Modals](#full-crud-with-forms--modals)
    - [📁 Project Structure](#-project-structure)
    - [Toast Store](#toast-store)
    - [Toast Component](#toast-component)
    - [Workspaces List Page](#workspaces-list-page)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Introduction to Form Actions

### Server-Side Form Handling

Form actions run on the server and handle form submissions.

**Real-World Scenario:** Create a contact form with server-side validation.

```typescript
// src/routes/contact/+page.server.ts
import type { Actions } from './$types';

export const actions: Actions = {
	// Named action: "default"
	default: async ({ request }) => {
		const data = await request.formData();
		const name = data.get('name')?.toString();
		const email = data.get('email')?.toString();
		const message = data.get('message')?.toString();

		console.log({ name, email, message });

		return { success: true };
	}
};
```

```svelte
<!-- src/routes/contact/+page.svelte -->
<script lang="ts">
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
</script>

<div class="container mx-auto max-w-2xl p-6">
	<h1 class="text-3xl font-bold mb-6">Contact Us</h1>

	<form method="POST" class="space-y-4">
		<div class="form-control">
			<label class="label">
				<span class="label-text">Name</span>
			</label>
			<input type="text" name="name" class="input input-bordered" required />
		</div>

		<div class="form-control">
			<label class="label">
				<span class="label-text">Email</span>
			</label>
			<input type="email" name="email" class="input input-bordered" required />
		</div>

		<div class="form-control">
			<label class="label">
				<span class="label-text">Message</span>
			</label>
			<textarea name="message" rows="5" class="textarea textarea-bordered" required></textarea>
		</div>

		<button type="submit" class="btn btn-primary"> Send Message </button>

		{#if form?.success}
			<div class="alert alert-success">
				<span>Message sent successfully!</span>
			</div>
		{/if}
	</form>
</div>
```

**Multiple Named Actions:**

```typescript
// src/routes/tasks/+page.server.ts
import type { Actions } from './$types';

export const actions: Actions = {
	create: async ({ request }) => {
		const data = await request.formData();
		const title = data.get('title')?.toString();
		// Create task
		return { success: true, action: 'create' };
	},

	delete: async ({ request }) => {
		const data = await request.formData();
		const id = data.get('id')?.toString();
		// Delete task
		return { success: true, action: 'delete' };
	}
};
```

```svelte
<!-- Use ?/actionName to target specific action -->
<form method="POST" action="?/create">
	<input name="title" />
	<button>Create</button>
</form>

<form method="POST" action="?/delete">
	<input name="id" value={task.id} hidden />
	<button>Delete</button>
</form>
```

> 💡 **Best Practice**: Use named actions for different operations on the same page.

---

## 2. Form Validation, Success & Error Messages

### Handling Errors Gracefully

Validate input and return helpful error messages.

```typescript
// src/routes/register/+page.server.ts
import { fail } from '@sveltejs/kit';
import type { Actions } from './$types';

export const actions: Actions = {
	default: async ({ request }) => {
		const data = await request.formData();
		const username = data.get('username')?.toString();
		const email = data.get('email')?.toString();
		const password = data.get('password')?.toString();

		// Validation
		const errors: Record<string, string> = {};

		if (!username || username.length < 3) {
			errors.username = 'Username must be at least 3 characters';
		}

		if (!email || !email.includes('@')) {
			errors.email = 'Please enter a valid email';
		}

		if (!password || password.length < 8) {
			errors.password = 'Password must be at least 8 characters';
		}

		if (Object.keys(errors).length > 0) {
			return fail(400, {
				errors,
				values: { username, email } // Return values to repopulate form
			});
		}

		// Success
		return {
			success: true,
			message: 'Registration successful!'
		};
	}
};
```

```svelte
<!-- src/routes/register/+page.svelte -->
<script lang="ts">
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
</script>

<div class="container mx-auto max-w-md p-6">
	<h1 class="text-3xl font-bold mb-6">Register</h1>

	{#if form?.success}
		<div class="alert alert-success mb-4">
			<span>{form.message}</span>
		</div>
	{/if}

	<form method="POST" class="space-y-4">
		<div class="form-control">
			<label class="label">
				<span class="label-text">Username</span>
			</label>
			<input
				type="text"
				name="username"
				value={form?.values?.username || ''}
				class="input input-bordered"
				class:input-error={form?.errors?.username}
			/>
			{#if form?.errors?.username}
				<label class="label">
					<span class="label-text-alt text-error">{form.errors.username}</span>
				</label>
			{/if}
		</div>

		<div class="form-control">
			<label class="label">
				<span class="label-text">Email</span>
			</label>
			<input
				type="email"
				name="email"
				value={form?.values?.email || ''}
				class="input input-bordered"
				class:input-error={form?.errors?.email}
			/>
			{#if form?.errors?.email}
				<label class="label">
					<span class="label-text-alt text-error">{form.errors.email}</span>
				</label>
			{/if}
		</div>

		<div class="form-control">
			<label class="label">
				<span class="label-text">Password</span>
			</label>
			<input
				type="password"
				name="password"
				class="input input-bordered"
				class:input-error={form?.errors?.password}
			/>
			{#if form?.errors?.password}
				<label class="label">
					<span class="label-text-alt text-error">{form.errors.password}</span>
				</label>
			{/if}
		</div>

		<button type="submit" class="btn btn-primary btn-block"> Register </button>
	</form>
</div>
```

> ⚠️ **Common Mistake**: Not preserving form values on validation errors.

---

## 3. Inserting Data Into the Database

### Creating Workspace Records

Insert validated data into PostgreSQL.

```typescript
// src/routes/workspaces/new/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import { workspaces } from '$lib/server/db/schema';
import type { Actions } from './$types';

function generateSlug(name: string): string {
	return name
		.toLowerCase()
		.replace(/[^a-z0-9]+/g, '-')
		.replace(/^-+|-+$/g, '');
}

export const actions: Actions = {
	default: async ({ request, locals }) => {
		if (!locals.user) {
			return fail(401, { error: 'You must be logged in' });
		}

		const data = await request.formData();
		const name = data.get('name')?.toString();
		const description = data.get('description')?.toString();

		// Validation
		if (!name || name.length < 3) {
			return fail(400, {
				error: 'Workspace name must be at least 3 characters',
				values: { name, description }
			});
		}

		const slug = generateSlug(name);

		// Check if slug exists
		const existing = await db.select().from(workspaces).where(eq(workspaces.slug, slug)).limit(1);

		if (existing.length > 0) {
			return fail(400, {
				error: 'A workspace with this name already exists',
				values: { name, description }
			});
		}

		// Insert into database
		const [workspace] = await db
			.insert(workspaces)
			.values({
				name,
				slug,
				description: description || null,
				ownerId: locals.user.id
			})
			.returning();

		// Redirect to new workspace
		throw redirect(303, `/workspaces/${workspace.slug}`);
	}
};
```

```svelte
<!-- src/routes/workspaces/new/+page.svelte -->
<script lang="ts">
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
</script>

<div class="container mx-auto max-w-2xl p-6">
	<h1 class="text-3xl font-bold mb-6">Create Workspace</h1>

	{#if form?.error}
		<div class="alert alert-error mb-4">
			<span>{form.error}</span>
		</div>
	{/if}

	<form method="POST" class="space-y-4">
		<div class="form-control">
			<label class="label">
				<span class="label-text">Workspace Name</span>
			</label>
			<input
				type="text"
				name="name"
				value={form?.values?.name || ''}
				class="input input-bordered"
				placeholder="My Awesome Workspace"
				required
			/>
			<label class="label">
				<span class="label-text-alt">This will be used to generate the URL</span>
			</label>
		</div>

		<div class="form-control">
			<label class="label">
				<span class="label-text">Description</span>
			</label>
			<textarea
				name="description"
				value={form?.values?.description || ''}
				class="textarea textarea-bordered"
				rows="4"
				placeholder="What's this workspace about?"
			></textarea>
		</div>

		<div class="flex gap-4">
			<button type="submit" class="btn btn-primary"> Create Workspace </button>
			<a href="/workspaces" class="btn btn-ghost"> Cancel </a>
		</div>
	</form>
</div>
```

---

## 4. Progressive Enhancement with use:enhance

### Making Forms Work Without JavaScript

Forms work without JS, but `use:enhance` improves the experience.

```svelte
<script lang="ts">
	import { enhance } from '$app/forms';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
	let loading = $state(false);
</script>

<form
	method="POST"
	use:enhance={() => {
		loading = true;

		return async ({ update }) => {
			await update();
			loading = false;
		};
	}}
>
	<input name="title" class="input input-bordered" required />

	<button type="submit" class="btn btn-primary" disabled={loading}>
		{loading ? 'Creating...' : 'Create'}
	</button>
</form>
```

**What `use:enhance` provides:**

- ✅ No full page reload
- ✅ Updates `$page.form` automatically
- ✅ Resets form on success
- ✅ Maintains scroll position
- ✅ Still works without JavaScript

> 💡 **Best Practice**: Always use `use:enhance` for better UX, but ensure forms work without it.

---

## 5. Customizing use:enhance Behavior

### Control Form Submission

Customize behavior with lifecycle callbacks.

```svelte
<script lang="ts">
	import { enhance } from '$app/forms';
	import { goto } from '$app/navigation';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
	let submitting = $state(false);
	let showSuccess = $state(false);
</script>

<form
	method="POST"
	use:enhance={({ formData, cancel }) => {
		submitting = true;

		// Modify form data before sending
		formData.append('timestamp', new Date().toISOString());

		// Cancel submission conditionally
		if (someCondition) {
			cancel();
			return;
		}

		return async ({ result, update }) => {
			submitting = false;

			if (result.type === 'success') {
				showSuccess = true;

				// Custom navigation instead of default
				setTimeout(() => {
					goto('/workspaces');
				}, 1500);

				// Don't call update() to prevent default behavior
			} else if (result.type === 'failure') {
				// Call update to show errors
				await update();
			} else {
				// Default behavior
				await update();
			}
		};
	}}
>
	<!-- Form fields -->

	{#if showSuccess}
		<div class="alert alert-success">
			<span>Workspace created! Redirecting...</span>
		</div>
	{/if}

	<button type="submit" class="btn btn-primary" disabled={submitting}>
		{submitting ? 'Saving...' : 'Save'}
	</button>
</form>
```

**`use:enhance` callbacks:**

- `formData`: Modify data before submission
- `cancel()`: Prevent submission
- `result`: Server response
- `update()`: Apply default SvelteKit behavior

---

## 6. SuperForms & Formsnap

### Advanced Form Management

SuperForms provides powerful form validation with Zod schemas.

**Installation:**

```bash
npm install sveltekit-superforms zod formsnap
npm install -D @tailwindcss/forms
```

**Define schema:**

```typescript
// src/lib/schemas/workspace.ts
import { z } from 'zod';

export const workspaceSchema = z.object({
	name: z.string().min(3, 'Name must be at least 3 characters'),
	description: z.string().max(500, 'Description too long').optional(),
	isPublic: z.boolean().default(false)
});

export type WorkspaceSchema = z.infer<typeof workspaceSchema>;
```

**Server action:**

```typescript
// src/routes/workspaces/new/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import { superValidate } from 'sveltekit-superforms';
import { zod } from 'sveltekit-superforms/adapters';
import { workspaceSchema } from '$lib/schemas/workspace';
import { db } from '$lib/server/db';
import { workspaces } from '$lib/server/db/schema';
import type { Actions, PageServerLoad } from './$types';

export const load: PageServerLoad = async () => {
	const form = await superValidate(zod(workspaceSchema));
	return { form };
};

export const actions: Actions = {
	default: async ({ request, locals }) => {
		const form = await superValidate(request, zod(workspaceSchema));

		if (!form.valid) {
			return fail(400, { form });
		}

		const slug = generateSlug(form.data.name);

		const [workspace] = await db
			.insert(workspaces)
			.values({
				name: form.data.name,
				slug,
				description: form.data.description,
				ownerId: locals.user!.id
			})
			.returning();

		throw redirect(303, `/workspaces/${workspace.slug}`);
	}
};
```

**Client form with Formsnap:**

```svelte
<!-- src/routes/workspaces/new/+page.svelte -->
<script lang="ts">
	import { superForm } from 'sveltekit-superforms';
	import { zodClient } from 'sveltekit-superforms/adapters';
	import { workspaceSchema } from '$lib/schemas/workspace';
	import * as Form from 'formsnap';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	const form = superForm(data.form, {
		validators: zodClient(workspaceSchema),
		resetForm: true
	});

	const { form: formData, enhance, errors, delayed } = form;
</script>

<div class="container mx-auto max-w-2xl p-6">
	<h1 class="text-3xl font-bold mb-6">Create Workspace</h1>

	<form method="POST" use:enhance class="space-y-4">
		<Form.Field {form} name="name">
			<Form.Control let:attrs>
				<Form.Label>Workspace Name</Form.Label>
				<input {...attrs} bind:value={$formData.name} class="input input-bordered w-full" />
			</Form.Control>
			<Form.FieldErrors class="text-error text-sm mt-1" />
		</Form.Field>

		<Form.Field {form} name="description">
			<Form.Control let:attrs>
				<Form.Label>Description</Form.Label>
				<textarea
					{...attrs}
					bind:value={$formData.description}
					class="textarea textarea-bordered w-full"
					rows="4"
				/>
			</Form.Control>
			<Form.FieldErrors class="text-error text-sm mt-1" />
		</Form.Field>

		<Form.Field {form} name="isPublic">
			<Form.Control let:attrs>
				<label class="label cursor-pointer justify-start gap-4">
					<input {...attrs} type="checkbox" bind:checked={$formData.isPublic} class="checkbox" />
					<span class="label-text">Make workspace public</span>
				</label>
			</Form.Control>
		</Form.Field>

		<button type="submit" class="btn btn-primary" disabled={$delayed}>
			{$delayed ? 'Creating...' : 'Create Workspace'}
		</button>
	</form>
</div>
```

**Benefits:**

- ✅ Client-side validation with Zod
- ✅ Server-side validation with same schema
- ✅ Automatic error handling
- ✅ Type-safe form data
- ✅ Form state management

---

## 7. Refactoring with SuperForms

### Clean Form Code

Extract form components for reusability.

```svelte
<!-- src/lib/components/forms/WorkspaceForm.svelte -->
<script lang="ts">
	import { superForm } from 'sveltekit-superforms';
	import { zodClient } from 'sveltekit-superforms/adapters';
	import { workspaceSchema } from '$lib/schemas/workspace';
	import * as Form from 'formsnap';

	let { data, action = '?/default' }: { data: any; action?: string } = $props();

	const form = superForm(data, {
		validators: zodClient(workspaceSchema)
	});

	const { form: formData, enhance, delayed } = form;
</script>

<form method="POST" {action} use:enhance class="space-y-4">
	<Form.Field {form} name="name">
		<Form.Control let:attrs>
			<Form.Label>Workspace Name</Form.Label>
			<input {...attrs} bind:value={$formData.name} class="input input-bordered w-full" />
		</Form.Control>
		<Form.FieldErrors class="text-error text-sm mt-1" />
	</Form.Field>

	<Form.Field {form} name="description">
		<Form.Control let:attrs>
			<Form.Label>Description</Form.Label>
			<textarea
				{...attrs}
				bind:value={$formData.description}
				class="textarea textarea-bordered w-full"
				rows="4"
			/>
		</Form.Control>
		<Form.FieldErrors class="text-error text-sm mt-1" />
	</Form.Field>

	<slot {delayed} />
</form>
```

**Usage:**

```svelte
<script lang="ts">
	import WorkspaceForm from '$lib/components/forms/WorkspaceForm.svelte';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<WorkspaceForm data={data.form} let:delayed>
	<button type="submit" class="btn btn-primary" disabled={delayed}>
		{delayed ? 'Creating...' : 'Create Workspace'}
	</button>
</WorkspaceForm>
```

---

## 8. Editing & Deleting Resources

### Update and Delete Actions

Implement full CRUD operations.

```typescript
// src/routes/workspaces/[slug]/settings/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import { superValidate } from 'sveltekit-superforms';
import { zod } from 'sveltekit-superforms/adapters';
import { workspaceSchema } from '$lib/schemas/workspace';
import { db } from '$lib/server/db';
import { workspaces } from '$lib/server/db/schema';
import { eq } from 'drizzle-orm';
import type { Actions, PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ parent }) => {
	const { workspace } = await parent();

	// Pre-populate form with existing data
	const form = await superValidate(
		{
			name: workspace.name,
			description: workspace.description || '',
			isPublic: workspace.isPublic
		},
		zod(workspaceSchema)
	);

	return { form };
};

export const actions: Actions = {
	update: async ({ request, params, locals }) => {
		const form = await superValidate(request, zod(workspaceSchema));

		if (!form.valid) {
			return fail(400, { form });
		}

		const slug = generateSlug(form.data.name);

		await db
			.update(workspaces)
			.set({
				name: form.data.name,
				slug,
				description: form.data.description,
				updatedAt: new Date()
			})
			.where(eq(workspaces.slug, params.slug));

		throw redirect(303, `/workspaces/${slug}/settings`);
	},

	delete: async ({ params, locals }) => {
		await db.delete(workspaces).where(eq(workspaces.slug, params.slug));

		throw redirect(303, '/workspaces');
	}
};
```

```svelte
<!-- src/routes/workspaces/[slug]/settings/+page.svelte -->
<script lang="ts">
	import WorkspaceForm from '$lib/components/forms/WorkspaceForm.svelte';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
	let showDeleteConfirm = $state(false);
</script>

<div class="container mx-auto max-w-2xl p-6">
	<h1 class="text-3xl font-bold mb-6">Workspace Settings</h1>

	<!-- Update Form -->
	<div class="card bg-base-100 shadow mb-6">
		<div class="card-body">
			<h2 class="card-title">General Settings</h2>
			<WorkspaceForm data={data.form} action="?/update" let:delayed>
				<button type="submit" class="btn btn-primary" disabled={delayed}>
					{delayed ? 'Saving...' : 'Save Changes'}
				</button>
			</WorkspaceForm>
		</div>
	</div>

	<!-- Delete Section -->
	<div class="card bg-base-100 shadow border-error">
		<div class="card-body">
			<h2 class="card-title text-error">Danger Zone</h2>
			<p class="text-base-content/60">Once you delete a workspace, there is no going back.</p>

			{#if showDeleteConfirm}
				<form method="POST" action="?/delete">
					<div class="alert alert-warning mb-4">
						<span>Are you sure? This action cannot be undone!</span>
					</div>
					<div class="flex gap-2">
						<button type="submit" class="btn btn-error"> Yes, Delete Workspace </button>
						<button type="button" class="btn btn-ghost" onclick={() => (showDeleteConfirm = false)}>
							Cancel
						</button>
					</div>
				</form>
			{:else}
				<button class="btn btn-error btn-outline" onclick={() => (showDeleteConfirm = true)}>
					Delete Workspace
				</button>
			{/if}
		</div>
	</div>
</div>
```

---

## 9. Shallow Routing with Modals

### Modal Routes Without Full Navigation

Open routes in modals while preserving URL state.

```typescript
// src/routes/workspaces/+page.server.ts
export const load: PageServerLoad = async ({ url }) => {
	const workspaces = await db.select().from(workspaces);

	// Check if modal should be shown
	const showNewModal = url.searchParams.has('new');

	let newWorkspaceForm;
	if (showNewModal) {
		newWorkspaceForm = await superValidate(zod(workspaceSchema));
	}

	return {
		workspaces,
		showNewModal,
		newWorkspaceForm
	};
};
```

```svelte
<!-- src/routes/workspaces/+page.svelte -->
<script lang="ts">
	import { goto } from '$app/navigation';
	import { page } from '$app/stores';
	import WorkspaceForm from '$lib/components/forms/WorkspaceForm.svelte';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	function closeModal() {
		// Remove query param (shallow routing)
		goto('/workspaces', { replaceState: true });
	}
</script>

<div class="container mx-auto p-6">
	<div class="flex justify-between items-center mb-6">
		<h1 class="text-3xl font-bold">Workspaces</h1>
		<a href="/workspaces?new" class="btn btn-primary"> + New Workspace </a>
	</div>

	<div class="grid md:grid-cols-3 gap-6">
		{#each data.workspaces as workspace}
			<div class="card bg-base-100 shadow">
				<div class="card-body">
					<h2 class="card-title">{workspace.name}</h2>
					<p>{workspace.description || 'No description'}</p>
					<div class="card-actions">
						<a href="/workspaces/{workspace.slug}" class="btn btn-primary btn-sm"> Open </a>
						<a href="?edit={workspace.slug}" class="btn btn-ghost btn-sm"> Edit </a>
					</div>
				</div>
			</div>
		{/each}
	</div>
</div>

<!-- New Workspace Modal -->
{#if data.showNewModal && data.newWorkspaceForm}
	<div class="modal modal-open">
		<div class="modal-box max-w-2xl">
			<button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2" onclick={closeModal}>
				✕
			</button>

			<h3 class="font-bold text-lg mb-4">Create Workspace</h3>

			<WorkspaceForm data={data.newWorkspaceForm} let:delayed>
				<div class="modal-action">
					<button type="button" class="btn btn-ghost" onclick={closeModal}> Cancel </button>
					<button type="submit" class="btn btn-primary" disabled={delayed}>
						{delayed ? 'Creating...' : 'Create'}
					</button>
				</div>
			</WorkspaceForm>
		</div>
		<div class="modal-backdrop" onclick={closeModal}></div>
	</div>
{/if}
```

**Benefits:**

- ✅ URL reflects modal state
- ✅ Browser back button closes modal
- ✅ Can share URL with modal open
- ✅ No full page reload
- ✅ Works with `replaceState` to avoid history pollution

---

## 10. Complete Example: Workspace Management Platform

### Full CRUD with Forms & Modals

Let's build the complete workspace management system!

**Features:**

- ✅ Create workspace (modal + dedicated page)
- ✅ Edit workspace (modal + settings page)
- ✅ Delete workspace with confirmation
- ✅ Form validation with SuperForms
- ✅ Optimistic updates
- ✅ Shallow routing for modals
- ✅ Toast notifications

### 📁 Project Structure

```
src/
├── lib/
│   ├── schemas/
│   │   └── workspace.ts
│   ├── components/
│   │   ├── forms/
│   │   │   └── WorkspaceForm.svelte
│   │   ├── modals/
│   │   │   ├── WorkspaceCreateModal.svelte
│   │   │   └── WorkspaceEditModal.svelte
│   │   └── Toast.svelte
│   └── stores/
│       └── toast.ts
├── routes/
│   └── workspaces/
│       ├── +page.server.ts
│       ├── +page.svelte
│       ├── new/
│       │   ├── +page.server.ts
│       │   └── +page.svelte
│       └── [slug]/
│           ├── +layout.server.ts
│           ├── +page.svelte
│           └── settings/
│               ├── +page.server.ts
│               └── +page.svelte
```

### Toast Store

```typescript
// src/lib/stores/toast.ts
import { writable } from 'svelte/store';

type Toast = {
	id: number;
	message: string;
	type: 'success' | 'error' | 'info';
};

function createToastStore() {
	const { subscribe, update } = writable<Toast[]>([]);

	let nextId = 0;

	function show(message: string, type: Toast['type'] = 'info', duration = 3000) {
		const id = nextId++;
		const toast: Toast = { id, message, type };

		update((toasts) => [...toasts, toast]);

		setTimeout(() => {
			remove(id);
		}, duration);
	}

	function remove(id: number) {
		update((toasts) => toasts.filter((t) => t.id !== id));
	}

	return {
		subscribe,
		success: (message: string) => show(message, 'success'),
		error: (message: string) => show(message, 'error'),
		info: (message: string) => show(message, 'info'),
		remove
	};
}

export const toast = createToastStore();
```

### Toast Component

```svelte
<!-- src/lib/components/Toast.svelte -->
<script lang="ts">
	import { toast } from '$lib/stores/toast';
	import { fly } from 'svelte/transition';
</script>

<div class="toast toast-end">
	{#each $toast as item (item.id)}
		<div
			class="alert"
			class:alert-success={item.type === 'success'}
			class:alert-error={item.type === 'error'}
			class:alert-info={item.type === 'info'}
			transition:fly={{ x: 300, duration: 300 }}
		>
			<span>{item.message}</span>
			<button class="btn btn-sm btn-ghost" onclick={() => toast.remove(item.id)}> ✕ </button>
		</div>
	{/each}
</div>
```

### Workspaces List Page

```typescript
// src/routes/workspaces/+page.server.ts
import { superValidate } from 'sveltekit-superforms';
import { zod } from 'sveltekit-superforms/adapters';
import { workspaceSchema } from '$lib/schemas/workspace';
import { db } from '$lib/server/db';
import { workspaces, workspaceMembers } from '$lib/server/db/schema';
import { eq } from 'drizzle-orm';
import type { Actions, PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ locals, url }) => {
	const userWorkspaces = await db
		.select({
			id: workspaces.id,
			name: workspaces.name,
			slug: workspaces.slug,
			description: workspaces.description,
			isPublic: workspaces.isPublic,
			createdAt: workspaces.createdAt,
			role: workspaceMembers.role
		})
		.from(workspaceMembers)
		.innerJoin(workspaces, eq(workspaceMembers.workspaceId, workspaces.id))
		.where(eq(workspaceMembers.userId, locals.user!.id));

	// Modal states
	const showNew = url.searchParams.has('new');
	const editSlug = url.searchParams.get('edit');

	let newForm, editForm, editWorkspace;

	if (showNew) {
		newForm = await superValidate(zod(workspaceSchema));
	}

	if (editSlug) {
		const [workspace] = await db
			.select()
			.from(workspaces)
			.where(eq(workspaces.slug, editSlug))
			.limit(1);

		if (workspace) {
			editWorkspace = workspace;
			editForm = await superValidate(
				{
					name: workspace.name,
					description: workspace.description || '',
					isPublic: workspace.isPublic
				},
				zod(workspaceSchema)
			);
		}
	}

	return {
		workspaces: userWorkspaces,
		showNew,
		newForm,
		editSlug,
		editForm,
		editWorkspace
	};
};

export const actions: Actions = {
	create: async ({ request, locals }) => {
		const form = await superValidate(request, zod(workspaceSchema));

		if (!form.valid) {
			return fail(400, { form });
		}

		const slug = generateSlug(form.data.name);

		await db.insert(workspaces).values({
			name: form.data.name,
			slug,
			description: form.data.description,
			ownerId: locals.user!.id
		});

		return { form, success: true };
	},

	update: async ({ request, url }) => {
		const editSlug = url.searchParams.get('edit');
		if (!editSlug) {
			return fail(400, { error: 'Missing workspace' });
		}

		const form = await superValidate(request, zod(workspaceSchema));

		if (!form.valid) {
			return fail(400, { form });
		}

		const newSlug = generateSlug(form.data.name);

		await db
			.update(workspaces)
			.set({
				name: form.data.name,
				slug: newSlug,
				description: form.data.description,
				updatedAt: new Date()
			})
			.where(eq(workspaces.slug, editSlug));

		return { form, success: true };
	},

	delete: async ({ url }) => {
		const editSlug = url.searchParams.get('edit');
		if (!editSlug) {
			return fail(400, { error: 'Missing workspace' });
		}

		await db.delete(workspaces).where(eq(workspaces.slug, editSlug));

		return { success: true, deleted: true };
	}
};
```

```svelte
<!-- src/routes/workspaces/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import { goto, invalidateAll } from '$app/navigation';
	import { superForm } from 'sveltekit-superforms';
	import { zodClient } from 'sveltekit-superforms/adapters';
	import { workspaceSchema } from '$lib/schemas/workspace';
	import { toast } from '$lib/stores/toast';
	import Toast from '$lib/components/Toast.svelte';
	import * as Form from 'formsnap';
	import type { PageData, ActionData } from './$types';

	let { data, form: actionResult }: { data: PageData; form: ActionData } = $props();

	// Handle action results
	$effect(() => {
		if (actionResult?.success) {
			if (actionResult.deleted) {
				toast.success('Workspace deleted');
			} else if (data.showNew) {
				toast.success('Workspace created');
			} else if (data.editSlug) {
				toast.success('Workspace updated');
			}
			closeModal();
			invalidateAll();
		}
	});

	// New workspace form
	const newWorkspaceForm = $derived(
		data.newForm
			? superForm(data.newForm, {
					validators: zodClient(workspaceSchema)
				})
			: null
	);

	// Edit workspace form
	const editWorkspaceForm = $derived(
		data.editForm
			? superForm(data.editForm, {
					validators: zodClient(workspaceSchema)
				})
			: null
	);

	function closeModal() {
		goto('/workspaces', { replaceState: true, noScroll: true });
	}

	let showDeleteConfirm = $state(false);
</script>

<Toast />

<div class="container mx-auto p-6">
	<div class="flex justify-between items-center mb-8">
		<div>
			<h1 class="text-4xl font-bold">Workspaces</h1>
			<p class="text-base-content/60">Manage your collaborative spaces</p>
		</div>
		<a href="?new" class="btn btn-primary gap-2">
			<svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
				<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M12 4v16m8-8H4" />
			</svg>
			New Workspace
		</a>
	</div>

	{#if data.workspaces.length === 0}
		<div class="text-center py-12">
			<p class="text-base-content/60 mb-4">You don't have any workspaces yet.</p>
			<a href="?new" class="btn btn-primary">Create Your First Workspace</a>
		</div>
	{:else}
		<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
			{#each data.workspaces as workspace}
				<div class="card bg-base-100 shadow hover:shadow-xl transition">
					<div class="card-body">
						<div class="flex justify-between items-start mb-2">
							<h2 class="card-title">{workspace.name}</h2>
							<div class="badge badge-outline">{workspace.role}</div>
						</div>

						<p class="text-base-content/60 text-sm flex-1">
							{workspace.description || 'No description'}
						</p>

						<div class="text-xs text-base-content/40 mb-4">
							Created {new Date(workspace.createdAt).toLocaleDateString()}
						</div>

						<div class="card-actions justify-end">
							<a href="/workspaces/{workspace.slug}" class="btn btn-primary btn-sm"> Open </a>
							{#if workspace.role === 'owner'}
								<a href="?edit={workspace.slug}" class="btn btn-ghost btn-sm"> Edit </a>
							{/if}
						</div>
					</div>
				</div>
			{/each}
		</div>
	{/if}
</div>

<!-- Create Modal -->
{#if data.showNew && newWorkspaceForm}
	{@const { form: formData, enhance: enhanceNew, delayed } = newWorkspaceForm}
	<div class="modal modal-open">
		<div class="modal-box max-w-2xl">
			<button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2" onclick={closeModal}>
				✕
			</button>

			<h3 class="font-bold text-2xl mb-6">Create Workspace</h3>

			<form method="POST" action="?/create" use:enhanceNew class="space-y-4">
				<Form.Field form={newWorkspaceForm} name="name">
					<Form.Control let:attrs>
						<Form.Label>Workspace Name</Form.Label>
						<input
							{...attrs}
							bind:value={$formData.name}
							class="input input-bordered w-full"
							placeholder="My Awesome Workspace"
						/>
					</Form.Control>
					<Form.FieldErrors class="text-error text-sm mt-1" />
				</Form.Field>

				<Form.Field form={newWorkspaceForm} name="description">
					<Form.Control let:attrs>
						<Form.Label>Description</Form.Label>
						<textarea
							{...attrs}
							bind:value={$formData.description}
							class="textarea textarea-bordered w-full"
							rows="4"
							placeholder="What's this workspace about?"
						/>
					</Form.Control>
					<Form.FieldErrors class="text-error text-sm mt-1" />
				</Form.Field>

				<Form.Field form={newWorkspaceForm} name="isPublic">
					<Form.Control let:attrs>
						<label class="label cursor-pointer justify-start gap-4">
							<input
								{...attrs}
								type="checkbox"
								bind:checked={$formData.isPublic}
								class="checkbox"
							/>
							<span class="label-text">Make workspace public</span>
						</label>
					</Form.Control>
				</Form.Field>

				<div class="modal-action">
					<button type="button" class="btn btn-ghost" onclick={closeModal}> Cancel </button>
					<button type="submit" class="btn btn-primary" disabled={$delayed}>
						{$delayed ? 'Creating...' : 'Create Workspace'}
					</button>
				</div>
			</form>
		</div>
		<div class="modal-backdrop" onclick={closeModal}></div>
	</div>
{/if}

<!-- Edit Modal -->
{#if data.editSlug && editWorkspaceForm && data.editWorkspace}
	{@const { form: formData, enhance: enhanceEdit, delayed } = editWorkspaceForm}
	<div class="modal modal-open">
		<div class="modal-box max-w-2xl">
			<button class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2" onclick={closeModal}>
				✕
			</button>

			<h3 class="font-bold text-2xl mb-6">Edit Workspace</h3>

			<form method="POST" action="?/update" use:enhanceEdit class="space-y-4">
				<Form.Field form={editWorkspaceForm} name="name">
					<Form.Control let:attrs>
						<Form.Label>Workspace Name</Form.Label>
						<input {...attrs} bind:value={$formData.name} class="input input-bordered w-full" />
					</Form.Control>
					<Form.FieldErrors class="text-error text-sm mt-1" />
				</Form.Field>

				<Form.Field form={editWorkspaceForm} name="description">
					<Form.Control let:attrs>
						<Form.Label>Description</Form.Label>
						<textarea
							{...attrs}
							bind:value={$formData.description}
							class="textarea textarea-bordered w-full"
							rows="4"
						/>
					</Form.Control>
					<Form.FieldErrors class="text-error text-sm mt-1" />
				</Form.Field>

				<div class="divider"></div>

				<!-- Delete Section -->
				<div class="p-4 border border-error rounded-lg">
					<h4 class="font-bold text-error mb-2">Danger Zone</h4>
					<p class="text-sm text-base-content/60 mb-4">
						Once you delete a workspace, there is no going back.
					</p>

					{#if showDeleteConfirm}
						<form method="POST" action="?/delete" use:enhance>
							<div class="alert alert-warning mb-4">
								<span>Are you sure? This action cannot be undone!</span>
							</div>
							<div class="flex gap-2">
								<button type="submit" class="btn btn-error btn-sm"> Yes, Delete </button>
								<button
									type="button"
									class="btn btn-ghost btn-sm"
									onclick={() => (showDeleteConfirm = false)}
								>
									Cancel
								</button>
							</div>
						</form>
					{:else}
						<button
							type="button"
							class="btn btn-error btn-outline btn-sm"
							onclick={() => (showDeleteConfirm = true)}
						>
							Delete Workspace
						</button>
					{/if}
				</div>

				<div class="modal-action">
					<button type="button" class="btn btn-ghost" onclick={closeModal}> Cancel </button>
					<button type="submit" class="btn btn-primary" disabled={$delayed}>
						{$delayed ? 'Saving...' : 'Save Changes'}
					</button>
				</div>
			</form>
		</div>
		<div class="modal-backdrop" onclick={closeModal}></div>
	</div>
{/if}
```

**Key Features Demonstrated:**

- ✅ **Form Actions**: Multiple named actions on one page
- ✅ **SuperForms**: Zod schema validation
- ✅ **Formsnap**: Clean form components
- ✅ **Progressive Enhancement**: Works without JavaScript
- ✅ **Shallow Routing**: Modals with URL state
- ✅ **Toast Notifications**: User feedback
- ✅ **Optimistic Updates**: Instant UI feedback
- ✅ **CRUD Operations**: Create, read, update, delete
- ✅ **Delete Confirmation**: Safety pattern

---

## 📝 Key Takeaways

✅ Form actions run on the server  
✅ Use `fail()` to return validation errors  
✅ `use:enhance` enables progressive enhancement  
✅ SuperForms provides type-safe validation  
✅ Formsnap simplifies form component code  
✅ Shallow routing shows modals without navigation  
✅ Always validate on both client and server  
✅ Preserve form values on validation errors  
✅ Use named actions for multiple operations  
✅ Show loading states during form submission  
✅ Implement delete confirmations  
✅ Toast notifications improve UX

---

## 🚀 Next Steps

You've mastered form actions and data mutations! Next up:

- **Section 17**: Authentication & Authorization
- **Section 18**: File Uploads & Storage
- Advanced form patterns (multi-step forms, file uploads)
- Real-time form collaboration
