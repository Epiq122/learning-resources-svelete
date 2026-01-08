# Section 7: Actions, Special Elements & Advanced Bindings Study Guide

**Complete lesson plan for learning Svelte 5 actions, special elements, and advanced component patterns**

---

## 📚 Learning Objectives

By the end of this section, you will:

- ✅ Create custom actions for reusable DOM behavior
- ✅ Integrate third-party libraries using actions
- ✅ Use special elements for global events and metadata
- ✅ Handle errors gracefully with error boundaries
- ✅ Bind to element dimensions and media properties
- ✅ Share data across component instances with module scripts
- ✅ Build real-world interactive features

---

## Table of Contents

1. [Introduction to Actions](#1-introduction-to-actions)
2. [Longpress Action](#2-longpress-action)
3. [Tippy.js Tooltips](#3-tippyjs-tooltips)
4. [Click Outside Detection](#4-click-outside-detection)
5. [Special Elements](#5-special-elements)
6. [Error Boundaries](#6-error-boundaries)
7. [Binding Dimensions](#7-binding-dimensions)
8. [Media Element Bindings](#8-media-element-bindings)
9. [Script Module Context](#9-script-module-context)
10. [Shared Component State](#10-shared-component-state)
11. [Multi-Video Controller](#11-multi-video-controller)

---

## 1. Introduction to Actions

### What are Actions?

Actions are functions that run when an element is **mounted** and **unmounted**. They're perfect for:

- Adding third-party library integrations
- Custom event listeners
- DOM manipulations that don't fit in Svelte's reactive model
- Reusable behaviors across multiple components

**Real-World Scenario:** You need to add tooltips to 20+ buttons across your app. Instead of copy-pasting tooltip code everywhere, create a `tooltip` action and reuse it!

### 📁 Files to Create

**Reusable Actions** (best practice!):

- `src/lib/actions/autofocus.ts` - Simple autofocus action
- `src/lib/actions/longpress.ts` - Longpress detection action
- `src/lib/actions/clickOutside.ts` - Click outside detection
- `src/lib/actions/tooltip.ts` - Tooltip integration

**Demo Page:**

- `src/routes/actions-demo/+page.svelte` - Shows all actions in use

**Best Practices:**

- Create actions as standalone functions in `src/lib/actions/`
- One action per file for easy imports
- Export typed action functions
- Actions are highly reusable across your app

### Action Pattern

```typescript
function actionName(node: HTMLElement, parameters?: any) {
  // Setup code runs when element is added to DOM
  // 'node' is the element the action is attached to
  // 'parameters' are any values passed to the action

  return {
    // Optional: Update when parameters change
    // Called whenever reactive parameters passed to action change
    update(newParameters: any) {
      // Update logic - modify behavior based on new parameters
    },

    // Optional: Cleanup when element is removed from DOM
    // ALWAYS remove event listeners here to prevent memory leaks!
    destroy() {
      // Cleanup logic (remove listeners, destroy third-party instances)
    },
  };
}
```

### Real-World Example: Auto-focus Action

**What it does:** Automatically focuses an input when it's added to the page. Useful for modals, search bars, or forms.

```svelte
<script lang="ts">
	function autofocus(node: HTMLInputElement) {
		// Focus the input when it's mounted
		node.focus();

		// No cleanup needed
	}
</script>

<div class="bg-gray-800 p-6 rounded-xl border border-gray-700 max-w-lg">
	<h2 class="m-0 mb-4 text-blue-400 text-2xl">Quick Search</h2>
	<!-- Input automatically focuses when modal opens -->
	<input
		type="text"
		placeholder="Search products..."
		use:autofocus
		class="w-full px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white text-base transition-colors focus:outline-none focus:border-blue-400"
	/>
</div>
```

---

## 2. Longpress Action

### Real-World Scenario: Delete Confirmation

Instead of showing a confirmation dialog, use longpress to delete items. Common in mobile apps!

**What it does:** Detects when user holds down on a button for a specified duration, then fires a custom event.

```svelte
<script lang="ts">
	function longpress(node: HTMLElement, duration: number = 500) {
		let timer: number;

		// Start timer when mouse/touch is pressed down
		function handleMouseDown() {
			// Set timeout to dispatch event after duration
			timer = setTimeout(() => {
				// Dispatch custom 'longpress' event that component can listen to
				node.dispatchEvent(new CustomEvent('longpress'));
			}, duration);
		}

		// Cancel timer if user releases before duration completes
		function handleMouseUp() {
			clearTimeout(timer);
		}

		// Add listeners
		node.addEventListener('mousedown', handleMouseDown);
		node.addEventListener('mouseup', handleMouseUp);
		node.addEventListener('mouseleave', handleMouseUp); // Cancel if mouse leaves

		// Also support touch events for mobile
		node.addEventListener('touchstart', handleMouseDown);
		node.addEventListener('touchend', handleMouseUp);

		return {
			destroy() {
				// Cleanup all listeners
				node.removeEventListener('mousedown', handleMouseDown);
				node.removeEventListener('mouseup', handleMouseUp);
				node.removeEventListener('mouseleave', handleMouseUp);
				node.removeEventListener('touchstart', handleMouseDown);
				node.removeEventListener('touchend', handleMouseUp);
			}
		};
	}

	interface TodoItem {
		id: number;
		text: string;
	}

	let todos = $state<TodoItem[]>([
		{ id: 1, text: 'Buy groceries' },
		{ id: 2, text: 'Finish project' },
		{ id: 3, text: 'Call mom' }
	]);

	let deletingId = $state<number | null>(null);

	function handleLongpress(id: number) {
		deletingId = id;
		// Visual feedback for 300ms then delete
		setTimeout(() => {
			todos = todos.filter((t) => t.id !== id);
			deletingId = null;
		}, 300);
	}
</script>

<div class="bg-gray-800 p-6 rounded-xl border border-gray-700 max-w-sm">
	<h2 class="m-0 mb-2 text-blue-400 text-2xl">Todo List</h2>
	<p class="m-0 mb-5 text-gray-500 text-sm">💡 Hold any item to delete it</p>

	{#each todos as todo (todo.id)}
		<div
			class="bg-gray-900 px-4 py-4 mb-2 rounded-lg border-2 border-gray-700 cursor-pointer transition-all flex justify-between items-center select-none hover:border-blue-400"
			class:border-red-400={deletingId === todo.id}
			class:bg-red-900/30={deletingId === todo.id}
			class:animate-shake={deletingId === todo.id}
			use:longpress={800}
			onlongpress={() => handleLongpress(todo.id)}
		>
			<span class="text-white text-base">{todo.text}</span>
			{#if deletingId === todo.id}
				<span class="text-red-400 text-xs font-semibold">Deleting...</span>
			{/if}
		</div>
	{/each}
</div>

<style>
	@keyframes shake {
		0%,
		100% {
			transform: translateX(0);
		}
		25% {
			transform: translateX(-4px);
		}
		75% {
			transform: translateX(4px);
		}
	}
</style>
```

**Key Concepts:**

- Custom events with `dispatchEvent(new CustomEvent('longpress'))`
- Cleanup is **critical** - always remove listeners in `destroy()`
- Support both mouse and touch events for mobile
- Use `setTimeout` for timing
- Visual feedback improves UX

---

## 3. Tippy.js Tooltips

### Real-World Scenario: Help System

Add helpful tooltips throughout your app to guide users without cluttering the UI.

**Installation:**

```bash
npm install tippy.js
```

**What it does:** Wraps the Tippy.js library in a reusable action. Add tooltips to any element!

```svelte
<script lang="ts">
	import tippy, { type Instance, type Props } from 'tippy.js';
	import 'tippy.js/dist/tippy.css'; // Default styles
	import 'tippy.js/themes/light.css'; // Light theme

	function tooltip(node: HTMLElement, options: Partial<Props>) {
		// Create tippy instance attached to this element
		// Tippy.js handles all tooltip positioning and display
		const instance: Instance = tippy(node, {
			...options,
			theme: 'custom' // Our custom theme defined in CSS
		});

		return {
			// Update tooltip content/options when reactive parameters change
			update(newOptions: Partial<Props>) {
				// setProps updates the tooltip without recreating it
				instance.setProps(newOptions);
			},
			// Clean up when element is removed from DOM
			destroy() {
				// CRITICAL: Destroy tippy instance to prevent memory leaks
				instance.destroy();
			}
		};
	}

	interface FormData {
		username: string;
		email: string;
		password: string;
	}

	let formData = $state<FormData>({
		username: '',
		email: '',
		password: ''
	});

	function handleSubmit() {
		console.log('Submitting:', formData);
	}
</script>

<form class="bg-gray-800 p-8 rounded-xl border border-gray-700 max-w-md" onsubmit={handleSubmit}>
	<h2 class="m-0 mb-6 text-blue-400 text-3xl">Create Account</h2>

	<div class="mb-5">
		<label for="username" class="flex items-center gap-2 text-gray-300 text-sm font-semibold mb-2">
			Username
			<span
				class="cursor-help text-base opacity-70 transition-opacity hover:opacity-100"
				use:tooltip={{
					content: 'Choose a unique username (3-20 characters)',
					placement: 'right'
				}}
			>
				ℹ️
			</span>
		</label>
		<input
			id="username"
			type="text"
			bind:value={formData.username}
			placeholder="johndoe"
			class="w-full px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white text-base transition-colors focus:outline-none focus:border-blue-400"
		/>
	</div>

	<div class="mb-5">
		<label for="email" class="flex items-center gap-2 text-gray-300 text-sm font-semibold mb-2">
			Email
			<span
				class="cursor-help text-base opacity-70 transition-opacity hover:opacity-100"
				use:tooltip={{
					content: "We'll send a verification email",
					placement: 'right'
				}}
			>
				ℹ️
			</span>
		</label>
		<input
			id="email"
			type="email"
			bind:value={formData.email}
			placeholder="john@example.com"
			class="w-full px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white text-base transition-colors focus:outline-none focus:border-blue-500"
		/>
	</div>

	<div class="mb-5">
		<label for="password" class="flex items-center gap-2 text-gray-300 text-sm font-semibold mb-2">
			Password
			<span
				class="cursor-help text-base opacity-70 transition-opacity hover:opacity-100"
				use:tooltip={{
					content: 'Must be at least 8 characters with numbers and symbols',
					placement: 'right',
					maxWidth: 200
				}}
			>
				ℹ️
			</span>
		</label>
		<input
			id="password"
			type="password"
			bind:value={formData.password}
			placeholder="••••••••"
			class="w-full px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white text-base transition-colors focus:outline-none focus:border-blue-500"
		/>
	</div>

	<button
		type="submit"
		class="w-full bg-blue-500 text-black border-none py-3.5 rounded-lg text-base font-bold cursor-pointer transition-all mt-2 hover:bg-blue-400 hover:-translate-y-0.5"
		use:tooltip={{
			content: 'Click to create your account',
			placement: 'bottom'
		}}
	>
		Sign Up
	</button>
</form>

<style>
	/* Custom Tippy theme */
	:global(.tippy-box[data-theme='custom']) {
		background: #2a2a2a;
		border: 1px solid #4a9eff;
		color: #fff;
		font-size: 13px;
		padding: 8px 12px;
		border-radius: 6px;
		box-shadow: 0 4px 12px rgba(0, 0, 0, 0.5);
	}

	:global(.tippy-box[data-theme='custom'][data-placement^='top'] > .tippy-arrow::before) {
		border-top-color: #4a9eff;
	}

	:global(.tippy-box[data-theme='custom'][data-placement^='bottom'] > .tippy-arrow::before) {
		border-bottom-color: #4a9eff;
	}

	:global(.tippy-box[data-theme='custom'][data-placement^='left'] > .tippy-arrow::before) {
		border-left-color: #4a9eff;
	}

	:global(.tippy-box[data-theme='custom'][data-placement^='right'] > .tippy-arrow::before) {
		border-right-color: #4a9eff;
	}
</style>
```

**Key Concepts:**

- Actions perfect for third-party library integration
- `update()` method lets you change tooltip content dynamically
- Always destroy library instances in `destroy()`
- Use `:global()` for library-specific styles
- Placement options: `top`, `bottom`, `left`, `right`

---

## 4. Click Outside Detection

### Real-World Scenario: Dropdown Menus

Close dropdowns/modals when user clicks outside. Essential for good UX!

**What it does:** Detects clicks outside an element and dispatches a custom event.

```svelte
<script lang="ts">
	function clickOutside(node: HTMLElement) {
		function handleClick(event: MouseEvent) {
			// Check if the clicked element is outside our element
			// node.contains() returns false if click is outside
			if (node && !node.contains(event.target as Node)) {
				// Dispatch custom event that component can listen to
				node.dispatchEvent(new CustomEvent('clickoutside'));
			}
		}

		// Use capture phase (true) to catch events before they bubble
		// This ensures we catch clicks even on elements that stop propagation
		document.addEventListener('click', handleClick, true);

		return {
			destroy() {
				document.removeEventListener('click', handleClick, true);
			}
		};
	}

	interface User {
		name: string;
		email: string;
		avatar: string;
	}

	let user: User = {
		name: 'Sarah Johnson',
		email: 'sarah@example.com',
		avatar: '👩‍💼'
	};

	let isOpen = $state(false);

	function logout() {
		console.log('Logging out...');
		isOpen = false;
	}
</script>

<div class="relative">
	<button
		class="bg-gray-800 border-2 border-gray-700 text-white px-4 py-2.5 rounded-lg flex items-center gap-3 cursor-pointer transition-all hover:border-blue-500"
		onclick={() => (isOpen = !isOpen)}
	>
		<span class="text-2xl">{user.avatar}</span>
		<span class="font-semibold text-sm">{user.name}</span>
		<span class="text-xs transition-transform ml-1" class:rotate-180={isOpen}>▼</span>
	</button>

	{#if isOpen}
		<div
			class="absolute top-full mt-2 right-0 bg-gray-800 border border-gray-700 rounded-xl min-w-72 shadow-2xl animate-slideDown z-50"
			use:clickOutside
			onclickoutside={() => (isOpen = false)}
		>
			<div class="p-4 flex items-center gap-3">
				<span class="text-5xl">{user.avatar}</span>
				<div class="flex flex-col gap-1">
					<strong class="text-white text-base">{user.name}</strong>
					<span class="text-gray-500 text-sm">{user.email}</span>
				</div>
			</div>

			<div class="h-px bg-gray-700 my-2"></div>

			<button
				class="w-full bg-transparent border-none text-white px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-2.5 hover:bg-gray-900"
			>
				⚙️ Settings
			</button>
			<button
				class="w-full bg-transparent border-none text-white px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-2.5 hover:bg-gray-900"
			>
				🎨 Appearance
			</button>
			<button
				class="w-full bg-transparent border-none text-white px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-2.5 hover:bg-gray-900"
			>
				📊 Analytics
			</button>

			<div class="h-px bg-gray-700 my-2"></div>

			<button
				class="w-full bg-transparent border-none text-red-400 px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-2.5 hover:bg-red-900/30"
				onclick={logout}
			>
				🚪 Log Out
			</button>
		</div>
	{/if}
</div>

<style>
	@keyframes slideDown {
		from {
			opacity: 0;
			transform: translateY(-10px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
</style>
```

**Key Concepts:**

- Use `document.addEventListener` with capture phase (`true`)
- Check if clicked element is inside using `node.contains()`
- Dispatch custom `clickoutside` event
- Perfect for dropdowns, modals, popovers
- Always clean up document listeners

---

## 5. Special Elements

### Real-World Scenarios for Special Elements

Special elements let you interact with browser APIs that aren't regular DOM elements:

- **`<svelte:window>`**: Keyboard shortcuts, scroll tracking, resize handling
- **`<svelte:document>`**: Global click detection, key presses
- **`<svelte:head>`**: Dynamic page titles, meta tags (SEO!)
- **`<svelte:body>`**: Add classes, handle body events

### Complete Example: Multi-Page App with Analytics

**What it does:** Shows real-world use of all special elements in a dashboard app.

```svelte
<script lang="ts">
	interface Page {
		id: string;
		title: string;
		description: string;
	}

	const pages: Page[] = [
		{ id: 'home', title: 'Dashboard', description: 'Overview and analytics' },
		{ id: 'products', title: 'Products', description: 'Manage your inventory' },
		{ id: 'orders', title: 'Orders', description: 'View recent orders' },
		{ id: 'settings', title: 'Settings', description: 'Configure your account' }
	];

	let currentPage = $state(pages[0]);

	// Window bindings
	let scrollY = $state(0);
	let windowWidth = $state(0);
	let windowHeight = $state(0);

	// Keyboard shortcuts
	function handleKeydown(e: KeyboardEvent) {
		// Ctrl/Cmd + K = Quick search
		if ((e.ctrlKey || e.metaKey) && e.key === 'k') {
			e.preventDefault();
			console.log('Quick search triggered!');
		}

		// Ctrl/Cmd + 1-4 = Switch pages
		if ((e.ctrlKey || e.metaKey) && e.key >= '1' && e.key <= '4') {
			e.preventDefault();
			const index = parseInt(e.key) - 1;
			currentPage = pages[index];
		}
	}

	// Device type
	const isMobile = $derived(windowWidth < 768);
	const isScrolled = $derived(scrollY > 50);
</script>

<!-- Update page title dynamically -->
<svelte:head>
	<title>{currentPage.title} - My SaaS Dashboard</title>
	<meta name="description" content={currentPage.description} />
</svelte:head>

<!-- Listen to window events and bind dimensions -->
<svelte:window
	bind:scrollY
	bind:innerWidth={windowWidth}
	bind:innerHeight={windowHeight}
	onkeydown={handleKeydown}
/>

<!-- Add class to body for global styling -->
<svelte:body class:mobile={isMobile} class:scrolled={isScrolled} />

<div class="bg-gray-900 min-h-screen text-gray-200">
	<!-- Fixed header that changes when scrolled -->
	<header
		class="sticky top-0 bg-gray-800 border-b-2 border-gray-700 px-6 py-5 flex justify-between items-center z-50 transition-all"
		class:!py-3={isScrolled}
		class:bg-gray-800/95={isScrolled}
		class:backdrop-blur-sm={isScrolled}
		class:shadow-lg={isScrolled}
	>
		<h1 class="m-0 text-blue-400 text-2xl">My SaaS Dashboard</h1>
		<div class="flex gap-4">
			<span class="bg-gray-900 px-3 py-1.5 rounded-md text-sm font-semibold font-mono"
				>📱 {windowWidth}px</span
			>
			<span class="bg-gray-900 px-3 py-1.5 rounded-md text-sm font-semibold font-mono"
				>📏 {windowHeight}px</span
			>
			<span class="bg-gray-900 px-3 py-1.5 rounded-md text-sm font-semibold font-mono"
				>📜 {Math.round(scrollY)}px</span
			>
		</div>
	</header>

	<!-- Page navigation -->
	<nav class="bg-gray-800 px-6 py-4 flex gap-3 border-b border-gray-700">
		{#each pages as page, i}
			<button
				class="bg-transparent border-2 border-gray-700 text-gray-500 px-4 py-2.5 rounded-lg font-semibold cursor-pointer transition-all flex items-center gap-2 hover:border-blue-500 hover:text-white"
				class:bg-blue-500={currentPage.id === page.id}
				class:border-blue-500={currentPage.id === page.id}
				class:text-black={currentPage.id === page.id}
				onclick={() => (currentPage = page)}
			>
				{page.title}
				<span class="text-xs opacity-60 bg-black/30 px-1.5 py-0.5 rounded"
					>⌘{i + 1}</span
				>
			</button>
		{/each}
	</nav>

	<!-- Page content -->
	<main class="py-10 px-6 max-w-6xl mx-auto">
		<h2 class="m-0 mb-2 text-white text-3xl">{currentPage.title}</h2>
		<p class="m-0 mb-8 text-gray-500 text-lg">{currentPage.description}</p>

		<div class="grid grid-cols-[repeat(auto-fit,minmax(280px,1fr))] gap-5 mb-10">
			<div class="bg-gray-800 border border-gray-700 rounded-xl p-6">
				<h3 class="m-0 mb-4 text-blue-400 text-lg">Device Info</h3>
				<p class="my-2 text-gray-300 text-sm">Type: {isMobile ? '📱 Mobile' : '💻 Desktop'}</p>
				<p class="my-2 text-gray-300 text-sm">Width: {windowWidth}px</p>
				<p class="my-2 text-gray-300 text-sm">Height: {windowHeight}px</p>
			</div>

			<div class="bg-gray-800 border border-gray-700 rounded-xl p-6">
				<h3 class="m-0 mb-4 text-blue-400 text-lg">Scroll Position</h3>
				<p class="my-2 text-gray-300 text-sm">Y: {Math.round(scrollY)}px</p>
				<p class="my-2 text-gray-300 text-sm">Status: {isScrolled ? 'Scrolled' : 'Top of page'}</p>
			</div>

			<div class="bg-gray-800 border border-gray-700 rounded-xl p-6">
				<h3 class="m-0 mb-4 text-blue-400 text-lg">Keyboard Shortcuts</h3>
				<p class="my-2 text-gray-300 text-sm">⌘K - Quick search</p>
				<p class="my-2 text-gray-300 text-sm">⌘1-4 - Switch pages</p>
			</div>
		</div>

		<!-- Add some content to enable scrolling -->
		<div class="h-screen"></div>
	</main>

	<footer class="bg-gray-800 border-t border-gray-700 px-6 py-5 text-center">
		<p class="m-0 text-gray-500 text-sm">💡 Try keyboard shortcuts! Press ⌘K or ⌘1-4</p>
	</footer>
</div>

<style>
	/* Global body styles based on state */
	:global(body.mobile) {
		font-size: 14px;
	}

	:global(body.scrolled) {
		/* Could add global scroll effects here */
	}
</style>
```

**Key Concepts:**

- **`<svelte:head>`**: SEO - dynamic titles/meta tags per page
- **`<svelte:window>`**: Bind scroll, size; listen to keys
- **`<svelte:body>`**: Add classes for global styling
- Keyboard shortcuts improve power user experience
- Responsive design with width bindings

---

## 6. Error Boundaries

### What are Error Boundaries?

Error boundaries catch JavaScript errors in child components and display a fallback UI instead of crashing the entire app.

**Real-World Scenario:** Your app loads data from an API. If parsing fails or a component throws an error, you want to show a friendly error message, not a blank screen!

**What it does:** Wraps potentially error-prone components and handles errors gracefully.

```svelte
<script lang="ts">
	import { onError } from 'svelte';

	interface ErrorInfo {
		message: string;
		stack?: string;
		timestamp: Date;
	}

	let error = $state<ErrorInfo | null>(null);

	// Catch errors from child components
	// This prevents the error from crashing the entire app
	onError((err) => {
		// Store error details to show in fallback UI
		error = {
			message: err.message || 'An unknown error occurred',
			stack: err.stack,
			timestamp: new Date()
		};
		console.error('Error caught by boundary:', err);

		// Return true to prevent error from bubbling to parent error boundaries
		// This stops the error here and shows our fallback UI
		return true;
	});

	function reset() {
		error = null;
	}
</script>

{#if error}
	<div
		class="bg-red-900/30 border-2 border-red-500 rounded-xl p-10 text-center max-w-xl mx-auto my-10"
	>
		<div class="text-6xl mb-4 animate-shake">⚠️</div>
		<h2 class="m-0 mb-4 text-red-400 text-3xl">Something went wrong</h2>
		<p class="text-gray-300 m-0 mb-3 text-base leading-relaxed">{error.message}</p>
		<p class="text-gray-500 text-sm m-0 mb-6">
			Occurred at {error.timestamp.toLocaleTimeString()}
		</p>
		{#if error.stack}
			<details class="bg-gray-900 border border-gray-700 rounded-lg p-4 text-left mb-6">
				<summary class="text-blue-400 cursor-pointer font-semibold mb-3">Technical Details</summary
				>
				<pre
					class="text-red-400 text-xs overflow-x-auto mt-3 mb-0 leading-snug">{error.stack}</pre>
			</details>
		{/if}
		<button
			onclick={reset}
			class="bg-blue-500 text-black border-none px-8 py-3.5 rounded-lg text-base font-bold cursor-pointer transition-all hover:bg-blue-400 hover:-translate-y-0.5"
		>
			Try Again
		</button>
	</div>
{:else}
	{@render children()}
{/if}
```

> **Note:** Add `import type { Snippet } from 'svelte';` and `let { children }: { children: Snippet } = $props();` in the script section.

<style>
	@keyframes shake {
		0%,
		100% {
			transform: translateX(0);
		}
		25% {
			transform: translateX(-10px);
		}
		75% {
			transform: translateX(10px);
		}
	}

	.error-details {
		background: #1a1a1a;
		border: 1px solid #3a3a3a;
		border-radius: 8px;
		padding: 16px;
		text-align: left;
		margin-bottom: 24px;
	}

	.error-details summary {
		color: #4a9eff;
		cursor: pointer;
		font-weight: 600;
		margin-bottom: 12px;
	}

	.error-details pre {
		color: #ff6b6b;
		font-size: 12px;
		overflow-x: auto;
		margin: 12px 0 0 0;
		line-height: 1.4;
	}

	.retry-btn {
		background: #4a9eff;
		color: #000;
		border: none;
		padding: 14px 32px;
		border-radius: 8px;
		font-size: 16px;
		font-weight: 700;
		cursor: pointer;
		transition: all 0.2s;
	}

	.retry-btn:hover {
		background: #6ab0ff;
		transform: translateY(-2px);
	}
</style>

````

### Usage Example: Protected Data Display

```svelte
<script lang="ts">
	import ErrorBoundary from './ErrorBoundary.svelte';

	interface UserData {
		name: string;
		email: string;
		posts: number;
	}

	// Simulate API response that might fail
	let userData = $state<UserData | null>(null);
	let loading = $state(false);

	async function loadData() {
		loading = true;
		try {
			// Simulate API call that might fail
			await new Promise((resolve) => setTimeout(resolve, 1000));

			// Randomly fail to demonstrate error boundary
			if (Math.random() > 0.5) {
				throw new Error('Failed to fetch user data from API');
			}

			userData = {
				name: 'Jane Smith',
				email: 'jane@example.com',
				posts: 42
			};
		} catch (err) {
			throw err; // Let error boundary catch it
		} finally {
			loading = false;
		}
	}
</script>

<div class="bg-gray-900 min-h-screen py-10 px-5 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-8">User Dashboard</h1>

	<button
		onclick={loadData}
		class="block mx-auto mb-8 bg-green-500 text-black border-none px-8 py-3 rounded-lg font-semibold cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
		disabled={loading}
	>
		{loading ? 'Loading...' : 'Load User Data'}
	</button>

	<ErrorBoundary>
		{#if userData}
			<div class="bg-gray-800 border border-gray-700 rounded-xl p-6 max-w-md mx-auto">
				<h2>{userData.name}</h2>
				<p>Email: {userData.email}</p>
				<p>Posts: {userData.posts}</p>
			</div>
		{/if}
	</ErrorBoundary>
</div>
````

**Key Concepts:**

- **`onError()`**: Catches errors from children
- **Return true**: Prevents error from bubbling up
- **Fallback UI**: Show friendly message instead of crash
- **Reset mechanism**: Let users try again
- **Real-world use**: API calls, third-party components, dynamic imports

---

## 7. Binding Dimensions

### What are Dimension Bindings?

Bind to an element's `clientWidth` and `clientHeight` to react to size changes. Perfect for responsive components!

**Real-World Scenario:** You're building a chart component that needs to redraw when the window resizes, or a responsive card grid that adjusts based on container width.

**What it does:** Creates responsive components that adapt to their container size.

```svelte
<script lang="ts">
	let containerWidth = $state(0);
	let containerHeight = $state(0);

	// Derived: Calculate responsive values
	const columns = $derived(
		containerWidth < 600 ? 1 : containerWidth < 900 ? 2 : containerWidth < 1200 ? 3 : 4
	);

	const fontSize = $derived(containerWidth < 600 ? 14 : containerWidth < 900 ? 16 : 18);

	const chartSize = $derived({
		width: containerWidth - 48,
		height: containerHeight - 100
	});

	interface Product {
		id: number;
		name: string;
		price: number;
		image: string;
	}

	const products: Product[] = [
		{ id: 1, name: 'Laptop', price: 999, image: '💻' },
		{ id: 2, name: 'Phone', price: 699, image: '📱' },
		{ id: 3, name: 'Tablet', price: 499, image: '📱' },
		{ id: 4, name: 'Watch', price: 299, image: '⌚' },
		{ id: 5, name: 'Headphones', price: 199, image: '🎧' },
		{ id: 6, name: 'Camera', price: 799, image: '📷' }
	];
</script>

<div
	class="bg-gray-900 min-h-screen py-6 px-5 text-gray-200"
	bind:clientWidth={containerWidth}
	bind:clientHeight={containerHeight}
>
	<div class="mb-6">
		<h2 class="m-0 mb-3 text-blue-400 text-3xl">Responsive Product Grid</h2>
		<div class="flex gap-4 flex-wrap">
			<span class="bg-gray-800 px-4 py-2 rounded-md text-sm font-semibold border border-gray-700"
				>Width: {containerWidth}px</span
			>
			<span class="bg-gray-800 px-4 py-2 rounded-md text-sm font-semibold border border-gray-700"
				>Height: {containerHeight}px</span
			>
			<span class="bg-gray-800 px-4 py-2 rounded-md text-sm font-semibold border border-gray-700"
				>Columns: {columns}</span
			>
		</div>
	</div>

	<div
		class="grid gap-5 mb-10"
		style="grid-template-columns: repeat({columns}, 1fr); font-size: {fontSize}px;"
	>
		{#each products as product}
			<div
				class="bg-gray-800 border border-gray-700 rounded-xl p-5 text-center transition-all hover:border-blue-500 hover:-translate-y-1"
			>
				<div class="text-5xl mb-3">{product.image}</div>
				<h3 class="m-0 mb-2 text-white">{product.name}</h3>
				<p class="m-0 text-green-400 text-xl font-bold">${product.price}</p>
			</div>
		{/each}
	</div>

	<div class="mb-4">
		<h3 class="text-blue-400 mb-4">Chart Area</h3>
		<div
			class="bg-gray-800 border-2 border-dashed border-gray-700 rounded-xl mx-auto transition-all"
			style="width: {chartSize.width}px; height: {chartSize.height}px;"
		>
			<div
				class="w-full h-full flex flex-col items-center justify-center text-gray-500 text-lg text-center"
			>
				📊 Chart would render here
				<br />
				{Math.round(chartSize.width)} × {Math.round(chartSize.height)}px
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **`bind:clientWidth`** and **`bind:clientHeight`**: Get element dimensions
- **Reactive derived values**: Recalculate based on size
- **Dynamic styles**: Use bound values in `style=""` attribute
- **Responsive design**: Adjust layout/content based on size
- **Real-world use**: Charts, grids, responsive typography

---

## 8. Media Element Bindings

### What are Media Bindings?

Bind to video/audio properties like `currentTime`, `duration`, `paused`, `volume`. Build custom media players!

**Real-World Scenario:** You're building a video course platform and need a custom player with progress tracking, speed controls, and volume management.

**What it does:** Creates a fully functional custom video player with all standard controls.

```svelte
<script lang="ts">
	// Bindable media properties
	let currentTime = $state(0);
	let duration = $state(0);
	let paused = $state(true);
	let volume = $state(0.7);
	let playbackRate = $state(1);

	// Derived values
	const progress = $derived((currentTime / duration) * 100 || 0);
	const timeRemaining = $derived(duration - currentTime);

	// Format time as MM:SS
	function formatTime(seconds: number): string {
		const mins = Math.floor(seconds / 60);
		const secs = Math.floor(seconds % 60);
		return `${mins}:${secs.toString().padStart(2, '0')}`;
	}

	function togglePlay() {
		paused = !paused;
	}

	function skip(seconds: number) {
		currentTime = Math.max(0, Math.min(duration, currentTime + seconds));
	}

	function changeSpeed(speed: number) {
		playbackRate = speed;
	}
</script>

<div
	class="bg-gray-800 rounded-xl overflow-hidden border border-gray-700 max-w-4xl mx-auto my-10"
>
	<div class="relative bg-black">
		<video
			src="https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
			bind:currentTime
			bind:duration
			bind:paused
			bind:volume
			bind:playbackRate
			class="w-full block"
		></video>

		<!-- Overlay controls (appear on hover) -->
		<div
			class="absolute inset-0 flex items-center justify-center bg-black/30 opacity-0 transition-opacity cursor-pointer hover:opacity-100"
		>
			<button
				onclick={togglePlay}
				class="bg-blue-500/90 border-none w-20 h-20 rounded-full text-3xl cursor-pointer transition-all hover:bg-blue-500 hover:scale-110"
			>
				{paused ? '▶️' : '⏸️'}
			</button>
		</div>
	</div>

	<!-- Control bar -->
	<div class="bg-gray-900 px-5 py-4">
		<!-- Progress bar -->
		<div class="relative mb-4">
			<input
				type="range"
				min="0"
				max={duration}
				bind:value={currentTime}
				class="absolute -top-2 left-0 w-full h-5 opacity-0 cursor-pointer"
			/>
			<div class="h-1.5 bg-gray-700 rounded overflow-hidden">
				<div class="h-full bg-blue-500 transition-all" style="width: {progress}%"></div>
			</div>
		</div>

		<!-- Main controls -->
		<div class="flex justify-between items-center gap-5 mb-3">
			<!-- Left side: Play, skip, time -->
			<div class="flex items-center gap-3">
				<button
					onclick={togglePlay}
					class="bg-gray-800 border-2 border-gray-700 text-white px-4 py-2 rounded-md text-sm font-semibold cursor-pointer transition-all hover:border-blue-500 hover:bg-gray-700"
				>
					{paused ? '▶️' : '⏸️'}
				</button>
				<button
					onclick={() => skip(-10)}
					class="bg-gray-800 border-2 border-gray-700 text-white px-4 py-2 rounded-md text-sm font-semibold cursor-pointer transition-all hover:border-blue-500 hover:bg-gray-700"
				>
					⏪ 10s
				</button>
				<button
					onclick={() => skip(10)}
					class="bg-gray-800 border-2 border-gray-700 text-white px-4 py-2 rounded-md text-sm font-semibold cursor-pointer transition-all hover:border-blue-500 hover:bg-gray-700"
				>
					10s ⏩
				</button>
				<span class="text-gray-300 text-sm font-semibold font-mono min-w-28">
					{formatTime(currentTime)} / {formatTime(duration)}
				</span>
			</div>

			<!-- Right side: Speed, volume -->
			<div class="flex items-center gap-3">
				<!-- Speed control -->
				<select
					bind:value={playbackRate}
					class="bg-gray-800 border-2 border-gray-700 text-white px-3 py-2 rounded-md text-sm cursor-pointer"
				>
					<option value={0.5}>0.5×</option>
					<option value={0.75}>0.75×</option>
					<option value={1}>1×</option>
					<option value={1.25}>1.25×</option>
					<option value={1.5}>1.5×</option>
					<option value={2}>2×</option>
				</select>

				<!-- Volume control -->
				<div class="flex items-center gap-2">
					<span class="text-xl">
						{volume === 0 ? '🔇' : volume < 0.5 ? '🔉' : '🔊'}
					</span>
					<input
						type="range"
						min="0"
						max="1"
						step="0.01"
						bind:value={volume}
						class="w-24 accent-blue-500"
					/>
					<span class="text-gray-500 text-sm font-semibold min-w-10"
						>{Math.round(volume * 100)}%</span
					>
				</div>
			</div>
		</div>

		<!-- Time remaining indicator -->
		<div class="text-center text-gray-500 text-sm">
			⏱️ {formatTime(timeRemaining)} remaining
		</div>
	</div>
</div>
```

**Key Concepts:**

- **`bind:currentTime`**: Track/control playback position
- **`bind:duration`**: Get total video length
- **`bind:paused`**: Control play/pause state
- **`bind:volume`**: Control audio level (0-1)
- **`bind:playbackRate`**: Control speed (0.5× to 2×)
- **Real-world use**: Course platforms, video players, podcasts

---

## 9. Script Module Context

### What is Module Context?

Code in `<script context="module">` runs **once per component file**, not per instance. Perfect for sharing data between all instances!

**Real-World Scenario:** You want to track how many instances of a component exist, or share a registry/cache between all instances.

**What it does:** Shows how module-level code is shared across all component instances.

```svelte
<script lang="ts" context="module">
	// This runs ONCE when the module loads
	let totalInstances = $state(0);
	let instanceRegistry: string[] = $state([]);

	console.log('Module script runs once!');
</script>

<script lang="ts">
	// This runs for EACH component instance
	import { onMount, onDestroy } from 'svelte';

	interface Props {
		name: string;
	}

	let { name }: Props = $props();

	// Each instance gets a unique ID
	const instanceId = `${name}-${totalInstances}`;

	onMount(() => {
		totalInstances++;
		instanceRegistry = [...instanceRegistry, instanceId];
		console.log(`${instanceId} mounted. Total: ${totalInstances}`);
	});

	onDestroy(() => {
		totalInstances--;
		instanceRegistry = instanceRegistry.filter((id) => id !== instanceId);
		console.log(`${instanceId} destroyed. Total: ${totalInstances}`);
	});
</script>

<div class="bg-gray-800 border border-gray-700 rounded-xl p-6 mb-4">
	<h3 class="m-0 mb-4 text-blue-400 text-xl">Instance: {instanceId}</h3>
	<div class="flex flex-col gap-3">
		<div class="bg-gray-900 p-3 rounded-md flex justify-between items-center">
			<span class="text-gray-500 text-sm font-semibold">Total Instances:</span>
			<span class="text-green-400 text-sm font-bold">{totalInstances}</span>
		</div>
		<div class="bg-gray-900 p-3 rounded-md flex justify-between items-center">
			<span class="text-gray-500 text-sm font-semibold">All Instances:</span>
			<span class="text-green-400 text-sm font-bold">{instanceRegistry.join(', ')}</span>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Module context**: Code runs once for the file
- **Shared state**: All instances see the same values
- **Instance tracking**: Count or register component instances
- **Performance**: Avoid duplicate initialization

---

## 10. Shared Component State

### Real-World Scenario: Coordinated Components

You're building a multi-select filter where all filter chips need to share selected state.

**What it does:** Creates filter chips that share state via module context.

```svelte
<!-- FilterChip.svelte -->
<script lang="ts" context="module">
	// Shared across ALL filter chips
	let selectedFilters = $state<Set<string>>(new Set());

	export function getSelectedFilters(): string[] {
		return Array.from(selectedFilters);
	}

	export function clearAllFilters() {
		selectedFilters = new Set();
	}
</script>

<script lang="ts">
	interface Props {
		id: string;
		label: string;
		icon?: string;
	}

	let { id, label, icon }: Props = $props();

	const isSelected = $derived(selectedFilters.has(id));

	function toggle() {
		if (isSelected) {
			selectedFilters.delete(id);
		} else {
			selectedFilters.add(id);
		}
		// Trigger reactivity
		selectedFilters = selectedFilters;
	}
</script>

<button
	class="bg-gray-800 border-2 border-gray-700 text-gray-400 px-4 py-2.5 rounded-full text-sm font-semibold cursor-pointer transition-all flex items-center gap-2 hover:border-blue-400 hover:bg-gray-700"
	class:bg-blue-400={isSelected}
	class:border-blue-400={isSelected}
	class:text-black={isSelected}
	onclick={toggle}
>
	{#if icon}
		<span class="text-base">{icon}</span>
	{/if}
	<span>{label}</span>
	{#if isSelected}
		<span class="text-xs font-bold">✓</span>
	{/if}
</button>
```

### Usage:

```svelte
<script lang="ts">
	import FilterChip, { getSelectedFilters, clearAllFilters } from './FilterChip.svelte';

	const selectedCount = $derived(getSelectedFilters().length);
</script>

<div class="bg-gray-900 py-8 px-8 rounded-xl max-w-2xl mx-auto my-10">
	<h2 class="m-0 mb-6 text-blue-400">Filter Products</h2>

	<div class="flex flex-wrap gap-3 mb-6">
		<FilterChip id="electronics" label="Electronics" icon="💻" />
		<FilterChip id="clothing" label="Clothing" icon="👕" />
		<FilterChip id="books" label="Books" icon="📚" />
		<FilterChip id="sports" label="Sports" icon="⚽" />
		<FilterChip id="home" label="Home" icon="🏠" />
		<FilterChip id="toys" label="Toys" icon="🎮" />
	</div>

	<div class="flex justify-between items-center pt-6 border-t border-gray-700">
		<div class="text-gray-500 text-sm font-semibold">
			{selectedCount} filter{selectedCount === 1 ? '' : 's'} selected
		</div>
		<button
			onclick={clearAllFilters}
			class="bg-red-400 text-white border-none px-5 py-2.5 rounded-lg font-semibold cursor-pointer"
		>
			Clear All
		</button>
	</div>
</div>
```

**Key Concepts:**

- **Shared state**: All chips access same `selectedFilters` Set
- **Export functions**: Provide API to external components
- **Reactivity**: Reassign to trigger updates (`selectedFilters = selectedFilters`)
- **Real-world use**: Filters, multi-select, coordinated animations

---

## 11. Multi-Video Controller

### Real-World Scenario: Video Course Platform

You're building a course with multiple video lessons. When one plays, pause all others!

**What it does:** Coordinates multiple video players to ensure only one plays at a time.

```svelte
<!-- VideoLesson.svelte -->
<script lang="ts" context="module">
	// Track currently playing video
	let currentlyPlaying = $state<string | null>(null);

	export function pauseAll() {
		currentlyPlaying = null;
	}
</script>

<script lang="ts">
	interface Props {
		id: string;
		title: string;
		videoSrc: string;
		duration: string;
	}

	let { id, title, videoSrc, duration }: Props = $props();

	let paused = $state(true);
	let currentTime = $state(0);
	let videoDuration = $state(0);

	const progress = $derived((currentTime / videoDuration) * 100 || 0);
	const isCurrentlyPlaying = $derived(currentlyPlaying === id);

	// When this video plays, pause all others
	$effect(() => {
		if (!paused && currentlyPlaying !== id) {
			currentlyPlaying = id;
		}
	});

	// When another video plays, pause this one
	$effect(() => {
		if (currentlyPlaying !== null && currentlyPlaying !== id && !paused) {
			paused = true;
		}
	});

	function formatTime(seconds: number): string {
		const mins = Math.floor(seconds / 60);
		const secs = Math.floor(seconds % 60);
		return `${mins}:${secs.toString().padStart(2, '0')}`;
	}
</script>

<div
	class="bg-gray-800 border-2 border-gray-700 rounded-xl overflow-hidden transition-all"
	class:border-blue-400={isCurrentlyPlaying}
	class:shadow-lg={isCurrentlyPlaying}
>
	<div class="p-5 flex justify-between items-center border-b border-gray-700">
		<div>
			<h3 class="m-0 mb-1 text-white text-lg">{title}</h3>
			<span class="text-gray-500 text-sm">⏱️ {duration}</span>
		</div>
		{#if isCurrentlyPlaying}
			<span
				class="bg-blue-400 text-black px-3 py-1.5 rounded-md text-xs font-bold animate-pulse"
				>▶️ Playing</span
			>
		{/if}
	</div>

	<video
		src={videoSrc}
		bind:currentTime
		bind:duration={videoDuration}
		bind:paused
		class="w-full block bg-black"
	></video>

	<div class="px-5 py-4 flex items-center gap-3">
		<button
			onclick={() => (paused = !paused)}
			class="bg-blue-400 text-black border-none px-5 py-2.5 rounded-md font-bold cursor-pointer transition-all whitespace-nowrap hover:bg-blue-300 hover:scale-105"
		>
			{paused ? '▶️ Play' : '⏸️ Pause'}
		</button>

		<div class="flex-1 h-1.5 bg-gray-700 rounded-sm overflow-hidden">
			<div class="h-full bg-blue-400 transition-[width_0.1s]" style="width: {progress}%"></div>
		</div>

		<span class="text-gray-500 text-sm font-semibold font-mono min-w-24 text-right">
			{formatTime(currentTime)} / {formatTime(videoDuration)}
		</span>
	</div>
</div>

<style>
	@keyframes pulse {
		0%,
		100% {
			opacity: 1;
		}
		50% {
			opacity: 0.7;
		}
	}
</style>
```

### Usage:

```svelte
<script lang="ts">
	import VideoLesson, { pauseAll } from './VideoLesson.svelte';
</script>

<div class="bg-gray-900 min-h-screen py-10 px-5">
	<div class="max-w-4xl mx-auto mb-8 flex justify-between items-center">
		<h1 class="m-0 text-blue-400 text-3xl">📚 Svelte 5 Mastery Course</h1>
		<button
			onclick={pauseAll}
			class="bg-red-400 text-white border-none px-6 py-3 rounded-lg font-bold cursor-pointer"
		>
			⏸️ Pause All Videos
		</button>
	</div>

	<div class="max-w-4xl mx-auto flex flex-col gap-6">
		<VideoLesson
			id="lesson-1"
			title="Lesson 1: Introduction to Runes"
			videoSrc="https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
			duration="12:30"
		/>
		<VideoLesson
			id="lesson-2"
			title="Lesson 2: State Management"
			videoSrc="https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ElephantsDream.mp4"
			duration="15:45"
		/>
		<VideoLesson
			id="lesson-3"
			title="Lesson 3: Component Patterns"
			videoSrc="https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/ForBiggerBlazes.mp4"
			duration="18:20"
		/>
	</div>
</div>
```

**Key Concepts:**

- **Module-level coordination**: Share `currentlyPlaying` across instances
- **$effect for sync**: Watch for changes and react
- **Export control functions**: `pauseAll()` for external control
- **Real-world use**: Video courses, audio playlists, slideshow galleries

---

## 📝 Key Takeaways

✅ Actions integrate third-party libraries and add reusable DOM behavior
✅ Special elements access browser APIs (window, document, head, body)
✅ Error boundaries catch errors and show fallback UI
✅ Dimension bindings create responsive components
✅ Media bindings build custom video/audio players
✅ Module context shares data across all component instances
✅ Always clean up in `destroy()` to prevent memory leaks

### 💡 Best Practices for Actions

**Action Organization:**

```
src/lib/actions/
├── dom/
│   ├── clickOutside.ts      # Click detection
│   ├── longpress.ts         # Long press gesture
│   └── autofocus.ts         # Focus management
├── integrations/
│   ├── tippy.ts             # Tooltip library
│   ├── sortable.ts          # Drag-drop library
│   └── chart.ts             # Chart library
└── index.ts                 # Barrel export
```

**Action Pattern Template:**

```typescript
export function myAction(node: HTMLElement, params?: ActionParams) {
  // 1. Setup - Initialize library/listeners
  const instance = library.create(node, params);

  // 2. Update - Handle parameter changes
  function update(newParams?: ActionParams) {
    instance.setOptions(newParams);
  }

  // 3. Cleanup - CRITICAL for memory leaks
  function destroy() {
    instance.destroy();
    // Remove event listeners
    // Cancel timers
    // Clear references
  }

  return { update, destroy };
}
```

**When to Use Actions vs Components:**

```svelte
<!-- ✅ Use actions for DOM enhancements -->
<button use:ripple use:tooltip={'Click me'}>Button</button>

<!-- ❌ Don't wrap in components unnecessarily -->
<TooltipWrapper text="Click me">
	<RippleWrapper>
		<button>Button</button>
	</RippleWrapper>
</TooltipWrapper>

<!-- ✅ Use components for complex UI with state -->
<Modal bind:this={modal}>
	<ModalHeader>Title</ModalHeader>
	<ModalContent>Content</ModalContent>
</Modal>
```

### ⚠️ Common Mistakes

1. **Forgetting to clean up** - memory leaks:

```typescript
// ❌ Bad - listeners never removed
export function clickOutside(node: HTMLElement) {
  window.addEventListener("click", handler);
  // Missing destroy!
}

// ✅ Good - cleanup in destroy
export function clickOutside(node: HTMLElement) {
  window.addEventListener("click", handler);
  return {
    destroy() {
      window.removeEventListener("click", handler);
    },
  };
}
```

2. **Not handling SSR** - actions run on server:

```typescript
// ❌ Bad - crashes during SSR
export function action(node: HTMLElement) {
  const instance = new BrowserOnlyLibrary(node);
}

// ✅ Good - checks for browser
export function action(node: HTMLElement) {
  if (typeof window === "undefined") return;
  const instance = new BrowserOnlyLibrary(node);
}
```

3. **Overusing module context** - causes coupling:

```typescript
// ❌ Bad - global state for everything
<script context="module">
  let theme = $state('light');
  let user = $state(null);
  let cart = $state([]);
</script>

// ✅ Good - use Context API or stores instead
import { getTheme, getUser, getCart } from '$lib/contexts';
```

### ⚡ Performance Tips

- **Debounce resize events** - Don't react to every pixel change
- **Use IntersectionObserver** - Better than scroll events for visibility
- **Lazy-load actions** - Import heavy libraries only when needed
- **Batch dimension updates** - Use `requestAnimationFrame` for smooth animations
- **Cleanup media bindings** - Pause videos when hidden to save resources

### 🧪 Testing Actions

```typescript
import { tick } from "svelte";
import { render } from "@testing-library/svelte";
import { clickOutside } from "./clickOutside";

test("clickOutside triggers callback", async () => {
  const callback = vi.fn();
  const node = document.createElement("div");

  clickOutside(node, callback);

  // Click outside
  document.body.click();
  await tick();

  expect(callback).toHaveBeenCalled();
});
```

---

## 🚀 End-of-Section Project: Interactive Dashboard with Rich Media

### Project Overview

Build an **interactive admin dashboard** that demonstrates actions, special elements, media bindings, and module-level state. This project combines all Section 7 concepts into a practical application.

**What You'll Build:**

- **Custom actions**: `clickOutside`, `tooltip`, `lazyLoad`, `shortcut`
- **Media bindings**: Video player with progress tracking
- **Dimension bindings**: Responsive layout detection
- **Module-level state**: Notification system across components
- **Special elements**: `<svelte:window>`, `<svelte:body>`, `<svelte:document>`

### 📁 Files to Create

**1. Click Outside Action** - `src/lib/actions/clickOutside.ts`

```typescript
export function clickOutside(
  node: HTMLElement,
  callback: () => void
): { destroy: () => void } {
  function handleClick(event: MouseEvent) {
    if (!node.contains(event.target as Node)) {
      callback();
    }
  }

  document.addEventListener("click", handleClick, true);

  return {
    destroy() {
      document.removeEventListener("click", handleClick, true);
    },
  };
}
```

**2. Tooltip Action** - `src/lib/actions/tooltip.ts`

```typescript
interface TooltipOptions {
  text: string;
  position?: "top" | "bottom" | "left" | "right";
}

export function tooltip(
  node: HTMLElement,
  options: TooltipOptions
): { update: (opts: TooltipOptions) => void; destroy: () => void } {
  let tooltipEl: HTMLDivElement | null = null;
  let currentOptions = options;

  function createTooltip() {
    tooltipEl = document.createElement("div");
    tooltipEl.className = `
			fixed z-50 px-3 py-2 text-sm text-white bg-gray-900 rounded-lg shadow-lg
			pointer-events-none transition-opacity duration-200
		`;
    tooltipEl.textContent = currentOptions.text;
    document.body.appendChild(tooltipEl);
    positionTooltip();
  }

  function positionTooltip() {
    if (!tooltipEl) return;
    const rect = node.getBoundingClientRect();
    const pos = currentOptions.position || "top";

    const positions = {
      top: { left: rect.left + rect.width / 2, top: rect.top - 8 },
      bottom: { left: rect.left + rect.width / 2, top: rect.bottom + 8 },
      left: { left: rect.left - 8, top: rect.top + rect.height / 2 },
      right: { left: rect.right + 8, top: rect.top + rect.height / 2 },
    };

    const { left, top } = positions[pos];
    tooltipEl.style.left = `${left}px`;
    tooltipEl.style.top = `${top}px`;
    tooltipEl.style.transform =
      pos === "top" || pos === "bottom"
        ? "translateX(-50%)" + (pos === "top" ? " translateY(-100%)" : "")
        : "translateY(-50%)" + (pos === "left" ? " translateX(-100%)" : "");
  }

  function removeTooltip() {
    tooltipEl?.remove();
    tooltipEl = null;
  }

  node.addEventListener("mouseenter", createTooltip);
  node.addEventListener("mouseleave", removeTooltip);

  return {
    update(newOptions: TooltipOptions) {
      currentOptions = newOptions;
      if (tooltipEl) {
        tooltipEl.textContent = currentOptions.text;
        positionTooltip();
      }
    },
    destroy() {
      removeTooltip();
      node.removeEventListener("mouseenter", createTooltip);
      node.removeEventListener("mouseleave", removeTooltip);
    },
  };
}
```

**3. Keyboard Shortcut Action** - `src/lib/actions/shortcut.ts`

```typescript
interface ShortcutOptions {
  key: string;
  ctrl?: boolean;
  shift?: boolean;
  alt?: boolean;
  callback: () => void;
}

export function shortcut(
  node: HTMLElement,
  options: ShortcutOptions
): { update: (opts: ShortcutOptions) => void; destroy: () => void } {
  let currentOptions = options;

  function handleKeydown(event: KeyboardEvent) {
    const matchesKey =
      event.key.toLowerCase() === currentOptions.key.toLowerCase();
    const matchesCtrl = !currentOptions.ctrl || event.ctrlKey || event.metaKey;
    const matchesShift = !currentOptions.shift || event.shiftKey;
    const matchesAlt = !currentOptions.alt || event.altKey;

    if (matchesKey && matchesCtrl && matchesShift && matchesAlt) {
      event.preventDefault();
      currentOptions.callback();
    }
  }

  window.addEventListener("keydown", handleKeydown);

  return {
    update(newOptions: ShortcutOptions) {
      currentOptions = newOptions;
    },
    destroy() {
      window.removeEventListener("keydown", handleKeydown);
    },
  };
}
```

**4. Notification System** - `src/lib/components/Notifications.svelte`

```svelte
<script lang="ts" module>
	interface Notification {
		id: number;
		type: 'success' | 'error' | 'warning' | 'info';
		message: string;
	}

	let notifications = $state<Notification[]>([]);
	let nextId = 0;

	export function notify(type: Notification['type'], message: string, duration = 4000) {
		const id = nextId++;
		notifications.push({ id, type, message });

		setTimeout(() => {
			notifications = notifications.filter((n) => n.id !== id);
		}, duration);
	}

	export function clearAll() {
		notifications = [];
	}
</script>

<script lang="ts">
	const icons = {
		success: '✅',
		error: '❌',
		warning: '⚠️',
		info: 'ℹ️'
	};

	const colors = {
		success: 'bg-green-600 border-green-400',
		error: 'bg-red-600 border-red-400',
		warning: 'bg-yellow-600 border-yellow-400',
		info: 'bg-blue-600 border-blue-400'
	};
</script>

<div class="fixed top-4 right-4 z-50 flex flex-col gap-3 max-w-sm">
	{#each notifications as notification (notification.id)}
		<div
			class="px-4 py-3 rounded-lg border-l-4 shadow-lg text-white flex items-center gap-3 animate-in slide-in-from-right {colors[
				notification.type
			]}"
		>
			<span class="text-xl">{icons[notification.type]}</span>
			<span class="flex-1">{notification.message}</span>
			<button
				class="text-white/70 hover:text-white"
				onclick={() => (notifications = notifications.filter((n) => n.id !== notification.id))}
			>
				✕
			</button>
		</div>
	{/each}
</div>

<style>
	@keyframes slide-in-from-right {
		from {
			transform: translateX(100%);
			opacity: 0;
		}
		to {
			transform: translateX(0);
			opacity: 1;
		}
	}

	.animate-in {
		animation: slide-in-from-right 0.3s ease-out;
	}
</style>
```

**5. Video Player Component** - `src/lib/components/VideoPlayer.svelte`

```svelte
<script lang="ts">
	import { tooltip } from '$lib/actions/tooltip';

	interface Props {
		src: string;
		title: string;
	}

	let { src, title }: Props = $props();

	let videoEl: HTMLVideoElement;
	let currentTime = $state(0);
	let duration = $state(0);
	let paused = $state(true);
	let volume = $state(1);
	let muted = $state(false);
	let playbackRate = $state(1);

	const progress = $derived(duration > 0 ? (currentTime / duration) * 100 : 0);

	function formatTime(seconds: number): string {
		const mins = Math.floor(seconds / 60);
		const secs = Math.floor(seconds % 60);
		return `${mins}:${secs.toString().padStart(2, '0')}`;
	}

	function seek(e: MouseEvent) {
		const rect = (e.target as HTMLElement).getBoundingClientRect();
		const percent = (e.clientX - rect.left) / rect.width;
		currentTime = percent * duration;
	}

	function togglePlay() {
		paused = !paused;
	}

	function cycleSpeed() {
		const speeds = [0.5, 1, 1.25, 1.5, 2];
		const currentIndex = speeds.indexOf(playbackRate);
		playbackRate = speeds[(currentIndex + 1) % speeds.length];
	}
</script>

<div class="bg-gray-900 rounded-xl overflow-hidden border border-gray-700">
	<div class="p-4 border-b border-gray-700 flex justify-between items-center">
		<h3 class="text-white font-semibold">{title}</h3>
		{#if !paused}
			<span class="bg-green-500 text-black px-2 py-1 rounded text-xs font-bold animate-pulse">
				▶ Playing
			</span>
		{/if}
	</div>

	<video
		bind:this={videoEl}
		bind:currentTime
		bind:duration
		bind:paused
		bind:volume
		bind:muted
		bind:playbackRate
		{src}
		class="w-full"
	></video>

	<div class="p-4 space-y-3">
		<!-- Progress bar -->
		<!-- svelte-ignore a11y_click_events_have_key_events a11y_no_static_element_interactions -->
		<div
			class="h-2 bg-gray-700 rounded-full cursor-pointer overflow-hidden"
			onclick={seek}
		>
			<div class="h-full bg-blue-500 transition-all" style:width="{progress}%"></div>
		</div>

		<div class="flex items-center justify-between">
			<div class="flex items-center gap-2">
				<button
					use:tooltip={{ text: paused ? 'Play (Space)' : 'Pause (Space)', position: 'top' }}
					class="w-10 h-10 bg-blue-500 hover:bg-blue-400 text-white rounded-lg font-bold transition-colors"
					onclick={togglePlay}
				>
					{paused ? '▶' : '⏸'}
				</button>

				<button
					use:tooltip={{ text: muted ? 'Unmute' : 'Mute', position: 'top' }}
					class="w-10 h-10 bg-gray-700 hover:bg-gray-600 text-white rounded-lg transition-colors"
					onclick={() => (muted = !muted)}
				>
					{muted ? '🔇' : '🔊'}
				</button>

				<input
					type="range"
					min="0"
					max="1"
					step="0.1"
					bind:value={volume}
					class="w-20 accent-blue-500"
				/>
			</div>

			<span class="text-gray-400 font-mono text-sm">
				{formatTime(currentTime)} / {formatTime(duration)}
			</span>

			<button
				use:tooltip={{ text: `Speed: ${playbackRate}x`, position: 'top' }}
				class="px-3 py-1 bg-gray-700 hover:bg-gray-600 text-white rounded-lg text-sm transition-colors"
				onclick={cycleSpeed}
			>
				{playbackRate}x
			</button>
		</div>
	</div>
</div>
```

**6. Dashboard Sidebar** - `src/lib/components/Sidebar.svelte`

```svelte
<script lang="ts">
	import { clickOutside } from '$lib/actions/clickOutside';
	import { shortcut } from '$lib/actions/shortcut';

	interface Props {
		open: boolean;
		onclose: () => void;
	}

	let { open, onclose }: Props = $props();

	const menuItems = [
		{ icon: '🏠', label: 'Dashboard', href: '/' },
		{ icon: '📊', label: 'Analytics', href: '/analytics' },
		{ icon: '👥', label: 'Users', href: '/users' },
		{ icon: '📁', label: 'Files', href: '/files' },
		{ icon: '⚙️', label: 'Settings', href: '/settings' }
	];
</script>

{#if open}
	<!-- Backdrop -->
	<div class="fixed inset-0 bg-black/50 z-40 lg:hidden" onclick={onclose}></div>

	<!-- Sidebar -->
	<aside
		use:clickOutside={onclose}
		use:shortcut={{ key: 'Escape', callback: onclose }}
		class="fixed left-0 top-0 h-full w-64 bg-gray-900 border-r border-gray-700 z-50 p-4"
	>
		<div class="flex justify-between items-center mb-8">
			<h2 class="text-xl font-bold text-white">📊 Dashboard</h2>
			<button
				class="lg:hidden text-gray-400 hover:text-white"
				onclick={onclose}
			>
				✕
			</button>
		</div>

		<nav class="space-y-2">
			{#each menuItems as item}
				<a
					href={item.href}
					class="flex items-center gap-3 px-4 py-3 rounded-lg text-gray-300 hover:bg-gray-800 hover:text-white transition-colors"
				>
					<span class="text-xl">{item.icon}</span>
					<span>{item.label}</span>
				</a>
			{/each}
		</nav>

		<div class="absolute bottom-4 left-4 right-4">
			<div class="text-xs text-gray-500">
				Press <kbd class="px-1 py-0.5 bg-gray-700 rounded">Esc</kbd> to close
			</div>
		</div>
	</aside>
{/if}
```

**7. Main Dashboard Page** - `src/routes/dashboard/+page.svelte`

```svelte
<script lang="ts">
	import { shortcut } from '$lib/actions/shortcut';
	import { tooltip } from '$lib/actions/tooltip';
	import Notifications, { notify } from '$lib/components/Notifications.svelte';
	import VideoPlayer from '$lib/components/VideoPlayer.svelte';
	import Sidebar from '$lib/components/Sidebar.svelte';

	// Dimension bindings for responsive layout
	let innerWidth = $state(0);
	let innerHeight = $state(0);

	const isMobile = $derived(innerWidth < 768);
	const isTablet = $derived(innerWidth >= 768 && innerWidth < 1024);

	// Sidebar state
	let sidebarOpen = $state(false);

	// Online status
	let online = $state(true);

	// Visibility tracking
	let hidden = $state(false);

	$effect(() => {
		if (!hidden) {
			console.log('Tab is visible - resuming updates');
		} else {
			console.log('Tab is hidden - pausing updates');
		}
	});

	// Keyboard shortcuts
	function handleShortcuts(node: HTMLElement) {
		return shortcut(node, {
			key: 'k',
			ctrl: true,
			callback: () => notify('info', 'Search opened! (Ctrl+K)')
		});
	}

	// Stats for dashboard
	const stats = [
		{ label: 'Total Users', value: '12,453', change: '+12%', icon: '👥' },
		{ label: 'Revenue', value: '$84,230', change: '+8%', icon: '💰' },
		{ label: 'Active Sessions', value: '1,429', change: '+23%', icon: '📊' },
		{ label: 'Conversion Rate', value: '3.2%', change: '+2%', icon: '📈' }
	];
</script>

<svelte:window bind:innerWidth bind:innerHeight bind:online />
<svelte:document bind:hidden />

<div use:handleShortcuts class="min-h-screen bg-gray-950">
	<!-- Notifications container -->
	<Notifications />

	<!-- Sidebar -->
	<Sidebar open={sidebarOpen} onclose={() => (sidebarOpen = false)} />

	<!-- Main content -->
	<div class="p-6 lg:pl-72">
		<!-- Header -->
		<header class="flex items-center justify-between mb-8">
			<div class="flex items-center gap-4">
				<button
					class="lg:hidden p-2 bg-gray-800 rounded-lg text-white"
					onclick={() => (sidebarOpen = true)}
				>
					☰
				</button>
				<h1 class="text-2xl font-bold text-white">Dashboard</h1>
			</div>

			<div class="flex items-center gap-4">
				<!-- Online indicator -->
				<div
					use:tooltip={{ text: online ? 'Connected' : 'Offline', position: 'bottom' }}
					class="flex items-center gap-2 px-3 py-1 rounded-full text-sm {online
						? 'bg-green-900 text-green-400'
						: 'bg-red-900 text-red-400'}"
				>
					<span class="w-2 h-2 rounded-full {online ? 'bg-green-400' : 'bg-red-400'}"></span>
					{online ? 'Online' : 'Offline'}
				</div>

				<!-- Screen size indicator -->
				<div class="text-gray-500 text-sm hidden md:block">
					{innerWidth}×{innerHeight}
					{#if isMobile}(Mobile){:else if isTablet}(Tablet){:else}(Desktop){/if}
				</div>

				<button
					use:tooltip={{ text: 'Notifications', position: 'bottom' }}
					class="p-2 bg-gray-800 rounded-lg text-white hover:bg-gray-700"
					onclick={() => notify('info', 'You have 3 new notifications')}
				>
					🔔
				</button>
			</div>
		</header>

		<!-- Stats Grid -->
		<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-4 gap-6 mb-8">
			{#each stats as stat}
				<div class="bg-gray-900 rounded-xl p-6 border border-gray-800">
					<div class="flex items-center justify-between mb-4">
						<span class="text-3xl">{stat.icon}</span>
						<span class="text-green-400 text-sm font-semibold">{stat.change}</span>
					</div>
					<div class="text-2xl font-bold text-white mb-1">{stat.value}</div>
					<div class="text-gray-500 text-sm">{stat.label}</div>
				</div>
			{/each}
		</div>

		<!-- Video Section -->
		<div class="grid grid-cols-1 lg:grid-cols-2 gap-6 mb-8">
			<VideoPlayer
				src="https://commondatastorage.googleapis.com/gtv-videos-bucket/sample/BigBuckBunny.mp4"
				title="Training Video: Getting Started"
			/>

			<div class="bg-gray-900 rounded-xl p-6 border border-gray-800">
				<h2 class="text-xl font-bold text-white mb-4">Quick Actions</h2>
				<div class="grid grid-cols-2 gap-4">
					<button
						use:tooltip={{ text: 'Create new user', position: 'top' }}
						class="p-4 bg-blue-600 hover:bg-blue-500 rounded-lg text-white transition-colors"
						onclick={() => notify('success', 'User created successfully!')}
					>
						➕ Add User
					</button>
					<button
						use:tooltip={{ text: 'Generate report', position: 'top' }}
						class="p-4 bg-purple-600 hover:bg-purple-500 rounded-lg text-white transition-colors"
						onclick={() => notify('info', 'Report generation started...')}
					>
						📄 Report
					</button>
					<button
						use:tooltip={{ text: 'Export all data', position: 'bottom' }}
						class="p-4 bg-green-600 hover:bg-green-500 rounded-lg text-white transition-colors"
						onclick={() => notify('success', 'Export completed!')}
					>
						📤 Export
					</button>
					<button
						use:tooltip={{ text: 'Open settings', position: 'bottom' }}
						class="p-4 bg-gray-700 hover:bg-gray-600 rounded-lg text-white transition-colors"
						onclick={() => notify('warning', 'Settings panel coming soon')}
					>
						⚙️ Settings
					</button>
				</div>
			</div>
		</div>

		<!-- Keyboard shortcuts help -->
		<div class="bg-gray-900 rounded-xl p-6 border border-gray-800">
			<h2 class="text-xl font-bold text-white mb-4">⌨️ Keyboard Shortcuts</h2>
			<div class="grid grid-cols-2 md:grid-cols-4 gap-4 text-sm">
				<div class="flex items-center gap-2 text-gray-400">
					<kbd class="px-2 py-1 bg-gray-700 rounded">Ctrl+K</kbd>
					<span>Search</span>
				</div>
				<div class="flex items-center gap-2 text-gray-400">
					<kbd class="px-2 py-1 bg-gray-700 rounded">Esc</kbd>
					<span>Close sidebar</span>
				</div>
				<div class="flex items-center gap-2 text-gray-400">
					<kbd class="px-2 py-1 bg-gray-700 rounded">Space</kbd>
					<span>Play/Pause</span>
				</div>
				<div class="flex items-center gap-2 text-gray-400">
					<kbd class="px-2 py-1 bg-gray-700 rounded">?</kbd>
					<span>Help</span>
				</div>
			</div>
		</div>
	</div>
</div>
```

### 🎯 Project Checklist

- [ ] `clickOutside` action closes sidebar when clicking outside
- [ ] `tooltip` action shows helpful hints on hover
- [ ] `shortcut` action handles Ctrl+K and Escape key
- [ ] Video player uses all media bindings (currentTime, duration, paused, volume, muted, playbackRate)
- [ ] Dimension bindings track window size for responsive layout
- [ ] `<svelte:window>` binds `innerWidth`, `innerHeight`, and `online`
- [ ] `<svelte:document>` tracks tab visibility with `hidden`
- [ ] Module-level notification system works across components
- [ ] Sidebar has proper cleanup on destroy

### 💡 Extension Ideas

1. **Add `longpress` action** for mobile touch interactions
2. **Implement `lazyLoad` action** for images below the fold
3. **Add `copyToClipboard` action** with feedback notification
4. **Track scroll position** with `<svelte:window bind:scrollY>`
5. **Add fullscreen toggle** using `<svelte:body>` class binding

---

## 🚀 Next Steps

After mastering this section, you'll be ready for:

- **Section 8**: Animations and transitions
- Building production-ready applications
- Integrating any third-party library
- Creating accessible, responsive UIs
