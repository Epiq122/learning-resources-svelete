# Section 6: Context API & Konva Integration Study Guide

**Complete lesson plan for learning Svelte 5 Context API and imperative library integration**

---

## 📚 Learning Objectives

By the end of this section, you will:

- ✅ Understand what the Context API is and when to use it
- ✅ Share state between components without prop drilling
- ✅ Handle missing context gracefully
- ✅ Encapsulate context logic for cleaner code
- ✅ Integrate imperative libraries (like Konva) with Svelte
- ✅ Create wrapper components for third-party libraries
- ✅ Handle events from imperative libraries
- ✅ Support reactive prop updates and bindings
- ✅ Expose component internals using exports

---

## Table of Contents

1. [Context API Basics](#1-context-api-basics)
2. [Sharing Scoped State](#2-sharing-scoped-state)
3. [Handling Missing Context](#3-handling-missing-context)
4. [Encapsulating Context Functions](#4-encapsulating-context-functions)
5. [Konva Setup](#5-konva-setup)
6. [Stage Component](#6-stage-component)
7. [Layer Component](#7-layer-component)
8. [Rect Component](#8-rect-component)
9. [Event Handling](#9-event-handling)
10. [Reactive Props](#10-reactive-props)
11. [Prop Binding](#11-prop-binding)
12. [Component Exports](#12-component-exports)

---

## 1. Context API Basics

### What is the Context API?

The Context API solves **prop drilling** - passing data through many component layers.

**Real-World Scenario:** You're building an admin dashboard. The logged-in user's info (name, permissions, avatar) needs to be available in:

- The header (to show user name)
- The sidebar (to show/hide admin options)
- Individual pages (to check permissions)
- Forms (to auto-fill "created by")

Without context, you'd pass `user` as props through 5+ component layers. With context, you set it once at the top and any component can access it!

### 📁 Files to Create

**Context Provider Component:**

- `src/lib/components/AuthDashboard.svelte` - Sets context for auth user

**Child Components:**

- `src/lib/components/BlogPostCard.svelte` - Consumes auth context
- `src/lib/components/UserProfileMenu.svelte` - Shows current user

**Demo Page:**

- `src/routes/context-demo/+page.svelte` - Assembles the demo

**Best Practices:**

- Set context in a parent/layout component
- Get context in any child component (any depth)
- Use TypeScript interfaces for type safety
- Context is scoped to component tree (not global)

**The Problem:\*\***

```svelte
<!-- ❌ Prop drilling - passing user through every single component -->
<Dashboard {user}>
	<Header {user}>
		<UserMenu {user} />
		<!-- Needs user -->
	</Header>
	<Sidebar {user}>
		<Navigation {user} />
		<!-- Needs user to show admin links -->
	</Sidebar>
	<MainContent {user}>
		<ArticleList {user}>
			<ArticleCard {user} />
			<!-- Needs user to show edit button -->
		</ArticleList>
	</MainContent>
</Dashboard>
```

**The Solution:**

```svelte
<!-- ✅ Context API - set once, use anywhere -->
<Dashboard {user}>
	<!-- Sets context here -->
	<Header>
		<UserMenu />
		<!-- Gets user from context -->
	</Header>
	<Sidebar>
		<Navigation />
		<!-- Gets user from context -->
	</Sidebar>
	<MainContent>
		<ArticleList>
			<ArticleCard />
			<!-- Gets user from context -->
		</ArticleList>
	</MainContent>
</Dashboard>
```

### When to Use Context

✅ **Use Context When:**

- **Authentication:** Current user info needed everywhere
- **Theme/Settings:** Dark mode, language, font size across all components
- **Shopping Cart:** Cart state accessed by header, product pages, checkout
- **Form Systems:** Parent form manages validation, children are inputs
- **Component Libraries:** Tabs, Accordion, Dialog - parent controls children

❌ **Don't Use Context When:**

- Only 1-2 components need the data (just pass props)
- You need truly global state across pages (use stores instead)
- Data doesn't follow parent-child relationship

---

### Real-World Example: User Authentication Dashboard

**Scenario:** Building a blog CMS where multiple components need to know:

- Who's logged in
- Their role (author, editor, admin)
- Whether they can edit/delete posts

### Component: AuthDashboard.svelte

**What it does:** Authenticates a user and provides their info to all child components via context. This is the root of your app after login.

```svelte
<script lang="ts">
	import { setContext } from 'svelte';
	import type { Snippet } from 'svelte';

	// Define the authenticated user structure
	interface AuthUser {
		id: string;
		name: string;
		email: string;
		role: 'author' | 'editor' | 'admin';
		avatar: string;
	}

	let { children }: { children: Snippet } = $props();

	// In a real app, this would come from your auth service/API
	let user = $state<AuthUser>({
		id: '123',
		name: 'Sarah Johnson',
		email: 'sarah@example.com',
		role: 'editor',
		avatar: '👩‍💼'
	});

	// Provide user to all children - they can access without props
	setContext('auth-user', user);
</script>

<div class="bg-gray-900 text-gray-200 min-h-screen">
	<div class="bg-gray-800 border-b-2 border-gray-700 py-4 px-6 flex justify-between items-center">
		<h1 class="m-0 text-blue-400 text-2xl font-bold">Blog CMS Dashboard</h1>
		<div class="flex items-center gap-3 bg-gray-900 py-2 px-4 rounded-lg border border-gray-700">
			<span class="text-3xl">{user.avatar}</span>
			<div class="flex flex-col gap-0.5">
				<span class="text-white font-semibold text-sm">{user.name}</span>
				<span
					class="bg-blue-400 text-black py-0.5 px-2 rounded text-xs font-semibold uppercase w-fit"
					>{user.role}</span
				>
			</div>
		</div>
	</div>

	<!-- All child components can now access user via context -->
	{@render children()}
</div>
```

### Component: BlogPostCard.svelte

**What it does:** Displays a blog post with edit/delete buttons ONLY if the logged-in user has permission. Gets user from context without any props!

```svelte
<script lang="ts">
	import { getContext } from 'svelte';

	// Define the user type (same as parent)
	interface AuthUser {
		id: string;
		name: string;
		email: string;
		role: 'author' | 'editor' | 'admin';
		avatar: string;
	}

	// Props for the blog post itself
	interface Props {
		post: {
			id: string;
			title: string;
			excerpt: string;
			authorId: string;
			authorName: string;
		};
	}

	let { post }: Props = $props();

	// Get the logged-in user from context (no prop drilling!)
	let user = getContext<AuthUser>('auth-user');

	// Derived: Can this user edit/delete this post?
	const isAuthor = $derived(post.authorId === user.id);
	const isEditorOrAdmin = $derived(user.role === 'editor' || user.role === 'admin');
	const canEdit = $derived(isAuthor || isEditorOrAdmin);
	const canDelete = $derived(user.role === 'admin');

	function handleEdit() {
		console.log(`${user.name} is editing post: ${post.title}`);
	}

	function handleDelete() {
		console.log(`${user.name} is deleting post: ${post.title}`);
	}
</script>

<article
	class="bg-gray-800 p-5 rounded-lg border border-gray-700 transition-[border-color] duration-200 hover:border-blue-400"
>
	<h3 class="m-0 mb-3 text-white text-xl">{post.title}</h3>
	<p class="m-0 mb-4 text-gray-300 leading-relaxed">{post.excerpt}</p>
	<div class="flex justify-between items-center gap-3">
		<span class="text-gray-500 text-sm">By {post.authorName}</span>
		{#if canEdit || canDelete}
			<div class="flex gap-2">
				{#if canEdit}
					<button
						onclick={handleEdit}
						class="border-none py-1.5 px-3 rounded-md text-xs font-semibold cursor-pointer transition-all duration-200 bg-blue-400 text-black hover:bg-blue-300 hover:-translate-y-px"
					>
						✏️ Edit
					</button>
				{/if}
				{#if canDelete}
					<button
						onclick={handleDelete}
						class="border-none py-1.5 px-3 rounded-md text-xs font-semibold cursor-pointer transition-all duration-200 bg-red-400 text-white hover:bg-red-300 hover:-translate-y-px"
					>
						🗑️ Delete
					</button>
				{/if}
			</div>
		{/if}
	</div>
</article>
```

**Key Concepts:**

- **Context flows down:** Parent sets, any descendant can get
- **No prop drilling:** BlogPostCard doesn't need user passed as prop
- **Type safety:** TypeScript ensures type matches between set/get
- **Real-world use:** Check permissions, show conditional UI
- **Key must match:** Both use `'auth-user'` as the key

---

## 2. Sharing Scoped State

### What is Scoped State?

Context creates **tree-scoped state** - only available to children of the component that sets it. Multiple instances can have different values!

### Component: ThemeProvider.svelte

**What it does:** Provides theme state to all children, with reactive updates when theme changes.

```svelte
<script lang="ts">
	import { setContext } from 'svelte';

	// Define the context interface
	interface ThemeContext {
		current: string;
		toggle: () => void;
	}

	// Reactive theme state
	let theme = $state<'dark' | 'light'>('dark');

	// Create context object with getter and method
	// Using a getter ensures reactivity - children get the latest value
	setContext<ThemeContext>('theme', {
		get current() {
			return theme; // Getter ensures this always returns the current value
		},
		toggle() {
			theme = theme === 'dark' ? 'light' : 'dark';
		}
	});

	let { children }: { children: Snippet } = $props();
</script>

<div
	class="min-h-screen p-5 transition-all duration-300 ease-in-out"
	class:bg-[#1a1a1a]={theme !== 'light'}
	class:text-[#e0e0e0]={theme !== 'light'}
	class:bg-[#f0f0f0]={theme === 'light'}
	class:text-[#1a1a1a]={theme === 'light'}
>
	{@render children()}
</div>
```

> **Note:** Add `import type { Snippet } from 'svelte';` at the top of the script.

### Component: ThemeButton.svelte

**What it does:** Accesses theme context and allows toggling between dark/light modes.

```svelte
<script lang="ts">
	import { getContext } from 'svelte';

	interface ThemeContext {
		current: string;
		toggle: () => void;
	}

	// Get the theme context
	let theme = getContext<ThemeContext>('theme');
</script>

<button class="py-3 px-6 rounded-lg text-base font-semibold cursor-pointer transition-all duration-300 ease-in-out flex items-center gap-2" class:bg-gray-800={theme.current !== 'light'} class:text-white={theme.current !== 'light'} class:border-2={theme.current !== 'light'} class:border-blue-400={theme.current !== 'light'} class:hover:bg-gray-700={theme.current !== 'light'} class:hover:border-blue-300={theme.current !== 'light'} class:hover:-translate-y-0.5={theme.current !== 'light'} class:bg-gray-100={theme.current === 'light'} class:text-gray-900={theme.current === 'light'} class:border-2={theme.current === 'light'} class:border-orange-500={theme.current === 'light'} class:hover:bg-gray-200={theme.current === 'light'} onclick={theme.toggle}>
	{theme.current === 'dark' ? '🌙' : '☀️'}
	{theme.current === 'dark' ? 'Dark' : 'Light'} Mode
</button>
```

**Key Concepts:**

- Use **getters** (`get current()`) for reactive values
- Use **methods** (`toggle()`) for actions
- Context updates automatically propagate to all children
- Each provider instance maintains its own state

---

## 3. Handling Missing Context

### Why Handle Missing Context?

If a component uses `getContext()` but no parent set that context, it returns `undefined`. This causes errors!

### Component: SafeContextUser.svelte

**What it does:** Safely handles the case where context might not exist, showing an error message instead of crashing.

```svelte
<script lang="ts">
	import { getContext } from 'svelte';

	interface User {
		name: string;
		role: string;
	}

	// getContext returns the value OR undefined if not found
	let user = getContext<User | undefined>('user');

	// Type guard: Check if user exists before using it
	const hasUser = user !== undefined;
</script>

{#if hasUser}
	<div class="bg-[#1a2a1a] text-green-400 p-5 rounded-lg border-2 border-green-400">
		<h2 class="m-0 mb-3 text-lg">✅ Context Found</h2>
		<p class="m-0 mb-2">Welcome {user.name}!</p>
		<span class="bg-green-400 text-black px-3 py-1 rounded text-xs font-semibold">{user.role}</span>
	</div>
{:else}
	<div class="bg-[#2a1a1a] text-red-400 p-5 rounded-lg border-2 border-red-400">
		<h2 class="m-0 mb-3 text-lg">⚠️ No Context Found</h2>
		<p class="m-0 mb-2">This component must be used inside a context provider</p>
		<code class="block bg-[rgba(0,0,0,0.3)] p-2 rounded mt-3 font-mono"
			>&lt;BasicContext&gt;...&lt;/BasicContext&gt;</code
		>
	</div>
{/if}
```

**Key Concepts:**

- Always type context as `Type | undefined`
- Use `{#if}` blocks to check if context exists
- Provide helpful error messages for developers
- This makes your components more reusable and robust

---

## 4. Encapsulating Context Functions

### Why Encapsulate?

Instead of exposing raw context in every component, create a helper function. This provides:

- Better error messages
- Type safety
- Default values
- Consistent access pattern

### File: contexts/modal.svelte.ts

**What it does:** Creates a reusable modal context with a helper function for safe access.

```typescript
// contexts/modal.svelte.ts
import { getContext, setContext } from "svelte";

// Define the modal context interface
export interface ModalContext {
  isOpen: boolean;
  content: (() => any) | null;
  open: (content: () => any) => void;
  close: () => void;
}

// Unique symbol as key (prevents naming conflicts)
const MODAL_KEY = Symbol("modal");

// Helper function to CREATE modal context (used in provider)
export function createModalContext(): ModalContext {
  let isOpen = $state(false);
  let content = $state<(() => any) | null>(null);

  const context: ModalContext = {
    get isOpen() {
      return isOpen;
    },
    get content() {
      return content;
    },
    open(modalContent: () => any) {
      content = modalContent;
      isOpen = true;
    },
    close() {
      isOpen = false;
      content = null;
    },
  };

  setContext(MODAL_KEY, context);
  return context;
}

// Helper function to GET modal context (used in children)
export function getModalContext(): ModalContext {
  const context = getContext<ModalContext | undefined>(MODAL_KEY);

  if (!context) {
    throw new Error(
      "Modal context not found. " +
        "Make sure to use this component inside <ModalProvider>"
    );
  }

  return context;
}
```

### Component: ModalProvider.svelte

**What it does:** Provides modal context to children and renders the modal overlay.

```svelte
<script lang="ts">
	import { createModalContext } from './contexts/modal.svelte';
	import type { Snippet } from 'svelte';

	// Create and set the modal context
	const modal = createModalContext();

	let { children }: { children: Snippet } = $props();
</script>

<div class="bg-[#1a1a1a] min-h-screen p-5">
	<!-- App content -->
	{@render children()}
</div>

<!-- Modal overlay (shown when modal is open) -->
{#if modal.isOpen}
	<div
		class="fixed inset-0 bg-[rgba(0,0,0,0.8)] flex items-center justify-center z-[1000] backdrop-blur-[4px]"
		onclick={modal.close}
	>
		<div
			class="bg-[#2a2a2a] p-8 rounded-xl border border-[#3a3a3a] max-w-[600px] w-[90%] relative shadow-[0_20px_60px_rgba(0,0,0,0.5)] animate-[slideIn_0.3s_ease]"
			onclick={(e) => e.stopPropagation()}
		>
			<button
				class="absolute top-4 right-4 bg-transparent border-none text-[#888] text-2xl cursor-pointer px-2 py-1 leading-none transition-colors hover:text-white"
				onclick={modal.close}>✕</button
			>
			{#if modal.content}
				{@render modal.content()}
			{/if}
		</div>
	</div>
{/if}

<style>
	@keyframes slideIn {
		from {
			opacity: 0;
			transform: translateY(-20px);
		}
		to {
			opacity: 1;
			transform: translateY(0);
		}
	}
</style>
```

### Component: ProductCard.svelte

**What it does:** Uses the modal context to display product details in a modal.

```svelte
<script lang="ts">
	import { getModalContext } from './contexts/modal.svelte';

	interface Product {
		id: number;
		name: string;
		price: number;
		description: string;
	}

	// Props
	let { product }: { product: Product } = $props();

	// Get modal context using our helper function
	const modal = getModalContext();
</script>

<div class="bg-[#2a2a2a] p-5 rounded-lg border border-[#3a3a3a] transition-[transform,border-color] duration-200 hover:-translate-y-1 hover:border-[#4a9eff]">
	<h3 class="m-0 mb-2 text-white text-lg">{product.name}</h3>
	<p class="text-[#4ade80] text-xl font-semibold m-0 mb-4">${product.price.toFixed(2)}</p>
	<button
		onclick={() => modal.open(() => (
			<div class="text-white">
				<h2 class="m-0 mb-4 text-[#4a9eff]">{product.name}</h2>
				<p class="text-[#ccc] leading-relaxed mb-5">{product.description}</p>
				<p class="text-[#4ade80] text-[28px] font-bold m-0 mb-5">${product.price.toFixed(2)}</p>
				<button class="bg-[#4ade80] text-black border-none py-[14px] px-7 rounded-lg font-semibold text-base cursor-pointer w-full">Add to Cart</button>
			</div>
		))}
		class="bg-[#4a9eff] text-black border-none py-[10px] px-5 rounded-md cursor-pointer font-semibold w-full transition-colors hover:bg-[#6ab0ff]"
	>
		View Details
	</button>
</div>


```

**Key Concepts:**

- Use helper functions instead of raw `getContext()`
- Throw descriptive errors if context missing
- Use Symbols as keys to prevent conflicts
- Encapsulation makes code cleaner and more maintainable

---

## 5. Konva Setup

### What is Konva?

Konva is a 2D canvas library for creating interactive graphics, animations, and drawings. It's **imperative** (not reactive like Svelte), so we need to integrate it carefully.

### Installation

```bash
npm install konva
npm install --save-dev @types/konva
```

### Why Wrap Konva in Svelte Components?

1. **Reactivity** - Konva doesn't know about Svelte's reactive state
2. **Component-based** - Wrap Konva nodes as Svelte components
3. **Context** - Use context to pass stage/layer references to children
4. **Events** - Convert Konva events to Svelte events

### File: contexts/konva.svelte.ts

**What it does:** Creates context helpers for passing Konva stage and layer references through component tree.

```typescript
// contexts/konva.svelte.ts
import { getContext, setContext } from "svelte";
import type Konva from "konva";

// Stage context (the root Konva container)
const STAGE_KEY = Symbol("konva-stage");

export function setStageContext(stage: Konva.Stage) {
  setContext(STAGE_KEY, stage);
}

export function getStageContext(): Konva.Stage {
  const stage = getContext<Konva.Stage | undefined>(STAGE_KEY);
  if (!stage) {
    throw new Error("Stage context not found. Use <Stage> component");
  }
  return stage;
}

// Layer context (Konva layer within stage)
const LAYER_KEY = Symbol("konva-layer");

export function setLayerContext(layer: Konva.Layer) {
  setContext(LAYER_KEY, layer);
}

export function getLayerContext(): Konva.Layer {
  const layer = getContext<Konva.Layer | undefined>(LAYER_KEY);
  if (!layer) {
    throw new Error("Layer context not found. Use <Layer> component");
  }
  return layer;
}
```

**Key Concepts:**

- Konva has a hierarchy: Stage → Layer → Shapes
- We'll pass each level via context
- Type safety with TypeScript types from `@types/konva`

> 💡 **Best Practice**: For production Konva apps, organize components by feature:
>
> ```
> src/lib/components/konva/
> ├── Stage.svelte
> ├── Layer.svelte
> ├── shapes/
> │   ├── Rect.svelte
> │   ├── Circle.svelte
> │   └── Text.svelte
> └── index.ts (barrel exports)
> ```

**⚠️ Common Mistakes:**

- Don't forget to destroy Konva nodes on unmount - memory leaks!
- Don't skip `layer.draw()` after updates - nothing will render
- Don't try to use Svelte reactivity directly - use $effect to sync

---

## 6. Stage Component

### Component: Stage.svelte

**What it does:** Creates the Konva Stage (root container) and provides it via context to children.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { setStageContext } from './contexts/konva.svelte';

	// Props
	interface Props {
		width: number;
		height: number;
	}

	let { width, height }: Props = $props();

	// DOM reference for the container
	let container: HTMLDivElement;

	// Konva stage instance
	let stage: Konva.Stage;

	// Create the stage when component mounts
	onMount(() => {
		// Create Konva stage imperatively
		stage = new Konva.Stage({
			container: container, // DOM element to attach to
			width: width,
			height: height
		});

		// Provide stage to child components via context
		setStageContext(stage);

		// Cleanup: destroy stage when component unmounts
		return () => {
			stage.destroy();
		};
	});
</script>

<!-- Container div where Konva will attach its canvas -->
<div
	bind:this={container}
	class="bg-[#1a1a1a] border-2 border-[#3a3a3a] rounded-lg overflow-hidden shadow-[0_4px_12px_rgba(0,0,0,0.5)]"
	style="width: {width}px; height: {height}px;"
>
	<!-- Only render children after stage is created -->
	{#if stage}
		{@render children()}
	{/if}
</div>
```

> **Note:** Add `import type { Snippet } from 'svelte';` and `let { children, ...props }: { children: Snippet; width: number; height: number } = $props();` in the script section.

<style>
	/* Style the Konva canvas inside */
	:global(canvas) {
		display: block;
	}
</style>

````

**Key Concepts:**

- `onMount()` - Create Konva stage after DOM is ready
- `bind:this` - Get reference to DOM element
- Return cleanup function to destroy stage on unmount
- Only render children after stage exists
- Set context so children can access the stage

---

## 7. Layer Component

### Component: Layer.svelte

**What it does:** Creates a Konva Layer (container for shapes) and provides it via context.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { getStageContext, setLayerContext } from './contexts/konva.svelte';

	// Get parent stage from context
	const stage = getStageContext();

	// Layer instance
	let layer: Konva.Layer;

	onMount(() => {
		// Create layer
		layer = new Konva.Layer();

		// Add layer to stage
		stage.add(layer);

		// Provide layer to children via context
		setLayerContext(layer);

		// Initial render
		layer.draw();

		// Cleanup: remove layer when component unmounts
		return () => {
			layer.destroy();
		};
	});
</script>

<!-- Slot for child shapes -->
{#if layer}
	{@render children()}
{/if}
````

> **Note:** Add `import type { Snippet } from 'svelte';` and `let { children }: { children: Snippet } = $props();` in the script section.

**Key Concepts:**

- Get stage from context using helper function
- Create layer and add to stage
- Provide layer via context for child shapes
- Call `layer.draw()` to render
- Cleanup on unmount

---

## 8. Rect Component

### Component: Rect.svelte

**What it does:** Creates a Konva rectangle shape with reactive props.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { getLayerContext } from './contexts/konva.svelte';

	// Props
	interface Props {
		x: number;
		y: number;
		width: number;
		height: number;
		fill: string;
		draggable?: boolean;
	}

	let { x, y, width, height, fill, draggable = false }: Props = $props();

	// Get layer from context
	const layer = getLayerContext();

	// Rect instance
	let rect: Konva.Rect;

	onMount(() => {
		// Create rectangle
		rect = new Konva.Rect({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		// Add to layer
		layer.add(rect);

		// Render
		layer.draw();

		// Cleanup
		return () => {
			rect.destroy();
		};
	});
</script>
```

**Key Concepts:**

- Get layer from context
- Create shape with initial props
- Add shape to layer
- Remember to call `layer.draw()` to render
- Destroy shape on unmount

---

## 9. Event Handling

### Component: RectWithEvents.svelte

**What it does:** Adds click, drag, and hover events to a Konva rectangle.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { getLayerContext } from './contexts/konva.svelte';

	interface Props {
		x: number;
		y: number;
		width: number;
		height: number;
		fill: string;
		draggable?: boolean;
		onclick?: (e: Konva.KonvaEventObject<MouseEvent>) => void;
		ondragend?: (e: Konva.KonvaEventObject<DragEvent>) => void;
	}

	let { x, y, width, height, fill, draggable = false, onclick, ondragend }: Props = $props();

	const layer = getLayerContext();
	let rect: Konva.Rect;

	onMount(() => {
		rect = new Konva.Rect({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		// Add click event
		if (onclick) {
			rect.on('click', onclick);
		}

		// Add drag end event
		if (ondragend) {
			rect.on('dragend', ondragend);
		}

		// Add hover effect
		rect.on('mouseenter', () => {
			document.body.style.cursor = 'pointer';
			rect.stroke('#4a9eff');
			rect.strokeWidth(3);
			layer.draw();
		});

		rect.on('mouseleave', () => {
			document.body.style.cursor = 'default';
			rect.stroke(null);
			rect.strokeWidth(0);
			layer.draw();
		});

		layer.add(rect);
		layer.draw();

		return () => {
			rect.destroy();
		};
	});
</script>
```

**Key Concepts:**

- Use `rect.on(eventName, handler)` to add events
- Common events: `click`, `dblclick`, `mouseenter`, `mouseleave`, `dragstart`, `dragmove`, `dragend`
- Call `layer.draw()` after changes to re-render
- Pass event handlers as props from parent

---

## 10. Reactive Props

### Component: ReactiveRect.svelte

**What it does:** Updates Konva rectangle when Svelte props change.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { getLayerContext } from './contexts/konva.svelte';

	interface Props {
		x: number;
		y: number;
		width: number;
		height: number;
		fill: string;
		draggable?: boolean;
	}

	let { x, y, width, height, fill, draggable = false }: Props = $props();

	const layer = getLayerContext();
	let rect: Konva.Rect;

	onMount(() => {
		rect = new Konva.Rect({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		layer.add(rect);
		layer.draw();

		return () => {
			rect.destroy();
		};
	});

	// Watch for prop changes and update Konva
	$effect(() => {
		if (!rect) return;

		// Update Konva rect properties when Svelte props change
		rect.setAttrs({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		// Re-render
		layer.draw();
	});
</script>
```

**Key Concepts:**

- Use `$effect()` to watch prop changes
- Use `rect.setAttrs()` to update multiple properties at once
- Always call `layer.draw()` after updates
- Check if rect exists before updating

---

## 11. Prop Binding

### Component: BindableRect.svelte

**What it does:** Supports two-way binding - when rectangle is dragged, update parent's x/y values.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { getLayerContext } from './contexts/konva.svelte';

	interface Props {
		x: number;
		y: number;
		width: number;
		height: number;
		fill: string;
		draggable?: boolean;
	}

	let {
		x = $bindable(),
		y = $bindable(),
		width,
		height,
		fill,
		draggable = false
	}: Props = $props();

	const layer = getLayerContext();
	let rect: Konva.Rect;

	onMount(() => {
		rect = new Konva.Rect({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		// When rect is dragged, update the bound props
		rect.on('dragmove', () => {
			x = rect.x();
			y = rect.y();
		});

		layer.add(rect);
		layer.draw();

		return () => {
			rect.destroy();
		};
	});

	// Update rect when props change
	$effect(() => {
		if (!rect) return;

		rect.setAttrs({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		layer.draw();
	});
</script>
```

**Usage:**

```svelte
<script lang="ts">
	let rectX = $state(50);
	let rectY = $state(50);
</script>

<Stage width={800} height={600}>
	<Layer>
		<!-- Bind x and y - they update when rect is dragged -->
		<BindableRect
			bind:x={rectX}
			bind:y={rectY}
			width={100}
			height={100}
			fill="#4a9eff"
			draggable={true}
		/>
	</Layer>
</Stage>

<p>X: {rectX}, Y: {rectY}</p>
```

**Key Concepts:**

- Use `$bindable()` to mark props as bindable
- Listen to Konva events (like `dragmove`) and update the prop
- This creates two-way binding between Svelte and Konva

---

## 12. Component Exports

### Component: ExportableRect.svelte

**What it does:** Exports the Konva node reference so parent components can access it directly.

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import Konva from 'konva';
	import { getLayerContext } from './contexts/konva.svelte';

	interface Props {
		x: number;
		y: number;
		width: number;
		height: number;
		fill: string;
		draggable?: boolean;
	}

	let { x, y, width, height, fill, draggable = false }: Props = $props();

	const layer = getLayerContext();

	// Bindable prop so parent can access the rect
	let rect = $state<Konva.Rect | undefined>(undefined);

	onMount(() => {
		rect = new Konva.Rect({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		layer.add(rect);
		layer.draw();

		return () => {
			rect?.destroy();
			rect = undefined;
		};
	});

	$effect(() => {
		if (!rect) return;

		rect.setAttrs({
			x,
			y,
			width,
			height,
			fill,
			draggable
		});

		layer.draw();
	});
</script>
```

**Usage:**

```svelte
<script lang="ts">
	import Konva from 'konva';
	import ExportableRect from './ExportableRect.svelte';

	// Bind to the exported rect
	let myRect: Konva.Rect | undefined;

	function animateRect() {
		if (!myRect) return;

		// Directly manipulate the Konva node
		myRect.to({
			rotation: 360,
			duration: 2,
			onFinish: () => {
				myRect.rotation(0);
			}
		});
	}
</script>

<button
	onclick={animateRect}
	class="bg-[#4a9eff] text-black border-none py-3 px-6 rounded-lg font-semibold cursor-pointer mb-4"
	>Animate</button
>

<Stage width={800} height={600}>
	<Layer>
		<ExportableRect bind:rect={myRect} x={100} y={100} width={100} height={100} fill="#4a9eff" />
	</Layer>
</Stage>
```

**Key Concepts:**

- Expose internal state for parent component access
- Parent can interact with component internals when needed
- Gives direct access to Konva node for advanced manipulation
- Useful for animations, tweens, or imperative operations

---

## Complete Example: Interactive Drawing App

### App.svelte

```svelte
<script lang="ts">
	import Stage from './Stage.svelte';
	import Layer from './Layer.svelte';
	import BindableRect from './BindableRect.svelte';

	interface Rectangle {
		id: number;
		x: number;
		y: number;
		width: number;
		height: number;
		fill: string;
	}

	let rectangles = $state<Rectangle[]>([
		{ id: 1, x: 50, y: 50, width: 100, height: 100, fill: '#4a9eff' },
		{ id: 2, x: 200, y: 100, width: 120, height: 80, fill: '#4ade80' },
		{ id: 3, x: 400, y: 150, width: 80, height: 120, fill: '#ff6b6b' }
	]);

	let selectedId = $state<number | null>(null);

	function addRectangle() {
		const newRect: Rectangle = {
			id: Date.now(),
			x: Math.random() * 600,
			y: Math.random() * 400,
			width: 100,
			height: 100,
			fill: `hsl(${Math.random() * 360}, 70%, 60%)`
		};
		rectangles = [...rectangles, newRect];
	}

	function deleteSelected() {
		if (selectedId === null) return;
		rectangles = rectangles.filter((r) => r.id !== selectedId);
		selectedId = null;
	}
</script>

<div class="bg-[#1a1a1a] min-h-screen p-10 text-[#e0e0e0]">
	<div class="mb-6">
		<h1 class="m-0 mb-4 text-[#4a9eff] text-[32px]">Interactive Konva Drawing</h1>
		<div class="flex gap-3 mb-4">
			<button
				onclick={addRectangle}
				class="bg-[#4ade80] text-black border-none py-3 px-6 rounded-lg font-semibold text-base cursor-pointer transition-all hover:bg-[#5ef090] hover:-translate-y-0.5"
			>
				➕ Add Rectangle
			</button>
			<button
				onclick={deleteSelected}
				class="bg-[#ff6b6b] text-white border-none py-3 px-6 rounded-lg font-semibold text-base cursor-pointer transition-all hover:bg-[#ff8787] hover:-translate-y-0.5 disabled:opacity-50 disabled:cursor-not-allowed disabled:hover:translate-y-0 disabled:hover:bg-[#ff6b6b]"
				disabled={selectedId === null}
			>
				🗑️ Delete Selected
			</button>
		</div>

		{#if selectedId !== null}
			{@const selected = rectangles.find((r) => r.id === selectedId)}
			{#if selected}
				<div class="bg-[#2a2a2a] p-4 rounded-lg border border-[#3a3a3a]">
					<p class="my-1 text-[#ccc]">
						<strong class="text-[#4a9eff]">Selected:</strong> Rectangle #{selectedId}
					</p>
					<p class="my-1 text-[#ccc]">
						Position: ({Math.round(selected.x)}, {Math.round(selected.y)})
					</p>
					<p class="my-1 text-[#ccc]">Size: {selected.width} × {selected.height}</p>
				</div>
			{/if}
		{/if}
	</div>

	<Stage width={800} height={600}>
		<Layer>
			{#each rectangles as rect (rect.id)}
				<BindableRect
					bind:x={rect.x}
					bind:y={rect.y}
					width={rect.width}
					height={rect.height}
					fill={rect.fill}
					draggable={true}
					onclick={() => (selectedId = rect.id)}
				/>
			{/each}
		</Layer>
	</Stage>
</div>
```

---

## 📝 Key Takeaways

✅ Context API solves prop drilling for nested component trees
✅ Use getters for reactive values, methods for actions
✅ Always handle missing context gracefully
✅ Encapsulate context logic in helper functions
✅ Konva integration requires wrapping imperative API in Svelte components
✅ Use `$effect()` to sync Svelte props with Konva properties
✅ Use `$bindable()` for two-way binding
✅ Export component internals when parent needs direct access

---

## 🚀 Next Steps

After mastering this section, you'll be ready for:

- **Section 7**: Actions, special elements, and advanced bindings
- **Section 8**: Animations and transitions
- Building complex component libraries
- Integrating other imperative libraries (D3, Three.js, etc.)
