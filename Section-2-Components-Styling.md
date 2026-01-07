# Section 2: Components & Styling

## 📚 Learning Objectives

By the end of this section, you will:

- Master snippets for reusable markup
- Use SASS and the class directive
- Extend HTML element attributes
- Apply dynamic styles with the style directive
- Use global CSS selectors
- Pass CSS variables as props
- Forward events and create custom events
- Understand event bubbling, capturing, and delegation
- Programmatically interact with components
- Dynamically render components with `<svelte:element>`

---

## Table of Contents

- [Section 2: Components \& Styling](#section-2-components--styling)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction to Snippets](#1-introduction-to-snippets)
    - [What are Snippets?](#what-are-snippets)
    - [📁 Files to Create](#-files-to-create)
  - [2. Passing Snippets as Props](#2-passing-snippets-as-props)
    - [What is This?](#what-is-this)
    - [📁 Files to Create](#-files-to-create-1)
    - [Usage:](#usage)
  - [3. Passing Arguments to Snippets](#3-passing-arguments-to-snippets)
    - [What is This?](#what-is-this-1)
    - [📁 Files to Create](#-files-to-create-2)
    - [Usage:](#usage-1)
  - [4. SASS \& Class Directive](#4-sass--class-directive)
    - [What is This?](#what-is-this-2)
    - [📁 Files to Create](#-files-to-create-3)
  - [5. HTML Attributes \& Spread Props](#5-html-attributes--spread-props)
    - [What is This?](#what-is-this-3)
    - [📁 Files to Create](#-files-to-create-4)
  - [6. Dynamic Styles \& CSS Variables](#6-dynamic-styles--css-variables)
    - [What is This?](#what-is-this-4)
    - [📁 Files to Create](#-files-to-create-5)
  - [7. Event Forwarding \& Custom Events](#7-event-forwarding--custom-events)
    - [What is This?](#what-is-this-5)
    - [📁 Files to Create](#-files-to-create-6)
  - [8. Component Bindings \& Programmatic Interaction](#8-component-bindings--programmatic-interaction)
    - [What is This?](#what-is-this-6)
    - [📁 Files to Create](#-files-to-create-7)
  - [9. Dynamic Components with svelte:element](#9-dynamic-components-with-svelteelement)
    - [What is This?](#what-is-this-7)
    - [📁 Files to Create](#-files-to-create-8)
  - [10. 🚀 End-of-Section Project: Component Library Dashboard](#10--end-of-section-project-component-library-dashboard)
    - [Project Overview](#project-overview)
    - [📁 Files to Create](#-files-to-create-9)
    - [What This Project Demonstrates](#what-this-project-demonstrates)
  - [📝 Key Takeaways](#-key-takeaways)
    - [💡 Best Practices for Component Architecture](#-best-practices-for-component-architecture)
    - [⚠️ Common Mistakes](#️-common-mistakes)
    - [⚡ Performance Tips](#-performance-tips)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Introduction to Snippets

### What are Snippets?

Snippets are reusable blocks of markup that can be defined and used multiple times within a component. Think of them as template fragments!

**Real-World Scenario:** You're building a card component that has multiple sections with similar layouts but different content.

**What it does:** Creates reusable markup patterns within a component.

### 📁 Files to Create

Create:

- `src/routes/snippets-intro/+page.svelte`

```svelte
<script lang="ts">
	interface Feature {
		icon: string;
		title: string;
		description: string;
	}

	const features: Feature[] = [
		{
			icon: '🚀',
			title: 'Fast',
			description: 'Lightning-fast performance'
		},
		{
			icon: '💪',
			title: 'Powerful',
			description: 'Built for production'
		},
		{
			icon: '🎨',
			title: 'Beautiful',
			description: 'Stunning design out of the box'
		}
	];
</script>

{#snippet featureCard(feature: Feature)}
	<div
		class="bg-gray-800 border-2 border-gray-700 rounded-2xl p-8 text-center transition-all duration-300 hover:border-blue-400 hover:-translate-y-2 hover:shadow-2xl"
	>
		<div class="text-6xl mb-4">{feature.icon}</div>
		<h3 class="m-0 mb-3 text-white text-2xl">{feature.title}</h3>
		<p class="m-0 text-gray-400 leading-relaxed">{feature.description}</p>
	</div>
{/snippet}

{#snippet headerSection(title: string, subtitle: string)}
	<div class="text-center mb-12">
		<h1 class="m-0 mb-3 text-blue-400 text-4xl font-extrabold">{title}</h1>
		<p class="m-0 text-gray-500 text-lg">{subtitle}</p>
	</div>
{/snippet}

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	{@render headerSection('Our Features', 'Everything you need to succeed')}

	<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6 max-w-7xl mx-auto mb-16">
		{#each features as feature}
			{@render featureCard(feature)}
		{/each}
	</div>

	{@render headerSection('Why Choose Us?', 'Built by developers, for developers')}
</div>
```

**Key Concepts:**

- **{#snippet name(params)}**: Define reusable markup
- **{@render snippetName(args)}**: Use the snippet
- **Type-safe**: TypeScript parameters work
- **Scoped**: Only available in same component

---

## 2. Passing Snippets as Props

### What is This?

Pass snippets to child components as props! Like React's render props or Vue's scoped slots.

**Real-World Scenario:** You're building a modal component where the parent controls the content and actions.

**What it does:** Creates a flexible modal that accepts custom content via snippets.

### 📁 Files to Create

Create:

- `src/lib/components/Modal.svelte` (reusable modal component)
- `src/routes/modal-demo/+page.svelte` (page demonstrating the modal)

```svelte
<!-- Modal.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		isOpen: boolean;
		onClose: () => void;
		title: string;
		children: Snippet;
		footer?: Snippet;
	}

	let { isOpen, onClose, title, children, footer }: Props = $props();
</script>

{#if isOpen}
	<div
		class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 backdrop-blur"
		onclick={onClose}
	>
		<div
			class="bg-gray-800 border-2 border-gray-700 rounded-2xl max-w-xl w-11/12 max-h-screen overflow-auto shadow-2xl"
			onclick={(e) => e.stopPropagation()}
		>
			<div class="flex justify-between items-center p-6 border-b-2 border-gray-700">
				<h2 class="m-0 text-blue-400 text-2xl">{title}</h2>
				<button
					class="bg-transparent border-none text-gray-500 text-3xl cursor-pointer p-0 w-8 h-8 flex items-center justify-center rounded-md transition-all duration-200 hover:bg-gray-700 hover:text-white"
					onclick={onClose}>✕</button
				>
			</div>

			<div class="p-6 text-gray-200 leading-relaxed">
				{@render children()}
			</div>

			{#if footer}
				<div class="p-6 border-t-2 border-gray-700 flex gap-3 justify-end">
					{@render footer()}
				</div>
			{/if}
		</div>
	</div>
{/if}
```

### Usage:

```svelte
<script lang="ts">
	import Modal from './Modal.svelte';

	let isDeleteModalOpen = $state(false);
	let isConfirmModalOpen = $state(false);

	function handleDelete() {
		console.log('Deleted!');
		isDeleteModalOpen = false;
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">Modal with Snippets Demo</h1>

	<div class="flex gap-4 justify-center">
		<button
			onclick={() => (isDeleteModalOpen = true)}
			class="px-7 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-red-400 text-white hover:bg-red-300 hover:-translate-y-0.5"
		>
			Delete Account
		</button>
		<button
			onclick={() => (isConfirmModalOpen = true)}
			class="px-7 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-blue-400 text-black hover:bg-blue-300 hover:-translate-y-0.5"
		>
			Show Confirmation
		</button>
	</div>

	<Modal
		isOpen={isDeleteModalOpen}
		onClose={() => (isDeleteModalOpen = false)}
		title="⚠️ Delete Account"
	>
		{#snippet children()}
			<p>Are you sure you want to delete your account?</p>
			<p>This action <strong>cannot be undone</strong>.</p>
			<ul>
				<li>All your data will be permanently removed</li>
				<li>Your subscription will be cancelled</li>
				<li>You'll lose access to all features</li>
			</ul>
		{/snippet}

		{#snippet footer()}
			<button
				onclick={() => (isDeleteModalOpen = false)}
				class="px-7 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-gray-700 text-white hover:bg-gray-600"
			>
				Cancel
			</button>
			<button
				onclick={handleDelete}
				class="px-7 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-red-400 text-white hover:bg-red-300 hover:-translate-y-0.5"
			>
				Yes, Delete My Account
			</button>
		{/snippet}
	</Modal>

	<Modal
		isOpen={isConfirmModalOpen}
		onClose={() => (isConfirmModalOpen = false)}
		title="✅ Success!"
	>
		{#snippet children()}
			<p>Your changes have been saved successfully!</p>
			<p>You can now continue using the app.</p>
		{/snippet}

		{#snippet footer()}
			<button
				onclick={() => (isConfirmModalOpen = false)}
				class="px-7 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-blue-400 text-black hover:bg-blue-300 hover:-translate-y-0.5"
			>
				Got It
			</button>
		{/snippet}
	</Modal>
</div>
```

**Key Concepts:**

- **Snippet<T>**: Type for snippet props
- **children**: Default snippet prop
- **Optional snippets**: Use `snippet?: Snippet`
- **{@render snippet()}**: Render passed snippets
- **Flexible components**: Parents control content

---

## 3. Passing Arguments to Snippets

### What is This?

Snippets can receive arguments from the rendering site! Like function parameters.

**Real-World Scenario:** You're building a data table where each row can have custom rendering logic.

**What it does:** Creates a flexible table with customizable cell rendering.

### 📁 Files to Create

Create:

- `src/lib/components/DataTable.svelte` (reusable table component)
- `src/routes/data-table-demo/+page.svelte` (page using the table)

```svelte
<!-- DataTable.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props<T> {
		data: T[];
		columns: {
			key: string;
			label: string;
			render?: Snippet<[T]>;
		}[];
	}

	let { data, columns }: Props<any> = $props();
</script>

<div class="bg-gray-800 border-2 border-gray-700 rounded-xl overflow-hidden">
	<table class="w-full border-collapse">
		<thead class="bg-gray-900">
			<tr>
				{#each columns as column}
					<th class="p-4 text-left text-blue-400 font-bold text-sm uppercase tracking-wide"
						>{column.label}</th
					>
				{/each}
			</tr>
		</thead>
		<tbody>
			{#each data as row}
				<tr class="border-t border-gray-700 transition-colors duration-200 hover:bg-gray-700">
					{#each columns as column}
						<td class="p-4 text-gray-200">
							{#if column.render}
								{@render column.render(row)}
							{:else}
								{row[column.key]}
							{/if}
						</td>
					{/each}
				</tr>
			{/each}
		</tbody>
	</table>
</div>
```

### Usage:

```svelte
<script lang="ts">
	import DataTable from './DataTable.svelte';

	interface User {
		id: number;
		name: string;
		email: string;
		role: 'admin' | 'user' | 'moderator';
		status: 'active' | 'inactive';
		joinedAt: string;
	}

	const users: User[] = [
		{
			id: 1,
			name: 'Alice Johnson',
			email: 'alice@example.com',
			role: 'admin',
			status: 'active',
			joinedAt: '2024-01-15'
		},
		{
			id: 2,
			name: 'Bob Smith',
			email: 'bob@example.com',
			role: 'user',
			status: 'active',
			joinedAt: '2024-03-20'
		},
		{
			id: 3,
			name: 'Carol White',
			email: 'carol@example.com',
			role: 'moderator',
			status: 'inactive',
			joinedAt: '2023-12-10'
		}
	];

	function getRoleBadgeColor(role: string) {
		return {
			admin: 'rgb(239 68 68)',
			moderator: 'rgb(249 115 22)',
			user: 'rgb(59 130 246)'
		}[role] || 'rgb(156 163 175)';
	}

	function getStatusColor(status: string) {
		return status === 'active' ? 'rgb(74 222 128)' : 'rgb(156 163 175)';
	}
</script>

{#snippet roleBadge(user: User)}
	<span class="badge" style="background: {getRoleBadgeColor(user.role)};">
		{user.role}
	</span>
{/snippet}

{#snippet statusIndicator(user: User)}
	<span class="status-indicator" style="color: {getStatusColor(user.status)};">
		● {user.status}
	</span>
{/snippet}

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-8 text-4xl">👥 User Management</h1>

	<DataTable
		data={users}
		columns={[
			{ key: 'name', label: 'Name' },
			{ key: 'email', label: 'Email' },
			{
				key: 'role',
				label: 'Role',
				render: roleBadge
			},
			{
				key: 'status',
				label: 'Status',
				render: statusIndicator
			},
			{ key: 'joinedAt', label: 'Joined' }
		]}
	/>
</div>

<style>
	:global(.badge) {
		padding: 6px 12px;
		border-radius: 20px;
		font-size: 12px;
		font-weight: 700;
		text-transform: uppercase;
		letter-spacing: 0.5px;
		color: rgb(0 0 0);
	}

	:global(.status-indicator) {
		font-weight: 600;
		text-transform: capitalize;
	}
</style>
```

**Key Concepts:**

- **Snippet<[ArgType]>**: Type snippets with arguments
- **{@render snippet(arg)}**: Pass arguments when rendering
- **Custom rendering**: Control how each cell displays
- **Type-safe**: Full TypeScript support

---

## 4. SASS & Class Directive

### What is This?

Use SASS for nested styles and the `class:` directive for conditional classes.

**Real-World Scenario:** You're building a notification system with different states and need conditional styling.

**What it does:** Creates notifications with dynamic classes based on state.

### 📁 Files to Create

Create:

- `src/routes/notifications/+page.svelte`

```svelte
<script lang="ts">
	type NotificationType = 'success' | 'error' | 'warning' | 'info';

	interface Notification {
		id: number;
		type: NotificationType;
		title: string;
		message: string;
		dismissible: boolean;
	}

	let notifications = $state<Notification[]>([
		{
			id: 1,
			type: 'success',
			title: 'Success!',
			message: 'Your changes have been saved.',
			dismissible: true
		},
		{
			id: 2,
			type: 'error',
			title: 'Error',
			message: 'Failed to connect to server.',
			dismissible: true
		},
		{
			id: 3,
			type: 'warning',
			title: 'Warning',
			message: 'Your session will expire in 5 minutes.',
			dismissible: false
		},
		{
			id: 4,
			type: 'info',
			title: 'Info',
			message: 'New features are available!',
			dismissible: true
		}
	]);

	function dismiss(id: number) {
		notifications = notifications.filter((n) => n.id !== id);
	}

	function addNotification(type: NotificationType) {
		const titles = {
			success: 'Success!',
			error: 'Error!',
			warning: 'Warning!',
			info: 'Info!'
		};

		notifications = [
			...notifications,
			{
				id: Date.now(),
				type,
				title: titles[type],
				message: `This is a ${type} notification`,
				dismissible: true
			}
		];
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-8 text-4xl">🔔 Notification System</h1>

	<div class="flex gap-3 justify-center mb-10 flex-wrap">
		<button onclick={() => addNotification('success')} class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer transition-all duration-200 bg-green-400 text-black hover:-translate-y-0.5 hover:shadow-lg"> Add Success </button>
		<button onclick={() => addNotification('error')} class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer transition-all duration-200 bg-red-400 text-white hover:-translate-y-0.5 hover:shadow-lg"> Add Error </button>
		<button onclick={() => addNotification('warning')} class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer transition-all duration-200 bg-orange-500 text-black hover:-translate-y-0.5 hover:shadow-lg"> Add Warning </button>
		<button onclick={() => addNotification('info')} class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer transition-all duration-200 bg-blue-400 text-black hover:-translate-y-0.5 hover:shadow-lg"> Add Info </button>
	</div>

	<div class="max-w-2xl mx-auto flex flex-col gap-4">
		{#each notifications as notification (notification.id)}
			<div
				class="border-2 rounded-xl p-5 flex items-start justify-between gap-4 relative transition-all duration-300"
				class:border-green-400={notification.type === 'success'}
				class:bg-green-400/5={notification.type === 'success'}
				class:border-red-400={notification.type === 'error'}
				class:bg-red-400/5={notification.type === 'error'}
				class:border-orange-500={notification.type === 'warning'}
				class:bg-orange-500/5={notification.type === 'warning'}
				class:border-blue-400={notification.type === 'info'}
				class:bg-blue-400/5={notification.type === 'info'}
			>
				<div class="flex gap-4 flex-1">
					<div class="text-3xl">
						{#if notification.type === 'success'}✅
						{:else if notification.type === 'error'}❌
						{:else if notification.type === 'warning'}⚠️
						{:else}ℹ️
						{/if}
					</div>
					<div>
						<h3 class="m-0 mb-2 text-white text-lg">{notification.title}</h3>
						<p class="m-0 text-gray-400 text-sm leading-relaxed">{notification.message}</p>
					</div>
				</div>
				{#if notification.dismissible}
					<button class="bg-transparent border-none text-gray-500 text-xl cursor-pointer p-1 px-2 rounded transition-all duration-200 hover:bg-white/10 hover:text-white" onclick={() => dismiss(notification.id)}> ✕ </button>
				{/if}
			</div>
		{/each}
	</div>
</div>

<style>
	@keyframes slideIn {
		from {
			transform: translateX(-100%);
			opacity: 0;
		}
		to {
			transform: translateX(0);
			opacity: 1;
		}
	}
</style>
```

**Key Concepts:**

- **class:name={condition}**: Add class conditionally
- **class:name**: Shorthand when var name matches class
- **SASS nesting**: Cleaner, more organized styles
- **Multiple classes**: Chain multiple `class:` directives

---

## 5. HTML Attributes & Spread Props

### What is This?

Learn how to spread HTML attributes to child elements and make components more flexible by accepting any HTML attribute.

**Real-World Scenario:** You're creating a button component that should accept all standard button attributes (disabled, type, aria-labels, etc.) without explicitly defining each one.

**What it does:** Demonstrates attribute spreading and forwarding for flexible, reusable components.

### 📁 Files to Create

Create:

- `src/lib/components/Button.svelte` - Reusable button with attribute spreading
- `src/routes/section-2/attributes-demo/+page.svelte` - Demo page

```svelte
<!-- Button.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { HTMLButtonAttributes } from 'svelte/elements';

	interface Props extends HTMLButtonAttributes {
		variant?: 'primary' | 'secondary' | 'danger';
		size?: 'sm' | 'md' | 'lg';
		children: Snippet;
	}

	let { variant = 'primary', size = 'md', children, ...restProps }: Props = $props();

	const variantClasses = {
		primary: 'bg-blue-400 text-black hover:bg-blue-300',
		secondary: 'bg-gray-700 text-white hover:bg-gray-600',
		danger: 'bg-red-400 text-white hover:bg-red-300'
	};

	const sizeClasses = {
		sm: 'px-3 py-1.5 text-sm',
		md: 'px-6 py-3 text-base',
		lg: 'px-8 py-4 text-lg'
	};
</script>

<button
	class="border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:-translate-y-0.5 {variantClasses[
		variant
	]} {sizeClasses[size]}"
	{...restProps}
>
	{@render children()}
</button>
```

```svelte
<!-- attributes-demo/+page.svelte -->
<script lang="ts">
	import Button from '$lib/components/Button.svelte';

	let clickCount = $state(0);

	function handleClick() {
		clickCount++;
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		📝 HTML Attributes & Spread Props
	</h1>

	<div class="max-w-4xl mx-auto space-y-6">
		<!-- Standard Usage -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Standard Button Variants</h2>
			<div class="flex gap-3 flex-wrap">
				<Button variant="primary" onclick={handleClick}>Primary Button</Button>
				<Button variant="secondary" onclick={handleClick}>Secondary Button</Button>
				<Button variant="danger" onclick={handleClick}>Danger Button</Button>
			</div>
			<p class="mt-4 text-gray-400">Clicked {clickCount} times</p>
		</div>

		<!-- Sizes -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Sizes</h2>
			<div class="flex gap-3 items-center flex-wrap">
				<Button size="sm">Small</Button>
				<Button size="md">Medium</Button>
				<Button size="lg">Large</Button>
			</div>
		</div>

		<!-- HTML Attributes (spread props) -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">HTML Attributes (via Spread)</h2>
			<div class="flex gap-3 flex-wrap">
				<Button disabled>Disabled</Button>
				<Button type="submit">Submit Type</Button>
				<Button title="This is a tooltip">Hover for Title</Button>
				<Button aria-label="Close dialog" variant="danger">✕ Close</Button>
				<Button data-testid="custom-button" id="my-button">Custom Attrs</Button>
			</div>
		</div>

		<!-- Explanation -->
		<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-6">
			<h3 class="m-0 mb-3 text-blue-400 font-bold">💡 How It Works</h3>
			<ul class="m-0 pl-5 space-y-2">
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded text-blue-400">...restProps</code> captures all extra
					props
				</li>
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded text-blue-400">{'{...restProps}'}</code> spreads
					them onto the element
				</li>
				<li>Accepts any HTML attribute: disabled, aria-*, data-*, id, class, etc.</li>
				<li>Makes components flexible without explicitly defining every prop</li>
			</ul>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Spread props**: `{...restProps}` forwards all unhandled props
- **Type safety**: Extend `HTMLButtonAttributes` from `svelte/elements` for proper typing
- **HTML attributes**: disabled, type, aria-_, data-_, etc. work automatically
- **Component flexibility**: Don't need to explicitly define every possible attribute

---

## 6. Dynamic Styles & CSS Variables

### What is This?

Apply dynamic inline styles and pass CSS custom properties (variables) as props to control component styling from the parent.

**Real-World Scenario:** You're building a progress bar component where the color and width can be customized by the parent, or a themed card component where the accent color is configurable.

**What it does:** Shows how to use the `style:` directive and pass CSS variables as props.

### 📁 Files to Create

Create:

- `src/lib/components/ProgressBar.svelte`
- `src/routes/section-2/styles-demo/+page.svelte`

```svelte
<!-- ProgressBar.svelte -->
<script lang="ts">
	interface Props {
		progress: number;
		color?: string;
		height?: string;
		label?: string;
	}

	let { progress, color = '#4a9eff', height = '24px', label }: Props = $props();

	const percentage = $derived(Math.min(100, Math.max(0, progress)));
</script>

<div class="bg-gray-800 rounded-full overflow-hidden" style:height>
	<div
		class="h-full flex items-center justify-center text-xs font-bold text-white transition-all duration-300"
		style:width="{percentage}%"
		style:background-color={color}
		style:--custom-color={color}
	>
		{#if label}
			<span>{label}</span>
		{:else}
			{percentage}%
		{/if}
	</div>
</div>
```

```svelte
<!-- styles-demo/+page.svelte -->
<script lang="ts">
	import ProgressBar from '$lib/components/ProgressBar.svelte';

	let progress1 = $state(65);
	let progress2 = $state(40);
	let progress3 = $state(85);
	let customColor = $state('#4a9eff');
	let customSize = $state('30');
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		🎨 Dynamic Styles & CSS Variables
	</h1>

	<div class="max-w-4xl mx-auto space-y-6">
		<!-- Progress Bars -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Progress Bars with Dynamic Styles</h2>
			<div class="space-y-6">
				<div>
					<label class="block mb-2 text-sm text-gray-400">Success: {progress1}%</label>
					<ProgressBar progress={progress1} color="#4ade80" />
					<input
						type="range"
						bind:value={progress1}
						min="0"
						max="100"
						class="w-full mt-2 accent-green-400"
					/>
				</div>

				<div>
					<label class="block mb-2 text-sm text-gray-400">Warning: {progress2}%</label>
					<ProgressBar progress={progress2} color="#fb923c" />
					<input
						type="range"
						bind:value={progress2}
						min="0"
						max="100"
						class="w-full mt-2 accent-orange-400"
					/>
				</div>

				<div>
					<label class="block mb-2 text-sm text-gray-400">Info: {progress3}%</label>
					<ProgressBar progress={progress3} color="#60a5fa" />
					<input
						type="range"
						bind:value={progress3}
						min="0"
						max="100"
						class="w-full mt-2 accent-blue-400"
					/>
				</div>
			</div>
		</div>

		<!-- Custom Color and Size -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Custom Color & Size</h2>
			<div class="space-y-4">
				<div>
					<label class="block mb-2 text-sm text-gray-400">Color: {customColor}</label>
					<input type="color" bind:value={customColor} class="w-full h-12 cursor-pointer" />
				</div>

				<div>
					<label class="block mb-2 text-sm text-gray-400">Height: {customSize}px</label>
					<input
						type="range"
						bind:value={customSize}
						min="16"
						max="60"
						class="w-full accent-blue-400"
					/>
				</div>

				<div class="mt-4">
					<ProgressBar progress={75} color={customColor} height="{customSize}px" label="Custom!" />
				</div>
			</div>
		</div>

		<!-- style: Directive Examples -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">style: Directive Examples</h2>
			<div class="space-y-4">
				<div
					class="p-4 rounded-lg bg-gray-900"
					style:color="#4ade80"
					style:border="2px solid #4ade80"
				>
					Dynamic border and text color
				</div>
				<div
					class="p-4 rounded-lg bg-gray-900"
					style:transform="rotate({progress1 / 10}deg)"
					style:transition="transform 0.3s"
				>
					Rotation based on progress: {(progress1 / 10).toFixed(1)}°
				</div>
				<div
					class="p-4 rounded-lg bg-gray-900"
					style:opacity={progress2 / 100}
					style:--custom-shadow="0 4px 12px rgba(74, 158, 255, 0.3)"
					style="box-shadow: var(--custom-shadow)"
				>
					Opacity based on progress: {(progress2 / 100).toFixed(2)}
				</div>
			</div>
		</div>

		<!-- Explanation -->
		<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-6">
			<h3 class="m-0 mb-3 text-blue-400 font-bold">💡 style: Directive</h3>
			<ul class="m-0 pl-5 space-y-2">
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded text-blue-400">style:property={'{value}'}</code
					> for dynamic inline styles
				</li>
				<li>Works with any CSS property: color, width, transform, etc.</li>
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded text-blue-400"
						>style:--css-var={'{value}'}</code
					> for CSS custom properties
				</li>
				<li>More performant than string concatenation</li>
				<li>Type-safe with TypeScript</li>
			</ul>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **style: directive**: `style:property={value}` for dynamic inline styles
- **CSS variables**: `style:--var-name={value}` for custom properties
- **Reactivity**: Styles update automatically when values change
- **Flexibility**: Pass colors, sizes, and styles as props
- **Performance**: Better than string manipulation

---

## 7. Event Forwarding & Custom Events

### What is This?

Learn how to forward DOM events from child components and create custom events for component communication.

**Real-World Scenario:** You're building a custom input component that needs to bubble up native events (focus, blur, input) and emit custom events (validation, submit).

**What it does:** Demonstrates event forwarding and creating custom events.

### 📁 Files to Create

Create:

- `src/lib/components/CustomInput.svelte`
- `src/routes/section-2/events-demo/+page.svelte`

```svelte
<!-- CustomInput.svelte -->
<script lang="ts">
	interface Props {
		value: string;
		placeholder?: string;
		type?: string;
		maxlength?: number;
		oninput?: (e: Event) => void;
		onfocus?: (e: FocusEvent) => void;
		onblur?: (e: FocusEvent) => void;
		onvalidate?: (isValid: boolean, value: string) => void;
		onclear?: () => void;
	}

	let {
		value = $bindable(),
		placeholder = '',
		type = 'text',
		maxlength,
		oninput,
		onfocus,
		onblur,
		onvalidate,
		onclear
	}: Props = $props();

	let isFocused = $state(false);
	let charCount = $derived(value.length);
	let isValid = $derived(charCount >= 3);

	function handleInput(e: Event) {
		const target = e.target as HTMLInputElement;
		value = target.value;
		oninput?.(e);
		onvalidate?.(isValid, value);
	}

	function handleFocus(e: FocusEvent) {
		isFocused = true;
		onfocus?.(e);
	}

	function handleBlur(e: FocusEvent) {
		isFocused = false;
		onblur?.(e);
	}

	function handleClear() {
		value = '';
		onclear?.();
		onvalidate?.(false, '');
	}
</script>

<div class="relative">
	<input
		{type}
		{placeholder}
		{maxlength}
		{value}
		oninput={handleInput}
		onfocus={handleFocus}
		onblur={handleBlur}
		class="w-full px-4 py-3 bg-gray-900 border-2 rounded-lg text-white pr-24 transition-colors"
		class:border-blue-400={isFocused}
		class:border-green-400={!isFocused && isValid && charCount > 0}
		class:border-red-400={!isFocused && !isValid && charCount > 0}
		class:border-gray-700={!isFocused && charCount === 0}
	/>
	{#if value}
		<button
			onclick={handleClear}
			class="absolute right-2 top-1/2 -translate-y-1/2 px-3 py-1 bg-gray-700 text-white text-sm rounded hover:bg-gray-600"
		>
			Clear
		</button>
	{/if}
	{#if maxlength}
		<div class="text-right text-xs mt-1 text-gray-500">
			{charCount} / {maxlength}
		</div>
	{/if}
</div>
```

```svelte
<!-- events-demo/+page.svelte -->
<script lang="ts">
	import CustomInput from '$lib/components/CustomInput.svelte';

	let name = $state('');
	let email = $state('');
	let message = $state('');

	let eventLog = $state<string[]>([]);

	function logEvent(event: string) {
		eventLog = [`[${new Date().toLocaleTimeString()}] ${event}`, ...eventLog.slice(0, 9)];
	}

	function handleValidate(isValid: boolean, value: string, field: string) {
		logEvent(`${field} validation: ${isValid ? '✓ Valid' : '✗ Invalid'} (${value.length} chars)`);
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		⚡ Event Forwarding & Custom Events
	</h1>

	<div class="max-w-4xl mx-auto grid gap-6">
		<!-- Form with Custom Events -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Custom Input Components</h2>
			<div class="space-y-4">
				<div>
					<label class="block mb-2 text-sm font-semibold">Name (min 3 chars)</label>
					<CustomInput
						bind:value={name}
						placeholder="Enter your name"
						maxlength={50}
						onfocus={() => logEvent('Name input focused')}
						onblur={() => logEvent('Name input blurred')}
						oninput={() => logEvent('Name input changed')}
						onvalidate={(valid, val) => handleValidate(valid, val, 'Name')}
						onclear={() => logEvent('Name cleared')}
					/>
				</div>

				<div>
					<label class="block mb-2 text-sm font-semibold">Email</label>
					<CustomInput
						bind:value={email}
						type="email"
						placeholder="your@email.com"
						maxlength={100}
						onfocus={() => logEvent('Email input focused')}
						onvalidate={(valid, val) => handleValidate(valid, val, 'Email')}
					/>
				</div>

				<div>
					<label class="block mb-2 text-sm font-semibold">Message</label>
					<CustomInput
						bind:value={message}
						placeholder="Your message..."
						maxlength={200}
						onfocus={() => logEvent('Message input focused')}
						onvalidate={(valid, val) => handleValidate(valid, val, 'Message')}
					/>
				</div>
			</div>
		</div>

		<!-- Event Log -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Event Log (Last 10)</h2>
			{#if eventLog.length === 0}
				<p class="text-gray-500 italic">No events yet. Interact with the inputs above!</p>
			{:else}
				<ul class="m-0 p-0 list-none space-y-2 font-mono text-sm">
					{#each eventLog as log}
						<li class="bg-gray-900 p-3 rounded border-l-4 border-blue-400">{log}</li>
					{/each}
				</ul>
			{/if}
		</div>

		<!-- Current Values -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Current Values</h2>
			<div class="bg-gray-900 p-4 rounded-lg font-mono text-sm">
				<div class="mb-2">
					<span class="text-gray-500">name:</span>
					<span class="text-green-400">"{name}"</span>
				</div>
				<div class="mb-2">
					<span class="text-gray-500">email:</span>
					<span class="text-green-400">"{email}"</span>
				</div>
				<div>
					<span class="text-gray-500">message:</span>
					<span class="text-green-400">"{message}"</span>
				</div>
			</div>
		</div>

		<!-- Explanation -->
		<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-6">
			<h3 class="m-0 mb-3 text-blue-400 font-bold">💡 Event System</h3>
			<ul class="m-0 pl-5 space-y-2">
				<li>
					<strong>Forward native events:</strong> Accept
					<code class="bg-gray-900 px-2 py-1 rounded">oninput</code>,
					<code class="bg-gray-900 px-2 py-1 rounded">onfocus</code>, etc. as props
				</li>
				<li>
					<strong>Custom events:</strong> Create props like
					<code class="bg-gray-900 px-2 py-1 rounded">onvalidate</code> and call them
				</li>
				<li>
					<strong>Bindable props:</strong> Use
					<code class="bg-gray-900 px-2 py-1 rounded">$bindable()</code> for two-way binding
				</li>
				<li><strong>Optional chaining:</strong> Use <code>?.() </code> to safely call callbacks</li>
			</ul>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Event forwarding**: Accept event handlers as props (oninput, onfocus, etc.)
- **Custom events**: Create custom callbacks like onvalidate, onclear
- **$bindable()**: Two-way binding for component values
- **Optional calling**: Use `callback?.()` to safely call optional callbacks
- **Event bubbling**: Events naturally bubble up to parent components

---

## 8. Component Bindings & Programmatic Interaction

### What is This?

Learn how to bind to component instances and call methods or access properties programmatically.

**Real-World Scenario:** You need to focus an input, trigger a modal to open, or call a component's internal method from a parent.

**What it does:** Shows component bindings and programmatic control.

### 📁 Files to Create

Create:

- `src/lib/components/Modal.svelte`
- `src/routes/section-2/bindings-demo/+page.svelte`

```svelte
<!-- Modal.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		title: string;
		children: Snippet;
	}

	let { title, children }: Props = $props();

	let isOpen = $state(false);

	export function open() {
		isOpen = true;
	}

	export function close() {
		isOpen = false;
	}

	export function toggle() {
		isOpen = !isOpen;
	}
</script>

{#if isOpen}
	<div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4" onclick={close}>
		<div
			class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 max-w-2xl w-full"
			onclick={(e) => e.stopPropagation()}
		>
			<div class="flex justify-between items-center mb-4">
				<h2 class="m-0 text-white text-2xl font-bold">{title}</h2>
				<button
					onclick={close}
					class="bg-transparent border-none text-gray-400 text-2xl cursor-pointer hover:text-white"
				>
					✕
				</button>
			</div>
			<div>{@render children()}</div>
		</div>
	</div>
{/if}
```

```svelte
<!-- bindings-demo/+page.svelte -->
<script lang="ts">
	import Modal from '$lib/components/Modal.svelte';

	// Type the component refs with the exported methods
	type ModalRef = { open: () => void; close: () => void; toggle: () => void };

	let modal1: ModalRef;
	let modal2: ModalRef;
	let modal3: ModalRef;

	let inputRef: HTMLInputElement;

	function focusInput() {
		inputRef?.focus();
	}

	function openAll() {
		modal1?.open();
		modal2?.open();
		modal3?.open();
	}

	function closeAll() {
		modal1?.close();
		modal2?.close();
		modal3?.close();
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		🔗 Component Bindings & Programmatic Control
	</h1>

	<div class="max-w-4xl mx-auto space-y-6">
		<!-- Component Bindings -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Modal Component Bindings</h2>
			<div class="flex gap-3 flex-wrap">
				<button
					onclick={() => modal1.open()}
					class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-blue-300"
				>
					Open Modal 1
				</button>
				<button
					onclick={() => modal2.toggle()}
					class="px-6 py-3 bg-green-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-green-300"
				>
					Toggle Modal 2
				</button>
				<button
					onclick={() => modal3.open()}
					class="px-6 py-3 bg-purple-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-purple-300"
				>
					Open Modal 3
				</button>
			</div>
			<div class="flex gap-3 mt-4">
				<button
					onclick={openAll}
					class="px-6 py-3 bg-orange-500 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-orange-400"
				>
					Open All
				</button>
				<button
					onclick={closeAll}
					class="px-6 py-3 bg-red-400 text-white border-none rounded-lg font-bold cursor-pointer hover:bg-red-300"
				>
					Close All
				</button>
			</div>
		</div>

		<!-- DOM Element Binding -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">DOM Element Binding</h2>
			<div class="flex gap-3 items-center">
				<input
					bind:this={inputRef}
					type="text"
					placeholder="This input can be focused programmatically"
					class="flex-1 px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white"
				/>
				<button
					onclick={focusInput}
					class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-blue-300"
				>
					Focus Input
				</button>
			</div>
		</div>

		<!-- Explanation -->
		<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-6">
			<h3 class="m-0 mb-3 text-blue-400 font-bold">💡 Component Bindings</h3>
			<ul class="m-0 pl-5 space-y-2">
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded">bind:this={'{variable}'}</code> binds component/element
					instance
				</li>
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded">export function</code> in child makes methods callable
					from parent
				</li>
				<li>Use optional chaining <code>?.</code> to safely call methods</li>
				<li>Works with both components and DOM elements</li>
				<li>Useful for imperatively controlling child components</li>
			</ul>
		</div>
	</div>
</div>

<!-- Modals -->
<Modal bind:this={modal1} title="Modal 1">
	<p class="m-0 mb-4">This is the content of Modal 1.</p>
	<button
		onclick={() => modal1.close()}
		class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-blue-300"
	>
		Close
	</button>
</Modal>

<Modal bind:this={modal2} title="Modal 2">
	<p class="m-0 mb-4">This modal was toggled!</p>
	<button
		onclick={() => modal2.close()}
		class="px-6 py-3 bg-green-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-green-300"
	>
		Close
	</button>
</Modal>

<Modal bind:this={modal3} title="Modal 3">
	<p class="m-0 mb-4">Modal 3 content here.</p>
	<button
		onclick={() => modal3.close()}
		class="px-6 py-3 bg-purple-400 text-black border-none rounded-lg font-bold cursor-pointer hover:bg-purple-300"
	>
		Close
	</button>
</Modal>
```

**Key Concepts:**

- **bind:this**: Bind component or element instance to a variable
- **export function**: Make component methods callable from parent
- **Programmatic control**: Call methods imperatively (open(), close(), etc.)
- **DOM references**: Access DOM elements directly
- **Optional chaining**: Safely call methods with `?.`

---

## 9. Dynamic Components with svelte:element

### What is This?

Render different HTML elements dynamically based on state using `<svelte:element>`.

**Real-World Scenario:** You're building a flexible heading component that can render as h1, h2, h3, etc., or a button that can be rendered as a button, a, or div based on props.

**What it does:** Shows how to dynamically choose which HTML element to render.

### 📁 Files to Create

Create:

- `src/lib/components/Heading.svelte`
- `src/routes/section-2/dynamic-demo/+page.svelte`

```svelte
<!-- Heading.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		level?: 1 | 2 | 3 | 4 | 5 | 6;
		color?: string;
		children: Snippet;
	}

	let { level = 1, color = 'white', children }: Props = $props();

	const tag = $derived(`h${level}` as 'h1' | 'h2' | 'h3' | 'h4' | 'h5' | 'h6');

	const sizes = {
		h1: 'text-4xl',
		h2: 'text-3xl',
		h3: 'text-2xl',
		h4: 'text-xl',
		h5: 'text-lg',
		h6: 'text-base'
	};
</script>

<svelte:element this={tag} class="font-bold m-0 {sizes[tag]}" style:color>
	{@render children()}
</svelte:element>
```

```svelte
<!-- dynamic-demo/+page.svelte -->
<script lang="ts">
	import Heading from '$lib/components/Heading.svelte';

	let headingLevel = $state<1 | 2 | 3 | 4 | 5 | 6>(1);
	let headingColor = $state('#4a9eff');
	let elementType = $state<'button' | 'a' | 'div'>('button');

	let dynamicElements = $state([
		{ tag: 'h1', content: 'Heading 1' },
		{ tag: 'h2', content: 'Heading 2' },
		{ tag: 'p', content: 'Paragraph' },
		{ tag: 'strong', content: 'Strong text' },
		{ tag: 'em', content: 'Emphasized text' }
	]);
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		🔄 Dynamic Components with svelte:element
	</h1>

	<div class="max-w-4xl mx-auto space-y-6">
		<!-- Dynamic Heading Component -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Dynamic Heading Component</h2>
			<div class="bg-gray-900 p-6 rounded-lg mb-4">
				<Heading level={headingLevel} color={headingColor}>
					This is a level {headingLevel} heading
				</Heading>
			</div>
			<div class="space-y-4">
				<div>
					<label class="block mb-2 text-sm">Level: {headingLevel}</label>
					<input
						type="range"
						bind:value={headingLevel}
						min="1"
						max="6"
						step="1"
						class="w-full accent-blue-400"
					/>
				</div>
				<div>
					<label class="block mb-2 text-sm">Color</label>
					<input type="color" bind:value={headingColor} class="w-full h-12 cursor-pointer" />
				</div>
			</div>
		</div>

		<!-- Dynamic Button/Link/Div -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Dynamic Element Type</h2>
			<div class="flex gap-3 mb-4">
				<button
					onclick={() => (elementType = 'button')}
					class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer"
					class:bg-blue-400={elementType === 'button'}
					class:text-black={elementType === 'button'}
					class:bg-gray-700={elementType !== 'button'}
					class:text-white={elementType !== 'button'}
				>
					Button
				</button>
				<button
					onclick={() => (elementType = 'a')}
					class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer"
					class:bg-blue-400={elementType === 'a'}
					class:text-black={elementType === 'a'}
					class:bg-gray-700={elementType !== 'a'}
					class:text-white={elementType !== 'a'}
				>
					Link
				</button>
				<button
					onclick={() => (elementType = 'div')}
					class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer"
					class:bg-blue-400={elementType === 'div'}
					class:text-black={elementType === 'div'}
					class:bg-gray-700={elementType !== 'div'}
					class:text-white={elementType !== 'div'}
				>
					Div
				</button>
			</div>
			<div class="bg-gray-900 p-6 rounded-lg">
				<svelte:element
					this={elementType}
					class="inline-block px-6 py-3 bg-green-400 text-black font-bold rounded-lg cursor-pointer hover:bg-green-300"
					href={elementType === 'a' ? '#' : undefined}
				>
					I'm a &lt;{elementType}&gt; element!
				</svelte:element>
			</div>
		</div>

		<!-- List of Dynamic Elements -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">List of Dynamic Elements</h2>
			<div class="space-y-3">
				{#each dynamicElements as { tag, content }}
					<div class="bg-gray-900 p-4 rounded-lg flex items-center gap-4">
						<code class="text-blue-400 text-sm font-mono">&lt;{tag}&gt;</code>
						<svelte:element this={tag} class="flex-1">
							{content}
						</svelte:element>
					</div>
				{/each}
			</div>
		</div>

		<!-- Explanation -->
		<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-6">
			<h3 class="m-0 mb-3 text-blue-400 font-bold">💡 svelte:element</h3>
			<ul class="m-0 pl-5 space-y-2">
				<li>
					<code class="bg-gray-900 px-2 py-1 rounded">&lt;svelte:element this={'{tag}'}&gt;</code>
					renders dynamic HTML elements
				</li>
				<li>Tag must be a valid HTML element name (string)</li>
				<li>Can apply classes, styles, and attributes normally</li>
				<li>Useful for flexible, reusable components (headings, links, buttons)</li>
				<li>Type-safe with TypeScript using union types</li>
			</ul>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **svelte:element**: Dynamically render different HTML elements
- **this={tag}**: Specify element type with a string variable
- **Type safety**: Use union types for valid element names
- **Flexibility**: Build components that adapt to different contexts
- **Semantic HTML**: Choose appropriate elements dynamically

---

## 10. 🚀 End-of-Section Project: Component Library Dashboard

### Project Overview

Build a **reusable component library dashboard** that showcases all the concepts from this section. This real-world project combines snippets, props, events, dynamic styling, and programmatic component control.

**What You'll Build:**

- A **Card** component with snippet slots for header, body, and footer
- A **Button** component with variants, sizes, and loading states
- A **Input** component with validation and error display
- A **Modal** component with programmatic open/close
- A **Dashboard** that showcases all components together

### 📁 Files to Create

**1. Card Component** - `src/lib/components/Card.svelte`

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		variant?: 'default' | 'elevated' | 'outlined';
		header?: Snippet;
		children: Snippet;
		footer?: Snippet;
	}

	let { variant = 'default', header, children, footer }: Props = $props();
</script>

<div
	class="rounded-xl overflow-hidden transition-all duration-200"
	class:bg-gray-800={variant === 'default'}
	class:border-gray-700={variant === 'default'}
	class:border-2={variant !== 'elevated'}
	class:bg-gray-800={variant === 'elevated'}
	class:shadow-xl={variant === 'elevated'}
	class:hover:shadow-2xl={variant === 'elevated'}
	class:hover:-translate-y-1={variant === 'elevated'}
	class:bg-transparent={variant === 'outlined'}
	class:border-gray-600={variant === 'outlined'}
>
	{#if header}
		<div class="p-4 border-b border-gray-700 bg-gray-900/50">
			{@render header()}
		</div>
	{/if}

	<div class="p-6">
		{@render children()}
	</div>

	{#if footer}
		<div class="p-4 border-t border-gray-700 bg-gray-900/50">
			{@render footer()}
		</div>
	{/if}
</div>
```

**2. Button Component** - `src/lib/components/Button.svelte`

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		variant?: 'primary' | 'secondary' | 'danger' | 'ghost';
		size?: 'sm' | 'md' | 'lg';
		loading?: boolean;
		disabled?: boolean;
		onclick?: () => void;
		children: Snippet;
	}

	let {
		variant = 'primary',
		size = 'md',
		loading = false,
		disabled = false,
		onclick,
		children
	}: Props = $props();

	const isDisabled = $derived(disabled || loading);
</script>

<button
	class="font-bold rounded-lg transition-all duration-200 flex items-center justify-center gap-2"
	class:px-4={size === 'sm'}
	class:py-2={size === 'sm'}
	class:text-sm={size === 'sm'}
	class:px-6={size === 'md'}
	class:py-3={size === 'md'}
	class:text-base={size === 'md'}
	class:px-8={size === 'lg'}
	class:py-4={size === 'lg'}
	class:text-lg={size === 'lg'}
	class:bg-blue-500={variant === 'primary'}
	class:hover:bg-blue-400={variant === 'primary' && !isDisabled}
	class:text-white={variant === 'primary'}
	class:bg-gray-600={variant === 'secondary'}
	class:hover:bg-gray-500={variant === 'secondary' && !isDisabled}
	class:text-white={variant === 'secondary'}
	class:bg-red-500={variant === 'danger'}
	class:hover:bg-red-400={variant === 'danger' && !isDisabled}
	class:text-white={variant === 'danger'}
	class:bg-transparent={variant === 'ghost'}
	class:hover:bg-gray-700={variant === 'ghost' && !isDisabled}
	class:text-gray-300={variant === 'ghost'}
	class:opacity-50={isDisabled}
	class:cursor-not-allowed={isDisabled}
	class:cursor-pointer={!isDisabled}
	disabled={isDisabled}
	{onclick}
>
	{#if loading}
		<div class="w-4 h-4 border-2 border-current border-t-transparent rounded-full animate-spin"></div>
	{/if}
	{@render children()}
</button>
```

**3. Input Component** - `src/lib/components/Input.svelte`

```svelte
<script lang="ts">
	interface Props {
		type?: 'text' | 'email' | 'password' | 'number';
		label?: string;
		placeholder?: string;
		value?: string;
		error?: string;
		required?: boolean;
		oninput?: (value: string) => void;
		onvalidate?: (isValid: boolean) => void;
	}

	let {
		type = 'text',
		label,
		placeholder = '',
		value = $bindable(''),
		error = '',
		required = false,
		oninput,
		onvalidate
	}: Props = $props();

	function handleInput(e: Event) {
		const target = e.target as HTMLInputElement;
		value = target.value;
		oninput?.(value);

		// Basic validation
		const isValid = !required || value.trim().length > 0;
		onvalidate?.(isValid);
	}
</script>

<div class="flex flex-col gap-2">
	{#if label}
		<label class="text-sm font-semibold text-gray-300">
			{label}
			{#if required}<span class="text-red-400">*</span>{/if}
		</label>
	{/if}

	<input
		{type}
		{placeholder}
		{value}
		oninput={handleInput}
		class="px-4 py-3 rounded-lg bg-gray-800 border-2 text-white transition-all duration-200 focus:outline-none"
		class:border-gray-600={!error}
		class:focus:border-blue-400={!error}
		class:border-red-400={error}
		class:focus:border-red-400={error}
	/>

	{#if error}
		<span class="text-sm text-red-400">{error}</span>
	{/if}
</div>
```

**4. Modal Component** - `src/lib/components/Modal.svelte`

```svelte
<script lang="ts">
	import type { Snippet } from 'svelte';

	interface Props {
		title: string;
		isOpen?: boolean;
		children: Snippet;
		footer?: Snippet;
	}

	let { title, isOpen = $bindable(false), children, footer }: Props = $props();

	export function open() {
		isOpen = true;
	}

	export function close() {
		isOpen = false;
	}

	export function toggle() {
		isOpen = !isOpen;
	}
</script>

{#if isOpen}
	<div
		class="fixed inset-0 bg-black/70 flex items-center justify-center z-50 backdrop-blur-sm"
		onclick={close}
	>
		<div
			class="bg-gray-800 border-2 border-gray-700 rounded-2xl max-w-lg w-11/12 shadow-2xl"
			onclick={(e) => e.stopPropagation()}
		>
			<div class="flex justify-between items-center p-6 border-b border-gray-700">
				<h2 class="text-xl font-bold text-white">{title}</h2>
				<button
					class="text-gray-400 hover:text-white text-2xl transition-colors"
					onclick={close}
				>
					✕
				</button>
			</div>

			<div class="p-6 text-gray-200">
				{@render children()}
			</div>

			{#if footer}
				<div class="p-6 border-t border-gray-700 flex gap-3 justify-end">
					{@render footer()}
				</div>
			{/if}
		</div>
	</div>
{/if}
```

**5. Dashboard Page** - `src/routes/component-library/+page.svelte`

```svelte
<script lang="ts">
	import Card from '$lib/components/Card.svelte';
	import Button from '$lib/components/Button.svelte';
	import Input from '$lib/components/Input.svelte';
	import Modal from '$lib/components/Modal.svelte';

	// Form state
	let name = $state('');
	let email = $state('');
	let formError = $state('');
	let isSubmitting = $state(false);

	// Modal reference
	let confirmModal: Modal;

	async function handleSubmit() {
		if (!name || !email) {
			formError = 'Please fill in all fields';
			return;
		}

		isSubmitting = true;
		// Simulate API call
		await new Promise((r) => setTimeout(r, 1500));
		isSubmitting = false;
		confirmModal.open();
	}

	function resetForm() {
		name = '';
		email = '';
		formError = '';
		confirmModal.close();
	}
</script>

<div class="bg-gray-900 min-h-screen p-8 text-gray-200">
	<div class="max-w-6xl mx-auto">
		<h1 class="text-4xl font-bold text-blue-400 mb-2">Component Library</h1>
		<p class="text-gray-400 mb-10">A showcase of reusable Svelte components</p>

		<div class="grid md:grid-cols-2 gap-8">
			<!-- Buttons Section -->
			<Card variant="elevated">
				{#snippet header()}
					<h2 class="text-lg font-bold text-white">Buttons</h2>
				{/snippet}

				<div class="flex flex-wrap gap-3">
					<Button variant="primary">Primary</Button>
					<Button variant="secondary">Secondary</Button>
					<Button variant="danger">Danger</Button>
					<Button variant="ghost">Ghost</Button>
				</div>

				<div class="flex flex-wrap gap-3 mt-4">
					<Button size="sm">Small</Button>
					<Button size="md">Medium</Button>
					<Button size="lg">Large</Button>
				</div>

				<div class="mt-4">
					<Button loading={true}>Loading...</Button>
				</div>
			</Card>

			<!-- Form Section -->
			<Card variant="elevated">
				{#snippet header()}
					<h2 class="text-lg font-bold text-white">Contact Form</h2>
				{/snippet}

				<div class="flex flex-col gap-4">
					<Input
						label="Name"
						placeholder="Enter your name"
						bind:value={name}
						required
					/>
					<Input
						type="email"
						label="Email"
						placeholder="you@example.com"
						bind:value={email}
						error={formError}
						required
					/>
					<Button onclick={handleSubmit} loading={isSubmitting}>
						Submit Form
					</Button>
				</div>
			</Card>

			<!-- Card Variants -->
			<Card variant="outlined">
				{#snippet header()}
					<h2 class="text-lg font-bold text-white">Outlined Card</h2>
				{/snippet}

				<p class="text-gray-400">
					This card uses the outlined variant with a transparent background.
				</p>

				{#snippet footer()}
					<Button variant="ghost" size="sm">Learn More</Button>
				{/snippet}
			</Card>

			<!-- Modal Trigger -->
			<Card>
				{#snippet header()}
					<h2 class="text-lg font-bold text-white">Modal Demo</h2>
				{/snippet}

				<p class="text-gray-400 mb-4">
					Click the button to open a modal with programmatic control.
				</p>
				<Button onclick={() => confirmModal.open()}>
					Open Modal
				</Button>
			</Card>
		</div>
	</div>
</div>

<!-- Confirmation Modal -->
<Modal bind:this={confirmModal} title="Success!">
	<p>Your form has been submitted successfully.</p>
	<p class="text-gray-400 mt-2">Name: {name}</p>
	<p class="text-gray-400">Email: {email}</p>

	{#snippet footer()}
		<Button variant="secondary" onclick={() => confirmModal.close()}>
			Close
		</Button>
		<Button onclick={resetForm}>
			Submit Another
		</Button>
	{/snippet}
</Modal>
```

### What This Project Demonstrates

| Concept              | Implementation                      |
| -------------------- | ----------------------------------- |
| **Snippets**         | Card header, body, footer slots     |
| **Snippet Props**    | Flexible component composition      |
| **class: directive** | Dynamic styling based on variants   |
| **Spread Props**     | Button and Input forward attributes |
| **style: directive** | Dynamic inline styles               |
| **Event Forwarding** | onclick, oninput handlers           |
| **Custom Events**    | onvalidate callback                 |
| **bind:this**        | Programmatic modal control          |
| **$bindable**        | Two-way binding for form values     |

---

## 📝 Key Takeaways

✅ **Snippets** are reusable template fragments  
✅ **Snippet props** make components highly flexible  
✅ **Snippet arguments** pass data to reusable markup  
✅ **SASS** provides cleaner, nested styling  
✅ **class: directive** for conditional classes  
✅ **Spread props** (`{...rest}`) forward attributes  
✅ **style: directive** for dynamic inline styles  
✅ **CSS variables** as props for themeable components  
✅ **Event forwarding** via props (oninput, onfocus, etc.)  
✅ **Custom events** with callback props  
✅ **Component bindings** (`bind:this`) for programmatic control  
✅ **svelte:element** for dynamic HTML elements

### 💡 Best Practices for Component Architecture

**Event Handling Patterns:**

- ✅ **Use event forwarding** for standard events (oninput, onclick, onfocus)
- ✅ **Create custom callbacks** for domain-specific events (onvalidate, onsave)
- ✅ **Optional callbacks** with `callback?.()` - prevents errors if not provided
- ❌ **Avoid $inspect()** in reusable components - it's for debugging only

**Component Organization:**

```
src/lib/components/
├── forms/               # Form-related components
│   ├── Input.svelte
│   ├── Select.svelte
│   └── Form.svelte
├── layout/              # Layout components
│   ├── Modal.svelte
│   ├── Card.svelte
│   └── Sidebar.svelte
└── ui/                  # Basic UI elements
    ├── Button.svelte
    ├── Badge.svelte
    └── Alert.svelte
```

### ⚠️ Common Mistakes

1. **Forgetting to export functions** for programmatic control:

```typescript
// ❌ Bad - function not accessible
function open() {
  isOpen = true;
}

// ✅ Good - exported for external use
export function open() {
  isOpen = true;
}
```

2. **Over-using bind:this** - only needed for programmatic control:

```svelte
<!-- ❌ Bad - unnecessary binding -->
<Modal bind:this={modal} />

<!-- ✅ Good - only bind when calling methods -->
<Modal bind:this={modal} />
<button onclick={() => modal.open()}>Open</button>
```

3. **Not forwarding rest props** - breaks composability:

```svelte
<!-- ❌ Bad - loses attributes -->
<input {type} {value} />

<!-- ✅ Good - forwards all props -->
<input {type} {value} {...rest} />
```

### ⚡ Performance Tips

- **CSS Variables** > Inline Styles - Use `style:` for dynamic values, CSS vars for themes
- **Conditional Classes** - Use `class:` directive, not ternaries in class strings
- **Snippet Caching** - Snippets are efficient, don't avoid them for performance

---

## 🚀 Next Steps

After mastering these component patterns, you'll be ready for:

- **Section 3**: Deep state management with nested objects and arrays
- **Section 4**: Advanced patterns like reactive classes and API integration
- Building sophisticated, production-ready component libraries
