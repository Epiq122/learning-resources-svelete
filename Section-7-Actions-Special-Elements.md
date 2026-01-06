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

  return {
    // Optional: Update when parameters change
    update(newParameters: any) {
      // Update logic
    },

    // Optional: Cleanup when element is removed
    destroy() {
      // Cleanup logic (remove listeners, etc.)
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

		function handleMouseDown() {
			timer = setTimeout(() => {
				// Dispatch custom event after duration
				node.dispatchEvent(new CustomEvent('longpress'));
			}, duration);
		}

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
			class:bg-[#2a1a1a]={deletingId === todo.id}
			class:animate-[shake_0.3s]={deletingId === todo.id}
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
		// Create tippy instance
		const instance: Instance = tippy(node, {
			...options,
			theme: 'custom' // Our custom theme
		});

		return {
			update(newOptions: Partial<Props>) {
				// Update tooltip when options change
				instance.setProps(newOptions);
			},
			destroy() {
				// Cleanup
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
		<label for="email" class="flex items-center gap-2 text-[#ccc] text-sm font-semibold mb-2">
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
			class="w-full px-4 py-3 bg-[#1a1a1a] border-2 border-[#3a3a3a] rounded-lg text-white text-base transition-colors focus:outline-none focus:border-[#4a9eff]"
		/>
	</div>

	<div class="mb-5">
		<label for="password" class="flex items-center gap-2 text-[#ccc] text-sm font-semibold mb-2">
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
			class="w-full px-4 py-3 bg-[#1a1a1a] border-2 border-[#3a3a3a] rounded-lg text-white text-base transition-colors focus:outline-none focus:border-[#4a9eff]"
		/>
	</div>

	<button
		type="submit"
		class="w-full bg-[#4a9eff] text-black border-none py-[14px] rounded-lg text-base font-bold cursor-pointer transition-all mt-2 hover:bg-[#6ab0ff] hover:-translate-y-0.5"
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
			// Check if click was outside the element
			if (node && !node.contains(event.target as Node)) {
				node.dispatchEvent(new CustomEvent('clickoutside'));
			}
		}

		// Use capture phase to catch events before they bubble
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
		class="bg-[#2a2a2a] border-2 border-[#3a3a3a] text-white px-4 py-[10px] rounded-lg flex items-center gap-3 cursor-pointer transition-all hover:border-[#4a9eff]"
		onclick={() => (isOpen = !isOpen)}
	>
		<span class="text-2xl">{user.avatar}</span>
		<span class="font-semibold text-sm">{user.name}</span>
		<span class="text-xs transition-transform ml-1" class:rotate-180={isOpen}>▼</span>
	</button>

	{#if isOpen}
		<div
			class="absolute top-[calc(100%+8px)] right-0 bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl min-w-[280px] shadow-[0_8px_24px_rgba(0,0,0,0.5)] animate-[slideDown_0.2s_ease] z-[100]"
			use:clickOutside
			onclickoutside={() => (isOpen = false)}
		>
			<div class="p-4 flex items-center gap-3">
				<span class="text-5xl">{user.avatar}</span>
				<div class="flex flex-col gap-1">
					<strong class="text-white text-[15px]">{user.name}</strong>
					<span class="text-[#888] text-[13px]">{user.email}</span>
				</div>
			</div>

			<div class="h-px bg-[#3a3a3a] my-2"></div>

			<button
				class="w-full bg-transparent border-none text-white px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-[10px] hover:bg-[#1a1a1a]"
			>
				⚙️ Settings
			</button>
			<button
				class="w-full bg-transparent border-none text-white px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-[10px] hover:bg-[#1a1a1a]"
			>
				🎨 Appearance
			</button>
			<button
				class="w-full bg-transparent border-none text-white px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-[10px] hover:bg-[#1a1a1a]"
			>
				📊 Analytics
			</button>

			<div class="h-px bg-[#3a3a3a] my-2"></div>

			<button
				class="w-full bg-transparent border-none text-[#ff6b6b] px-4 py-3 text-left cursor-pointer text-sm font-medium transition-colors flex items-center gap-[10px] hover:bg-[#2a1a1a]"
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

<div class="bg-[#1a1a1a] min-h-screen text-[#e0e0e0]">
	<!-- Fixed header that changes when scrolled -->
	<header
		class="sticky top-0 bg-[#2a2a2a] border-b-2 border-[#3a3a3a] px-6 py-5 flex justify-between items-center z-[100] transition-all"
		class:!py-3={isScrolled}
		class:bg-[rgba(42,42,42,0.95)]={isScrolled}
		class:backdrop-blur-[10px]={isScrolled}
		class:shadow-[0_4px_12px_rgba(0,0,0,0.3)]={isScrolled}
	>
		<h1 class="m-0 text-[#4a9eff] text-2xl">My SaaS Dashboard</h1>
		<div class="flex gap-4">
			<span class="bg-[#1a1a1a] px-3 py-1.5 rounded-md text-[13px] font-semibold font-mono"
				>📱 {windowWidth}px</span
			>
			<span class="bg-[#1a1a1a] px-3 py-1.5 rounded-md text-[13px] font-semibold font-mono"
				>📏 {windowHeight}px</span
			>
			<span class="bg-[#1a1a1a] px-3 py-1.5 rounded-md text-[13px] font-semibold font-mono"
				>📜 {Math.round(scrollY)}px</span
			>
		</div>
	</header>

	<!-- Page navigation -->
	<nav class="bg-[#2a2a2a] px-6 py-4 flex gap-3 border-b border-[#3a3a3a]">
		{#each pages as page, i}
			<button
				class="bg-transparent border-2 border-[#3a3a3a] text-[#888] px-4 py-[10px] rounded-lg font-semibold cursor-pointer transition-all flex items-center gap-2 hover:border-[#4a9eff] hover:text-white"
				class:bg-[#4a9eff]={currentPage.id === page.id}
				class:border-[#4a9eff]={currentPage.id === page.id}
				class:text-black={currentPage.id === page.id}
				onclick={() => (currentPage = page)}
			>
				{page.title}
				<span class="text-[11px] opacity-60 bg-[rgba(0,0,0,0.3)] px-1.5 py-0.5 rounded"
					>⌘{i + 1}</span
				>
			</button>
		{/each}
	</nav>

	<!-- Page content -->
	<main class="py-10 px-6 max-w-[1200px] mx-auto">
		<h2 class="m-0 mb-2 text-white text-[32px]">{currentPage.title}</h2>
		<p class="m-0 mb-8 text-[#888] text-lg">{currentPage.description}</p>

		<div class="grid grid-cols-[repeat(auto-fit,minmax(280px,1fr))] gap-5 mb-10">
			<div class="bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl p-6">
				<h3 class="m-0 mb-4 text-[#4a9eff] text-lg">Device Info</h3>
				<p class="my-2 text-[#ccc] text-sm">Type: {isMobile ? '📱 Mobile' : '💻 Desktop'}</p>
				<p class="my-2 text-[#ccc] text-sm">Width: {windowWidth}px</p>
				<p class="my-2 text-[#ccc] text-sm">Height: {windowHeight}px</p>
			</div>

			<div class="bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl p-6">
				<h3 class="m-0 mb-4 text-[#4a9eff] text-lg">Scroll Position</h3>
				<p class="my-2 text-[#ccc] text-sm">Y: {Math.round(scrollY)}px</p>
				<p class="my-2 text-[#ccc] text-sm">Status: {isScrolled ? 'Scrolled' : 'Top of page'}</p>
			</div>

			<div class="bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl p-6">
				<h3 class="m-0 mb-4 text-[#4a9eff] text-lg">Keyboard Shortcuts</h3>
				<p class="my-2 text-[#ccc] text-sm">⌘K - Quick search</p>
				<p class="my-2 text-[#ccc] text-sm">⌘1-4 - Switch pages</p>
			</div>
		</div>

		<!-- Add some content to enable scrolling -->
		<div class="h-[1000px]"></div>
	</main>

	<footer class="bg-[#2a2a2a] border-t border-[#3a3a3a] px-6 py-5 text-center">
		<p class="m-0 text-[#888] text-sm">💡 Try keyboard shortcuts! Press ⌘K or ⌘1-4</p>
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
	onError((err) => {
		error = {
			message: err.message || 'An unknown error occurred',
			stack: err.stack,
			timestamp: new Date()
		};
		console.error('Error caught by boundary:', err);

		// Return true to prevent error from bubbling up
		return true;
	});

	function reset() {
		error = null;
	}
</script>

{#if error}
	<div
		class="bg-[#2a1a1a] border-2 border-[#ff6b6b] rounded-xl p-10 text-center max-w-[600px] mx-auto my-10"
	>
		<div class="text-[64px] mb-4 animate-[shake_0.5s]">⚠️</div>
		<h2 class="m-0 mb-4 text-[#ff6b6b] text-[28px]">Something went wrong</h2>
		<p class="text-[#ccc] m-0 mb-3 text-base leading-[1.6]">{error.message}</p>
		<p class="text-[#888] text-[13px] m-0 mb-6">
			Occurred at {error.timestamp.toLocaleTimeString()}
		</p>
		{#if error.stack}
			<details class="bg-[#1a1a1a] border border-[#3a3a3a] rounded-lg p-4 text-left mb-6">
				<summary class="text-[#4a9eff] cursor-pointer font-semibold mb-3">Technical Details</summary
				>
				<pre
					class="text-[#ff6b6b] text-xs overflow-x-auto mt-3 mb-0 leading-[1.4]">{error.stack}</pre>
			</details>
		{/if}
		<button
			onclick={reset}
			class="bg-[#4a9eff] text-black border-none px-8 py-[14px] rounded-lg text-base font-bold cursor-pointer transition-all hover:bg-[#6ab0ff] hover:-translate-y-0.5"
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

<div class="bg-[#1a1a1a] min-h-screen py-10 px-5 text-[#e0e0e0]">
	<h1 class="text-center text-[#4a9eff] m-0 mb-8">User Dashboard</h1>

	<button
		onclick={loadData}
		class="block mx-auto mb-8 bg-[#4ade80] text-black border-none px-8 py-3 rounded-lg font-semibold cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
		disabled={loading}
	>
		{loading ? 'Loading...' : 'Load User Data'}
	</button>

	<ErrorBoundary>
		{#if userData}
			<div class="bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl p-6 max-w-[400px] mx-auto">
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
	class="bg-[#1a1a1a] min-h-screen py-6 px-5 text-[#e0e0e0]"
	bind:clientWidth={containerWidth}
	bind:clientHeight={containerHeight}
>
	<div class="mb-6">
		<h2 class="m-0 mb-3 text-[#4a9eff] text-[28px]">Responsive Product Grid</h2>
		<div class="flex gap-4 flex-wrap">
			<span class="bg-[#2a2a2a] px-4 py-2 rounded-md text-sm font-semibold border border-[#3a3a3a]"
				>Width: {containerWidth}px</span
			>
			<span class="bg-[#2a2a2a] px-4 py-2 rounded-md text-sm font-semibold border border-[#3a3a3a]"
				>Height: {containerHeight}px</span
			>
			<span class="bg-[#2a2a2a] px-4 py-2 rounded-md text-sm font-semibold border border-[#3a3a3a]"
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
				class="bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl p-5 text-center transition-all hover:border-[#4a9eff] hover:-translate-y-1"
			>
				<div class="text-5xl mb-3">{product.image}</div>
				<h3 class="m-0 mb-2 text-white">{product.name}</h3>
				<p class="m-0 text-[#4ade80] text-xl font-bold">${product.price}</p>
			</div>
		{/each}
	</div>

	<div class="mb-4">
		<h3 class="text-[#4a9eff] mb-4">Chart Area</h3>
		<div
			class="bg-[#2a2a2a] border-2 border-dashed border-[#3a3a3a] rounded-xl mx-auto transition-all"
			style="width: {chartSize.width}px; height: {chartSize.height}px;"
		>
			<div
				class="w-full h-full flex flex-col items-center justify-center text-[#888] text-lg text-center"
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
	class="bg-[#2a2a2a] rounded-xl overflow-hidden border border-[#3a3a3a] max-w-[900px] mx-auto my-10"
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
			class="absolute inset-0 flex items-center justify-center bg-[rgba(0,0,0,0.3)] opacity-0 transition-opacity cursor-pointer hover:opacity-100"
		>
			<button
				onclick={togglePlay}
				class="bg-[rgba(74,158,255,0.9)] border-none w-20 h-20 rounded-full text-[32px] cursor-pointer transition-all hover:bg-[#4a9eff] hover:scale-110"
			>
				{paused ? '▶️' : '⏸️'}
			</button>
		</div>
	</div>

	<!-- Control bar -->
	<div class="bg-[#1a1a1a] px-5 py-4">
		<!-- Progress bar -->
		<div class="relative mb-4">
			<input
				type="range"
				min="0"
				max={duration}
				bind:value={currentTime}
				class="absolute -top-2 left-0 w-full h-5 opacity-0 cursor-pointer"
			/>
			<div class="h-1.5 bg-[#3a3a3a] rounded-[3px] overflow-hidden">
				<div class="h-full bg-[#4a9eff] transition-[width_0.1s]" style="width: {progress}%"></div>
			</div>
		</div>

		<!-- Main controls -->
		<div class="flex justify-between items-center gap-5 mb-3">
			<!-- Left side: Play, skip, time -->
			<div class="flex items-center gap-3">
				<button
					onclick={togglePlay}
					class="bg-[#2a2a2a] border-2 border-[#3a3a3a] text-white px-4 py-2 rounded-md text-sm font-semibold cursor-pointer transition-all hover:border-[#4a9eff] hover:bg-[#3a3a3a]"
				>
					{paused ? '▶️' : '⏸️'}
				</button>
				<button
					onclick={() => skip(-10)}
					class="bg-[#2a2a2a] border-2 border-[#3a3a3a] text-white px-4 py-2 rounded-md text-sm font-semibold cursor-pointer transition-all hover:border-[#4a9eff] hover:bg-[#3a3a3a]"
				>
					⏪ 10s
				</button>
				<button
					onclick={() => skip(10)}
					class="bg-[#2a2a2a] border-2 border-[#3a3a3a] text-white px-4 py-2 rounded-md text-sm font-semibold cursor-pointer transition-all hover:border-[#4a9eff] hover:bg-[#3a3a3a]"
				>
					10s ⏩
				</button>
				<span class="text-[#ccc] text-sm font-semibold font-mono min-w-[110px]">
					{formatTime(currentTime)} / {formatTime(duration)}
				</span>
			</div>

			<!-- Right side: Speed, volume -->
			<div class="flex items-center gap-3">
				<!-- Speed control -->
				<select
					bind:value={playbackRate}
					class="bg-[#2a2a2a] border-2 border-[#3a3a3a] text-white px-3 py-2 rounded-md text-sm cursor-pointer"
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
						class="w-[100px] accent-[#4a9eff]"
					/>
					<span class="text-[#888] text-[13px] font-semibold min-w-[40px]"
						>{Math.round(volume * 100)}%</span
					>
				</div>
			</div>
		</div>

		<!-- Time remaining indicator -->
		<div class="text-center text-[#888] text-[13px]">
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

<div class="bg-[#2a2a2a] border border-[#3a3a3a] rounded-xl p-6 mb-4">
	<h3 class="m-0 mb-4 text-[#4a9eff] text-xl">Instance: {instanceId}</h3>
	<div class="flex flex-col gap-3">
		<div class="bg-[#1a1a1a] p-3 rounded-md flex justify-between items-center">
			<span class="text-[#888] text-sm font-semibold">Total Instances:</span>
			<span class="text-[#4ade80] text-sm font-bold">{totalInstances}</span>
		</div>
		<div class="bg-[#1a1a1a] p-3 rounded-md flex justify-between items-center">
			<span class="text-[#888] text-sm font-semibold">All Instances:</span>
			<span class="text-[#4ade80] text-sm font-bold">{instanceRegistry.join(', ')}</span>
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
	class="bg-[#2a2a2a] border-2 border-[#3a3a3a] text-[#ccc] px-4 py-[10px] rounded-[20px] text-sm font-semibold cursor-pointer transition-all flex items-center gap-2 hover:border-[#4a9eff] hover:bg-[#3a3a3a]"
	class:bg-[#4a9eff]={isSelected}
	class:border-[#4a9eff]={isSelected}
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

<div class="bg-[#1a1a1a] py-8 px-8 rounded-xl max-w-[700px] mx-auto my-10">
	<h2 class="m-0 mb-6 text-[#4a9eff]">Filter Products</h2>

	<div class="flex flex-wrap gap-3 mb-6">
		<FilterChip id="electronics" label="Electronics" icon="💻" />
		<FilterChip id="clothing" label="Clothing" icon="👕" />
		<FilterChip id="books" label="Books" icon="📚" />
		<FilterChip id="sports" label="Sports" icon="⚽" />
		<FilterChip id="home" label="Home" icon="🏠" />
		<FilterChip id="toys" label="Toys" icon="🎮" />
	</div>

	<div class="flex justify-between items-center pt-6 border-t border-[#3a3a3a]">
		<div class="text-[#888] text-sm font-semibold">
			{selectedCount} filter{selectedCount === 1 ? '' : 's'} selected
		</div>
		<button
			onclick={clearAllFilters}
			class="bg-[#ff6b6b] text-white border-none px-5 py-[10px] rounded-lg font-semibold cursor-pointer"
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
	class="bg-[#2a2a2a] border-2 border-[#3a3a3a] rounded-xl overflow-hidden transition-all"
	class:border-[#4a9eff]={isCurrentlyPlaying}
	class:shadow-[0_4px_16px_rgba(74,158,255,0.3)]={isCurrentlyPlaying}
>
	<div class="p-5 flex justify-between items-center border-b border-[#3a3a3a]">
		<div>
			<h3 class="m-0 mb-1 text-white text-lg">{title}</h3>
			<span class="text-[#888] text-[13px]">⏱️ {duration}</span>
		</div>
		{#if isCurrentlyPlaying}
			<span
				class="bg-[#4a9eff] text-black px-3 py-1.5 rounded-md text-xs font-bold animate-[pulse_2s_infinite]"
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
			class="bg-[#4a9eff] text-black border-none px-5 py-[10px] rounded-md font-bold cursor-pointer transition-all whitespace-nowrap hover:bg-[#6ab0ff] hover:scale-105"
		>
			{paused ? '▶️ Play' : '⏸️ Pause'}
		</button>

		<div class="flex-1 h-1.5 bg-[#3a3a3a] rounded-[3px] overflow-hidden">
			<div class="h-full bg-[#4a9eff] transition-[width_0.1s]" style="width: {progress}%"></div>
		</div>

		<span class="text-[#888] text-[13px] font-semibold font-mono min-w-[100px] text-right">
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

<div class="bg-[#1a1a1a] min-h-screen py-10 px-5">
	<div class="max-w-[900px] mx-auto mb-8 flex justify-between items-center">
		<h1 class="m-0 text-[#4a9eff] text-[32px]">📚 Svelte 5 Mastery Course</h1>
		<button
			onclick={pauseAll}
			class="bg-[#ff6b6b] text-white border-none px-6 py-3 rounded-lg font-bold cursor-pointer"
		>
			⏸️ Pause All Videos
		</button>
	</div>

	<div class="max-w-[900px] mx-auto flex flex-col gap-6">
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

## 🚀 Next Steps

After mastering this section, you'll be ready for:

- **Section 8**: Animations and transitions
- Building production-ready applications
- Integrating any third-party library
- Creating accessible, responsive UIs
