# Section 1: Fundamentals & Reactivity

## 📚 Learning Objectives

By the end of this section, you will:

- Understand what Svelte is and how it differs from React
- Master signal-based reactivity with `$state`, `$derived`, and `$effect`
- Know when to use effects vs. derived state
- Build reactive components with proper state management
- Understand pure functions and the microtask queue

---

## Table of Contents

- [Section 1: Fundamentals \& Reactivity](#section-1-fundamentals--reactivity)
	- [📚 Learning Objectives](#-learning-objectives)
	- [Table of Contents](#table-of-contents)
	- [1. What is Svelte?](#1-what-is-svelte)
		- [What is Svelte?](#what-is-svelte)
		- [Svelte Equivalent:](#svelte-equivalent)
	- [3. Svelte vs SvelteKit](#3-svelte-vs-sveltekit)
		- [What's the Difference?](#whats-the-difference)
	- [4. Pure Functions](#4-pure-functions)
		- [What are Pure Functions?](#what-are-pure-functions)
		- [📁 Files to Create](#-files-to-create)
	- [5. The Microtask Queue](#5-the-microtask-queue)
		- [What is the Microtask Queue?](#what-is-the-microtask-queue)
		- [📁 Files to Create](#-files-to-create-1)
	- [6. Component Basics](#6-component-basics)
		- [What are Svelte Components?](#what-are-svelte-components)
		- [📁 Files to Create](#-files-to-create-2)
		- [Usage:](#usage)
	- [7. Adding Reactivity with $state](#7-adding-reactivity-with-state)
		- [What is $state?](#what-is-state)
		- [📁 Files to Create](#-files-to-create-3)
	- [8. Signal-Based Reactivity Deep Dive](#8-signal-based-reactivity-deep-dive)
		- [What are Signals?](#what-are-signals)
		- [📁 Files to Create](#-files-to-create-4)
	- [9. Derived State with $derived](#9-derived-state-with-derived)
		- [What is $derived?](#what-is-derived)
		- [📁 Files to Create](#-files-to-create-5)
	- [10. Side Effects with $effect](#10-side-effects-with-effect)
		- [What is $effect?](#what-is-effect)
		- [📁 Files to Create](#-files-to-create-6)
	- [11. Complete Counter Component](#11-complete-counter-component)
		- [Building a Full-Featured Counter](#building-a-full-featured-counter)
		- [📁 Files to Create](#-files-to-create-7)
	- [📝 Key Takeaways](#-key-takeaways)
	- [🚀 Next Steps](#-next-steps)

---

## 1. What is Svelte?

### What is Svelte?

Svelte is a **compiler** that transforms your components into highly optimized vanilla JavaScript at **build time**. Unlike React or Vue, there's no virtual DOM or runtime overhead!

**Real-World Scenario:** You're building a high-performance dashboard that needs to update thousands of data points in real-time. Svelte's compiler approach means less JavaScript sent to the browser and faster updates.

**What it does:** Converts your declarative component code into imperative DOM operations.

> 💡 **Best Practice**: Svelte's compiler means you ship less JavaScript to the browser. A typical Svelte app is 30-40% smaller than equivalent React apps.

````svelte
<script lang="ts">
	// This gets compiled away! No runtime framework shipped.
	let count = $state(0);
</script>

<button
	class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg text-base font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
	onclick={() => count++}
>
	Clicked {count}
	{count === 1 ? 'time' : 'times'}
</button>
- **Less code**: Boilerplate is minimal
- **Better performance**: Smaller bundles, faster runtime

**⚠️ Common Mistakes:**
- Don't try to use React patterns in Svelte - they're fundamentally different
- Don't look for a virtual DOM - Svelte doesn't have one
- Don't import React hooks - Svelte has runes instead

**⚡ Performance Tips:**
- Svelte's compiler optimizes at build time, not runtime
- No reconciliation overhead means faster updates
- Smaller bundle sizes improve initial load time

---

## 2. Svelte vs React

### How Does Svelte Compare to React?

React uses a virtual DOM and runtime reconciliation. Svelte compiles components to optimized DOM operations.

**Real-World Scenario:** You're migrating from React and want to understand the differences.

**What it does:** Shows equivalent code in both frameworks.

### 📁 Files to Create

For this comparison, create:
- `src/routes/todo-comparison/+page.svelte` (for the Svelte version)

### React Example:

```tsx
import { useState } from 'react';

function TodoApp() {
	const [todos, setTodos] = useState([{ id: 1, text: 'Learn Svelte', done: false }]);
	const [input, setInput] = useState('');

	const addTodo = () => {
		if (!input.trim()) return;
		setTodos([
			...todos,
			{
				id: Date.now(),
				text: input,
				done: false
			}
		]);
		setInput('');
	};

	const toggle = (id) => {
		setTodos(todos.map((todo) => (todo.id === id ? { ...todo, done: !todo.done } : todo)));
	};

	return (
		<div className="app">
			<h1>Todo List (React)</h1>
			<div className="input-group">
				<input
					value={input}
					onChange={(e) => setInput(e.target.value)}
					onKeyDown={(e) => e.key === 'Enter' && addTodo()}
					placeholder="Add a todo..."
				/>
				<button onClick={addTodo}>Add</button>
			</div>
			<ul>
				{todos.map((todo) => (
					<li key={todo.id} className={todo.done ? 'done' : ''}>
						<input type="checkbox" checked={todo.done} onChange={() => toggle(todo.id)} />
						<span>{todo.text}</span>
					</li>
				))}
			</ul>
		</div>
	);
}
````

### Svelte Equivalent:

```svelte
<script lang="ts">
	interface Todo {
		id: number;
		text: string;
		done: boolean;
	}

	let todos = $state<Todo[]>([{ id: 1, text: 'Learn Svelte', done: false }]);
	let input = $state('');

	function addTodo() {
		if (!input.trim()) return;
		todos.push({ id: Date.now(), text: input, done: false });
		input = '';
	}

	function toggle(id: number) {
		const todo = todos.find((t) => t.id === id);
		if (todo) todo.done = !todo.done;
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-8">Todo List (Svelte)</h1>
	<div class="max-w-lg mx-auto mb-6 flex gap-3">
		<input
			class="flex-1 bg-gray-800 border-2 border-gray-700 text-white px-4 py-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
			bind:value={input}
			onkeydown={(e) => e.key === 'Enter' && addTodo()}
			placeholder="Add a todo..."
		/>
		<button
			class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg font-bold cursor-pointer"
			onclick={addTodo}>Add</button
		>
	</div>
	<ul class="max-w-[500px] mx-auto list-none p-0">
		{#each todos as todo}
			<li class="bg-gray-800 border-2 border-gray-700 rounded-lg p-4 mb-3 flex items-center gap-3">
				<input
					type="checkbox"
					class="w-5 h-5 cursor-pointer accent-blue-400"
					bind:checked={todo.done}
				/>
				<span
					class="flex-1 text-base"
					class:line-through={todo.done}
					class:text-gray-500={todo.done}>{todo.text}</span
				>
			</li>
		{/each}
	</ul>
</div>
```

**Key Differences:**

- **Less code**: No `useState`, `setTodos`, spreading arrays
- **Direct mutation**: `todos.push()` works! (fine-grained reactivity)
- **Two-way binding**: `bind:value` and `bind:checked`
- **No keys needed**: Svelte tracks by identity
- **Scoped CSS**: Styles don't leak to other components

> 💡 **Best Practice**: Svelte's fine-grained reactivity means you can mutate arrays directly. Use `todos.push()` instead of `setTodos([...todos, newTodo])`.

**⚠️ Common Mistakes:**

- Don't try to use `setState` - just assign values directly
- Don't spread arrays unnecessarily - direct mutations work
- Don't forget `bind:` for two-way binding (it's not automatic)

**⚡ Performance Comparison:**

- Svelte: ~15-20% less code than React equivalent
- No virtual DOM diffing overhead
- Fine-grained updates only change what's needed

---

## 3. Svelte vs SvelteKit

### What's the Difference?

- **Svelte**: The component framework/compiler
- **SvelteKit**: Full-stack framework built on top of Svelte (like Next.js for React)

**Real-World Scenario:** You're starting a new project and need to choose.

**When to use Svelte alone:**

- Simple single-page apps
- Embedded widgets
- Learning the fundamentals

**When to use SvelteKit:**

- Multi-page applications
- Need routing
- Server-side rendering (SSR)
- API endpoints
- Production apps

**Creating a SvelteKit Project:**

```bash
npm create svelte@latest my-app
cd my-app
npm install
npm run dev
```

**Key Concepts:**

- Svelte = component library
- SvelteKit = full framework with routing, SSR, etc.
- Start with Svelte to learn, move to SvelteKit for production

> 💡 **Best Practice**: Use SvelteKit for any production app - it provides routing, SSR, and better developer experience out of the box.

**⚠️ Common Mistakes:**

- Don't use Svelte alone for multi-page apps - you'll rebuild what SvelteKit provides
- Don't skip SvelteKit docs if you're building a real app
- Don't confuse Svelte (the language) with SvelteKit (the framework)

**Decision Tree:**

```
Need routing? → Yes → Use SvelteKit
Need SSR? → Yes → Use SvelteKit
Building widget? → Yes → Use Svelte alone
Learning basics? → Yes → Use Svelte alone
Production app? → Yes → Use SvelteKit
```

---

## 4. Pure Functions

### What are Pure Functions?

A pure function always returns the same output for the same input and has no side effects.

**Real-World Scenario:** You're calculating prices with tax in an e-commerce app. Pure functions make this predictable and testable!

**What it does:** Shows pure vs impure functions.

### 📁 Files to Create

Create:

- `src/routes/pure-functions/+page.svelte`

```svelte
<script lang="ts">
	// ✅ PURE: Same input = same output, no side effects
	function calculateTotal(price: number, taxRate: number): number {
		return price + price * taxRate;
	}

	// ❌ IMPURE: Depends on external state
	let globalDiscount = 0.1;
	function calculateWithDiscount(price: number): number {
		return price - price * globalDiscount; // External dependency!
	}

	// ❌ IMPURE: Has side effects (modifies external state)
	let orderCount = 0;
	function processOrder(price: number): number {
		orderCount++; // Side effect!
		return price;
	}

	// ✅ PURE: Better version
	function processOrderPure(price: number, currentCount: number) {
		return {
			price,
			newCount: currentCount + 1
		};
	}

	// Demo state
	let price = $state(100);
	let taxRate = $state(0.08);

	const total = $derived(calculateTotal(price, taxRate));
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">💰 Pure Functions Demo</h1>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 max-w-2xl mx-auto mb-6">
		<h2 class="m-0 mb-5 text-white text-2xl font-bold">✅ Pure Function Example</h2>
		<div class="flex flex-col gap-5 mb-6">
			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Price: ${price}
				<input type="range" class="w-full accent-blue-400" bind:value={price} min="0" max="200" />
			</label>
			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Tax Rate: {(taxRate * 100).toFixed(0)}%
				<input
					type="range"
					class="w-full accent-blue-400"
					bind:value={taxRate}
					min="0"
					max="0.2"
					step="0.01"
				/>
			</label>
		</div>
		<div class="bg-blue-400 text-black p-4 rounded-lg text-center text-2xl font-extrabold mb-4">
			Total: ${total.toFixed(2)}
		</div>
		<p class="text-gray-500 text-sm m-0 italic">
			✅ <code class="bg-gray-900 text-blue-400 px-1.5 py-0.5 rounded font-mono"
				>calculateTotal()</code
			> is pure: same inputs always produce same output
		</p>
	</div>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 max-w-2xl mx-auto mb-6">
		<h2 class="m-0 mb-5 text-white text-2xl font-bold">Benefits of Pure Functions</h2>
		<ul class="m-0 pl-5">
			<li class="mb-3 leading-relaxed">🧪 <strong>Testable:</strong> Easy to unit test</li>
			<li class="mb-3 leading-relaxed">🔮 <strong>Predictable:</strong> No surprises</li>
			<li class="mb-3 leading-relaxed">🔄 <strong>Cacheable:</strong> Can memoize results</li>
			<li class="mb-3 leading-relaxed">🧩 <strong>Composable:</strong> Easy to combine</li>
			<li class="mb-3 leading-relaxed">🐛 <strong>Debuggable:</strong> No hidden dependencies</li>
		</ul>
	</div>
</div>
```

**Key Concepts:**

- **Pure**: Same input → same output, no side effects
- **Impure**: Depends on external state or modifies it
- **Why it matters**: Predictability, testability, debugging
- **In Svelte**: Use pure functions in `$derived` for predictable reactivity

> 💡 **Best Practice**: Keep business logic in pure functions. Use $effect only for side effects (API calls, localStorage, DOM manipulation).

**⚠️ Common Mistakes:**

- Don't mix calculations with side effects
- Don't rely on external state in pure functions
- Don't mutate function parameters

**🧪 Testing Tip:**

```typescript
// Pure functions are easy to test!
test('calculateTotal adds tax', () => {
	expect(calculateTotal(100, 0.1)).toBe(110);
});
```

---

## 5. The Microtask Queue

### What is the Microtask Queue?

JavaScript has multiple task queues. The microtask queue runs **before** the next render, making it perfect for batching state updates!

**Real-World Scenario:** You're updating multiple pieces of state and want the DOM to update only once (not for each state change).

**What it does:** Demonstrates how Svelte batches updates automatically.

### 📁 Files to Create

Create:

- `src/routes/microtask-queue/+page.svelte`

```svelte
<script lang="ts">
	let count = $state(0);
	let doubled = $state(0);
	let tripled = $state(0);
	let renderCount = $state(0);

	// This runs AFTER the DOM updates
	$effect(() => {
		// Read any state to track renders
		(count, doubled, tripled);
		renderCount++;
	});

	function updateMultiple() {
		// All three updates happen in the microtask queue
		// DOM only updates ONCE after all three!
		count++;
		doubled = count * 2;
		tripled = count * 3;

		console.log('Updates queued, but DOM not updated yet!');
	}

	function updateWithDelay() {
		count++;

		// queueMicrotask runs before next render
		queueMicrotask(() => {
			doubled = count * 2;
			console.log('Microtask executed');
		});

		// setTimeout runs AFTER render
		setTimeout(() => {
			tripled = count * 3;
			console.log('Timeout executed (after render)');
		}, 0);
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">⚡ Microtask Queue Demo</h1>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 max-w-3xl mx-auto mb-6">
		<h2 class="m-0 mb-5 text-white text-2xl font-bold">State Values</h2>
		<div class="grid grid-cols-[repeat(auto-fit,minmax(150px,1fr))] gap-4">
			<div class="bg-gray-900 p-4 rounded-lg text-center">
				<span class="block text-sm mb-2 font-semibold">Count:</span>
				<span class="block text-3xl font-extrabold">{count}</span>
			</div>
			<div class="bg-gray-900 p-4 rounded-lg text-center">
				<span class="block text-sm mb-2 font-semibold">Doubled:</span>
				<span class="block text-3xl font-extrabold">{doubled}</span>
			</div>
			<div class="bg-gray-900 p-4 rounded-lg text-center">
				<span class="block text-sm mb-2 font-semibold">Tripled:</span>
				<span class="block text-3xl font-extrabold">{tripled}</span>
			</div>
			<div class="bg-blue-400 text-black p-4 rounded-lg text-center">
				<span class="block text-sm mb-2 font-semibold">Render Count:</span>
				<span class="block text-3xl font-extrabold">{renderCount}</span>
			</div>
		</div>
	</div>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 max-w-3xl mx-auto mb-6">
		<h2 class="m-0 mb-5 text-white text-2xl font-bold">Try It Out</h2>
		<div class="flex gap-3 mb-4">
			<button
				onclick={updateMultiple}
				class="flex-1 px-6 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-blue-400 text-black hover:bg-blue-300 hover:-translate-y-0.5"
			>
				Update All (Batched)
			</button>
			<button
				onclick={updateWithDelay}
				class="flex-1 px-6 py-3.5 border-none rounded-lg text-base font-bold cursor-pointer transition-all duration-200 bg-green-400 text-black hover:bg-green-300 hover:-translate-y-0.5"
			>
				Update with Microtask/Timeout
			</button>
		</div>
		<p class="bg-blue-400/10 border-l-4 border-blue-400 px-4 py-3 m-0 rounded text-sm">
			💡 Notice: Multiple state updates only cause ONE render! Svelte batches them automatically.
		</p>
	</div>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 max-w-3xl mx-auto mb-6">
		<h2 class="m-0 mb-5 text-white text-2xl font-bold">How It Works</h2>
		<ol class="m-0 pl-6">
			<li class="mb-3 leading-relaxed">
				<strong>Synchronous code runs:</strong> All state updates queued
			</li>
			<li class="mb-3 leading-relaxed">
				<strong>Microtasks run:</strong> queueMicrotask, Promises
			</li>
			<li class="mb-3 leading-relaxed"><strong>Render happens:</strong> DOM updates once</li>
			<li class="mb-3 leading-relaxed"><strong>Macrotasks run:</strong> setTimeout, setInterval</li>
		</ol>
	</div>
</div>
```

**Key Concepts:**

- **Microtasks**: Run before next render (Promises, queueMicrotask)
- **Macrotasks**: Run after render (setTimeout, setInterval)
- **Batching**: Svelte automatically batches state updates
- **Performance**: One render instead of many

---

## 6. Component Basics

### What are Svelte Components?

A Svelte component is a `.svelte` file with `<script>`, markup, and `<style>` sections.

**Real-World Scenario:** You're building a user profile card component.

**What it does:** Creates a reusable profile card component.

### 📁 Files to Create

Create:

- `src/lib/components/ProfileCard.svelte` (reusable component)
- `src/routes/profile-demo/+page.svelte` (page to use the component)

```svelte
<!-- ProfileCard.svelte -->
<script lang="ts">
	interface Props {
		name: string;
		role: string;
		avatar?: string;
		bio?: string;
		stats?: {
			followers: number;
			following: number;
			posts: number;
		};
	}

	let {
		name,
		role,
		avatar = '👤',
		bio,
		stats = { followers: 0, following: 0, posts: 0 }
	}: Props = $props();

	let isFollowing = $state(false);

	function toggleFollow() {
		isFollowing = !isFollowing;
	}
</script>

<div
	class="bg-gray-800 border-2 border-gray-700 rounded-2xl p-8 text-center max-w-sm transition-all duration-300 hover:border-blue-400 hover:-translate-y-1 hover:shadow-2xl"
>
	<div class="text-7xl mb-4">{avatar}</div>

	<h2 class="m-0 mb-2 text-white text-2xl font-extrabold">{name}</h2>
	<p class="m-0 mb-4 text-blue-400 text-base font-semibold">{role}</p>

	{#if bio}
		<p class="text-gray-400 text-sm leading-relaxed m-0 mb-6">{bio}</p>
	{/if}

	<div class="flex justify-around mb-6 py-5 border-t border-b border-gray-700">
		<div class="flex flex-col gap-1">
			<span class="text-2xl font-extrabold text-white">{stats.followers}</span>
			<span class="text-xs text-gray-500 uppercase font-semibold">Followers</span>
		</div>
		<div class="flex flex-col gap-1">
			<span class="text-2xl font-extrabold text-white">{stats.following}</span>
			<span class="text-xs text-gray-500 uppercase font-semibold">Following</span>
		</div>
		<div class="flex flex-col gap-1">
			<span class="text-2xl font-extrabold text-white">{stats.posts}</span>
			<span class="text-xs text-gray-500 uppercase font-semibold">Posts</span>
		</div>
	</div>

	<button
		class="w-full border-none px-6 py-3.5 rounded-lg text-base font-bold cursor-pointer transition-all duration-200"
		class:bg-blue-400={!isFollowing}
		class:text-black={!isFollowing}
		class:hover:bg-blue-300={!isFollowing}
		class:bg-green-400={isFollowing}
		class:hover:bg-green-300={isFollowing}
		class:hover:scale-[1.02]={true}
		onclick={toggleFollow}
	>
		{isFollowing ? '✓ Following' : '+ Follow'}
	</button>
</div>
```

### Usage:

```svelte
<script lang="ts">
	import ProfileCard from './ProfileCard.svelte';
</script>

<div class="bg-gray-900 min-h-screen p-10 flex justify-center items-center">
	<ProfileCard
		name="Sarah Chen"
		role="Senior Developer"
		avatar="👩‍💻"
		bio="Building beautiful web experiences with Svelte. Coffee enthusiast ☕"
		stats={{ followers: 1234, following: 567, posts: 89 }}
	/>
</div>
```

**Key Concepts:**

- **Three sections**: `<script>`, markup, `<style>`
- **$props()**: Define component props with TypeScript
- **Scoped styles**: CSS only affects this component
- **Default values**: Use destructuring for defaults

---

## 7. Adding Reactivity with $state

### What is $state?

`$state()` creates reactive state that automatically updates the UI when changed.

**Real-World Scenario:** You're building a shopping cart where quantities update in real-time.

**What it does:** Creates a fully reactive shopping cart.

### 📁 Files to Create

Create:

- `src/routes/shopping-cart/+page.svelte`

```svelte
<script lang="ts">
	interface CartItem {
		id: number;
		name: string;
		price: number;
		quantity: number;
		image: string;
	}

	let cart = $state<CartItem[]>([
		{ id: 1, name: 'Laptop', price: 999, quantity: 1, image: '💻' },
		{ id: 2, name: 'Mouse', price: 49, quantity: 2, image: '🖱️' },
		{ id: 3, name: 'Keyboard', price: 129, quantity: 1, image: '⌨️' }
	]);

	// Derived values automatically recalculate
	const subtotal = $derived(cart.reduce((sum, item) => sum + item.price * item.quantity, 0));
	const tax = $derived(subtotal * 0.08);
	const total = $derived(subtotal + tax);
	const itemCount = $derived(cart.reduce((sum, item) => sum + item.quantity, 0));

	function updateQuantity(id: number, delta: number) {
		const item = cart.find((i) => i.id === id);
		if (item) {
			item.quantity = Math.max(0, item.quantity + delta);
			// Remove if quantity reaches 0
			if (item.quantity === 0) {
				cart = cart.filter((i) => i.id !== id);
			}
		}
	}

	function removeItem(id: number) {
		cart = cart.filter((i) => i.id !== id);
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-8">🛒 Shopping Cart</h1>

	<div class="max-w-4xl mx-auto mb-6 text-right">
		<span class="bg-gray-800 px-4 py-2 rounded-full font-semibold text-blue-400"
			>{itemCount} {itemCount === 1 ? 'item' : 'items'}</span
		>
	</div>

	{#if cart.length === 0}
		<div class="text-center py-20 px-5 text-gray-500">
			<p>Your cart is empty</p>
			<p class="text-6xl mt-5">🛍️</p>
		</div>
	{:else}
		<div class="max-w-4xl mx-auto mb-8">
			{#each cart as item (item.id)}
				<div
					class="bg-gray-800 border-2 border-gray-700 rounded-xl p-5 mb-4 flex items-center gap-5 relative"
				>
					<div class="text-5xl">{item.image}</div>
					<div class="flex-1">
						<h3 class="m-0 mb-1 text-white text-lg">{item.name}</h3>
						<p class="m-0 text-gray-500 text-sm">${item.price.toFixed(2)} each</p>
					</div>
					<div class="flex items-center gap-3">
						<button
							class="bg-gray-700 text-white border-none w-8 h-8 rounded-md text-lg cursor-pointer transition-all duration-200 hover:bg-blue-400 hover:text-black"
							onclick={() => updateQuantity(item.id, -1)}>−</button
						>
						<span class="font-bold text-lg min-w-8 text-center">{item.quantity}</span>
						<button
							class="bg-gray-700 text-white border-none w-8 h-8 rounded-md text-lg cursor-pointer transition-all duration-200 hover:bg-blue-400 hover:text-black"
							onclick={() => updateQuantity(item.id, 1)}>+</button
						>
					</div>
					<div class="font-extrabold text-xl text-green-400 min-w-[100px] text-right">
						${(item.price * item.quantity).toFixed(2)}
					</div>
					<button
						class="absolute top-2.5 right-2.5 bg-transparent border-none text-gray-600 text-xl cursor-pointer p-1 px-2 hover:text-red-400"
						onclick={() => removeItem(item.id)}
					>
						✕
					</button>
				</div>
			{/each}
		</div>

		<div class="max-w-4xl mx-auto bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<div class="flex justify-between py-3 text-base">
				<span>Subtotal:</span>
				<span>${subtotal.toFixed(2)}</span>
			</div>
			<div class="flex justify-between py-3 text-base">
				<span>Tax (8%):</span>
				<span>${tax.toFixed(2)}</span>
			</div>
			<div
				class="flex justify-between border-t-2 border-gray-700 mt-3 pt-5 text-2xl font-extrabold text-blue-400"
			>
				<span>Total:</span>
				<span>${total.toFixed(2)}</span>
			</div>
			<button
				class="w-full bg-green-400 text-black border-none p-4 rounded-lg text-lg font-bold cursor-pointer mt-5 transition-all duration-200 hover:bg-green-300 hover:-translate-y-0.5"
			>
				Proceed to Checkout
			</button>
		</div>
	{/if}
</div>
```

**Key Concepts:**

- **$state()**: Creates reactive state
- **Direct mutation**: `item.quantity++` works!
- **Fine-grained**: Only affected UI updates
- **$derived()**: Automatic recalculation
- **No manual subscriptions**: Svelte tracks dependencies

---

## 8. Signal-Based Reactivity Deep Dive

### What are Signals?

Signals are the foundation of Svelte 5's reactivity system. They're reactive primitives that track dependencies automatically.

**Real-World Scenario:** You're building a form where multiple fields depend on each other, and you want automatic updates without manual tracking.

**What it does:** Demonstrates how signals work under the hood and how Svelte tracks dependencies.

### 📁 Files to Create

Create:

- `src/routes/signals-demo/+page.svelte`

```svelte
<script lang="ts">
	// $state creates a signal
	let count = $state(0);
	let name = $state('');
	let enabled = $state(true);

	// Signals can be objects
	let user = $state({
		firstName: 'John',
		lastName: 'Doe',
		age: 30
	});

	// Arrays are reactive too
	let todos = $state<string[]>(['Learn Svelte', 'Build an app']);

	// Derived signals automatically track dependencies
	const fullName = $derived(`${user.firstName} ${user.lastName}`);
	const isAdult = $derived(user.age >= 18);
	const todoCount = $derived(todos.length);

	function incrementAge() {
		user.age++;
	}

	function addTodo() {
		if (name.trim()) {
			todos.push(name);
			name = '';
		}
	}

	function removeTodo(index: number) {
		todos.splice(index, 1);
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">📡 Signal-Based Reactivity</h1>

	<div class="max-w-4xl mx-auto grid gap-6">
		<!-- Primitive Signals -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Primitive Signals</h2>
			<div class="flex gap-4 items-center mb-4">
				<button
					onclick={() => count++}
					class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300"
				>
					Increment: {count}
				</button>
				<label class="flex items-center gap-2">
					<input type="checkbox" bind:checked={enabled} class="w-5 h-5" />
					<span>Enabled: {enabled}</span>
				</label>
			</div>
		</div>

		<!-- Object Signals -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Object Signals</h2>
			<div class="flex gap-4 mb-4">
				<input
					type="text"
					bind:value={user.firstName}
					placeholder="First Name"
					class="flex-1 px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white"
				/>
				<input
					type="text"
					bind:value={user.lastName}
					placeholder="Last Name"
					class="flex-1 px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white"
				/>
			</div>
			<div class="flex gap-4 items-center">
				<button
					onclick={incrementAge}
					class="px-6 py-3 bg-green-400 text-black border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-green-300"
				>
					Age: {user.age}
				</button>
				<div class="text-lg">
					Full Name: <strong class="text-blue-400">{fullName}</strong>
				</div>
				<div class="text-lg">
					{isAdult ? '✓ Adult' : '✗ Minor'}
				</div>
			</div>
		</div>

		<!-- Array Signals -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Array Signals ({todoCount} items)</h2>
			<div class="flex gap-3 mb-4">
				<input
					type="text"
					bind:value={name}
					placeholder="Add a todo..."
					class="flex-1 px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white"
					onkeydown={(e) => e.key === 'Enter' && addTodo()}
				/>
				<button
					onclick={addTodo}
					class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300"
				>
					Add
				</button>
			</div>
			<ul class="m-0 p-0 list-none">
				{#each todos as todo, i}
					<li
						class="flex justify-between items-center bg-gray-900 p-4 rounded-lg mb-2 border-2 border-gray-700"
					>
						<span>{todo}</span>
						<button
							onclick={() => removeTodo(i)}
							class="bg-red-400 text-black px-4 py-2 border-none rounded cursor-pointer hover:bg-red-300"
						>
							Remove
						</button>
					</li>
				{/each}
			</ul>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Signals**: Reactive primitives that notify dependents when changed
- **Automatic tracking**: Svelte knows what depends on what
- **Fine-grained updates**: Only affected parts re-render
- **No subscriptions**: No `.subscribe()` or `useEffect` needed

---

## 9. Derived State with $derived

### What is $derived?

`$derived()` creates computed values that automatically update when their dependencies change. Think Excel formulas!

**Real-World Scenario:** You're building a dashboard where metrics are calculated from raw data, and everything updates automatically.

**What it does:** Shows various ways to use derived state.

### 📁 Files to Create

Create:

- `src/routes/derived-demo/+page.svelte`

```svelte
<script lang="ts">
	interface Product {
		name: string;
		price: number;
		quantity: number;
	}

	let products = $state<Product[]>([
		{ name: 'Laptop', price: 999, quantity: 2 },
		{ name: 'Mouse', price: 29, quantity: 5 },
		{ name: 'Keyboard', price: 79, quantity: 3 }
	]);

	let discountPercent = $state(10);
	let taxRate = $state(8);

	// Simple derived value
	const subtotal = $derived(products.reduce((sum, p) => sum + p.price * p.quantity, 0));

	// Chained derived values
	const discount = $derived(subtotal * (discountPercent / 100));
	const afterDiscount = $derived(subtotal - discount);
	const tax = $derived(afterDiscount * (taxRate / 100));
	const total = $derived(afterDiscount + tax);

	// Derived objects
	const stats = $derived({
		itemCount: products.reduce((sum, p) => sum + p.quantity, 0),
		productCount: products.length,
		averagePrice: products.length ? subtotal / products.reduce((sum, p) => sum + p.quantity, 0) : 0
	});

	// Derived with complex logic
	const savingsMessage = $derived.by(() => {
		if (discount > 100) return `🎉 You're saving over $100!`;
		if (discount > 50) return `💰 Great savings of $${discount.toFixed(2)}!`;
		if (discount > 0) return `✓ Saving $${discount.toFixed(2)}`;
		return '💡 Add a discount code to save!';
	});

	function updateQuantity(index: number, delta: number) {
		products[index].quantity = Math.max(0, products[index].quantity + delta);
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">🧮 Derived State Demo</h1>

	<div class="max-w-4xl mx-auto">
		<!-- Products -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 mb-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Products</h2>
			{#each products as product, i}
				<div class="flex items-center gap-4 bg-gray-900 p-4 rounded-lg mb-3">
					<span class="flex-1 font-semibold">{product.name}</span>
					<span class="text-gray-400">${product.price}</span>
					<div class="flex items-center gap-2">
						<button
							onclick={() => updateQuantity(i, -1)}
							class="bg-gray-700 text-white w-8 h-8 border-none rounded cursor-pointer hover:bg-blue-400"
							>−</button
						>
						<span class="w-8 text-center font-bold">{product.quantity}</span>
						<button
							onclick={() => updateQuantity(i, 1)}
							class="bg-gray-700 text-white w-8 h-8 border-none rounded cursor-pointer hover:bg-blue-400"
							>+</button
						>
					</div>
					<span class="text-green-400 font-bold min-w-[80px] text-right"
						>${(product.price * product.quantity).toFixed(2)}</span
					>
				</div>
			{/each}
		</div>

		<!-- Controls -->
		<div class="grid grid-cols-2 gap-4 mb-6">
			<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
				<label class="flex flex-col gap-2">
					<span class="font-semibold">Discount: {discountPercent}%</span>
					<input
						type="range"
						bind:value={discountPercent}
						min="0"
						max="50"
						class="w-full accent-blue-400"
					/>
				</label>
			</div>
			<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
				<label class="flex flex-col gap-2">
					<span class="font-semibold">Tax Rate: {taxRate}%</span>
					<input
						type="range"
						bind:value={taxRate}
						min="0"
						max="15"
						class="w-full accent-blue-400"
					/>
				</label>
			</div>
		</div>

		<!-- Stats -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 mb-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Stats (All Derived)</h2>
			<div class="grid grid-cols-3 gap-4 text-center">
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">{stats.itemCount}</div>
					<div class="text-sm text-gray-500">Total Items</div>
				</div>
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">{stats.productCount}</div>
					<div class="text-sm text-gray-500">Product Types</div>
				</div>
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">${stats.averagePrice.toFixed(2)}</div>
					<div class="text-sm text-gray-500">Avg Price/Item</div>
				</div>
			</div>
		</div>

		<!-- Breakdown -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Price Breakdown</h2>
			<div class="space-y-3">
				<div class="flex justify-between text-lg">
					<span>Subtotal:</span>
					<span>${subtotal.toFixed(2)}</span>
				</div>
				<div class="flex justify-between text-lg text-red-400">
					<span>Discount ({discountPercent}%):</span>
					<span>-${discount.toFixed(2)}</span>
				</div>
				<div class="flex justify-between text-lg">
					<span>After Discount:</span>
					<span>${afterDiscount.toFixed(2)}</span>
				</div>
				<div class="flex justify-between text-lg">
					<span>Tax ({taxRate}%):</span>
					<span>${tax.toFixed(2)}</span>
				</div>
				<div
					class="flex justify-between text-2xl font-bold text-green-400 border-t-2 border-gray-700 pt-3"
				>
					<span>Total:</span>
					<span>${total.toFixed(2)}</span>
				</div>
			</div>
			<div class="mt-4 bg-blue-400/10 border-l-4 border-blue-400 p-4 rounded">
				<p class="m-0 text-sm">{savingsMessage}</p>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **$derived()**: Auto-recalculates when dependencies change
- **Chaining**: Derived values can depend on other derived values
- **$derived.by()**: Use for complex logic with multiple statements
- **Performance**: Only recalculates when dependencies actually change

---

## 10. Side Effects with $effect

### What is $effect?

`$effect()` runs side effects when dependencies change. Use it for DOM manipulation, logging, API calls, etc.

**Real-World Scenario:** You need to sync state with localStorage, send analytics, or update the document title.

**What it does:** Demonstrates proper use of effects and common patterns.

### 📁 Files to Create

Create:

- `src/routes/effects-demo/+page.svelte`

```svelte
<script lang="ts">
	import { onMount } from 'svelte';

	let count = $state(0);
	let name = $state('User');
	let theme = $state<'light' | 'dark'>('dark');
	let notifications = $state<string[]>([]);

	// Effect runs when count changes
	$effect(() => {
		document.title = `Count: ${count}`;
		console.log('Title updated:', count);
	});

	// Effect with cleanup
	$effect(() => {
		const interval = setInterval(() => {
			console.log(`Current count: ${count}`);
		}, 5000);

		// Cleanup function
		return () => {
			clearInterval(interval);
			console.log('Interval cleared');
		};
	});

	// Effect for localStorage sync
	$effect(() => {
		localStorage.setItem('user-name', name);
		console.log('Saved to localStorage:', name);
	});

	// Effect for theme
	$effect(() => {
		document.body.classList.toggle('light-theme', theme === 'light');
		console.log('Theme changed:', theme);
	});

	// Load from localStorage on mount
	onMount(() => {
		const savedName = localStorage.getItem('user-name');
		if (savedName) {
			name = savedName;
		}
	});

	function addNotification() {
		const message = `Action at ${new Date().toLocaleTimeString()}`;
		notifications.push(message);

		// Auto-remove after 3 seconds
		setTimeout(() => {
			notifications = notifications.filter((n) => n !== message);
		}, 3000);
	}

	// Track when notifications change
	$effect(() => {
		if (notifications.length > 0) {
			console.log('Notifications:', notifications.length);
		}
	});
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">💫 Effects Demo</h1>

	<div class="max-w-4xl mx-auto space-y-6">
		<!-- Document Title Effect -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Document Title Effect</h2>
			<p class="text-gray-400 mb-4">Count changes update the browser tab title!</p>
			<div class="flex gap-3">
				<button
					onclick={() => count++}
					class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300"
				>
					Increment: {count}
				</button>
				<button
					onclick={() => (count = 0)}
					class="px-6 py-3 bg-gray-700 text-white border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-gray-600"
				>
					Reset
				</button>
			</div>
		</div>

		<!-- LocalStorage Effect -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">LocalStorage Sync</h2>
			<p class="text-gray-400 mb-4">Name is automatically saved to localStorage!</p>
			<input
				type="text"
				bind:value={name}
				placeholder="Enter your name"
				class="w-full px-4 py-3 bg-gray-900 border-2 border-gray-700 rounded-lg text-white"
			/>
			<p class="text-sm text-gray-500 mt-2">✓ Synced to localStorage automatically</p>
		</div>

		<!-- Theme Effect -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Theme Effect</h2>
			<p class="text-gray-400 mb-4">Effect updates document.body class!</p>
			<div class="flex gap-3">
				<button
					onclick={() => (theme = 'dark')}
					class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer transition-all duration-200"
					class:bg-blue-400={theme === 'dark'}
					class:text-black={theme === 'dark'}
					class:bg-gray-700={theme !== 'dark'}
					class:text-white={theme !== 'dark'}
				>
					Dark
				</button>
				<button
					onclick={() => (theme = 'light')}
					class="px-6 py-3 border-none rounded-lg font-bold cursor-pointer transition-all duration-200"
					class:bg-blue-400={theme === 'light'}
					class:text-black={theme === 'light'}
					class:bg-gray-700={theme !== 'light'}
					class:text-white={theme !== 'light'}
				>
					Light
				</button>
			</div>
		</div>

		<!-- Notifications with Cleanup -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">
				Notifications ({notifications.length})
			</h2>
			<button
				onclick={addNotification}
				class="px-6 py-3 bg-green-400 text-black border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-green-300 mb-4"
			>
				Add Notification (auto-removes in 3s)
			</button>
			<div class="space-y-2">
				{#each notifications as notification}
					<div class="bg-gray-900 p-4 rounded-lg border-l-4 border-green-400">
						{notification}
					</div>
				{/each}
			</div>
		</div>

		<!-- Console Logging -->
		<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-6">
			<h3 class="m-0 mb-3 text-blue-400 font-bold">💡 Open DevTools Console</h3>
			<p class="m-0 text-sm">Effects log messages when they run. Check the console to see them!</p>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **$effect()**: Runs side effects when dependencies change
- **Cleanup**: Return a function to clean up (intervals, listeners)
- **Automatic tracking**: Svelte knows what state the effect reads
- **When to use**: DOM manipulation, API calls, logging, sync with external systems
- **When NOT to use**: Deriving state (use `$derived` instead)

---

## 11. Complete Counter Component

### Building a Full-Featured Counter

Let's combine everything we've learned into a polished, production-ready counter component.

**Real-World Scenario:** You need a reusable counter with limits, step control, history tracking, and persistence.

**What it does:** Demonstrates all Svelte 5 reactivity features in one component.

### 📁 Files to Create

Create:

- `src/routes/section-1/counter/+page.svelte`

```svelte
<script lang="ts">
	import { onMount } from 'svelte';

	// State
	let count = $state(0);
	let step = $state(1);
	let min = $state(-10);
	let max = $state(10);
	let history = $state<number[]>([0]);
	let autoIncrement = $state(false);

	// Derived values
	const atMin = $derived(count <= min);
	const atMax = $derived(count >= max);
	const canUndo = $derived(history.length > 1);
	const average = $derived(
		history.length ? history.reduce((sum, val) => sum + val, 0) / history.length : 0
	);

	// Status message with complex logic
	const statusMessage = $derived.by(() => {
		if (atMax) return '🔴 Maximum reached!';
		if (atMin) return '🔴 Minimum reached!';
		if (count > 0) return '🟢 Positive';
		if (count < 0) return '🔵 Negative';
		return '⚪ Zero';
	});

	// Effects
	$effect(() => {
		document.title = `Counter: ${count}`;
	});

	$effect(() => {
		localStorage.setItem('counter-value', count.toString());
	});

	$effect(() => {
		if (!autoIncrement) return;

		const interval = setInterval(() => {
			if (count < max) {
				increment();
			} else {
				autoIncrement = false;
			}
		}, 100);

		return () => clearInterval(interval);
	});

	// Functions
	function increment() {
		if (count + step <= max) {
			count += step;
			addToHistory();
		}
	}

	function decrement() {
		if (count - step >= min) {
			count -= step;
			addToHistory();
		}
	}

	function reset() {
		count = 0;
		history = [0];
	}

	function addToHistory() {
		history.push(count);
		if (history.length > 10) {
			history = history.slice(-10);
		}
	}

	function undo() {
		if (canUndo) {
			history.pop();
			count = history[history.length - 1];
		}
	}

	// Load saved value
	onMount(() => {
		const saved = localStorage.getItem('counter-value');
		if (saved) {
			count = parseInt(saved, 10);
			history = [count];
		}
	});
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		🔢 Complete Counter Component
	</h1>

	<div class="max-w-4xl mx-auto space-y-6">
		<!-- Main Counter Display -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-8 text-center">
			<div class="text-8xl font-bold text-white mb-4">{count}</div>
			<div class="text-xl text-gray-400 mb-6">{statusMessage}</div>

			<div class="flex gap-3 justify-center mb-4">
				<button
					onclick={decrement}
					disabled={atMin}
					class="px-8 py-4 border-none rounded-lg text-xl font-bold cursor-pointer transition-all duration-200 disabled:opacity-30 disabled:cursor-not-allowed bg-red-400 text-black hover:bg-red-300 disabled:hover:bg-red-400"
				>
					− {step}
				</button>
				<button
					onclick={reset}
					class="px-8 py-4 bg-gray-700 text-white border-none rounded-lg text-xl font-bold cursor-pointer transition-all duration-200 hover:bg-gray-600"
				>
					Reset
				</button>
				<button
					onclick={increment}
					disabled={atMax}
					class="px-8 py-4 border-none rounded-lg text-xl font-bold cursor-pointer transition-all duration-200 disabled:opacity-30 disabled:cursor-not-allowed bg-green-400 text-black hover:bg-green-300 disabled:hover:bg-green-400"
				>
					+ {step}
				</button>
			</div>

			<div class="flex gap-3 justify-center">
				<button
					onclick={undo}
					disabled={!canUndo}
					class="px-6 py-3 bg-blue-400 text-black border-none rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 disabled:opacity-30 disabled:cursor-not-allowed"
				>
					↶ Undo
				</button>
				<label class="flex items-center gap-2 px-6 py-3 bg-gray-700 rounded-lg cursor-pointer">
					<input type="checkbox" bind:checked={autoIncrement} class="w-5 h-5" />
					<span class="font-bold">Auto Increment</span>
				</label>
			</div>
		</div>

		<!-- Controls -->
		<div class="grid grid-cols-3 gap-4">
			<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
				<label class="flex flex-col gap-2">
					<span class="font-semibold">Step: {step}</span>
					<input type="range" bind:value={step} min="1" max="5" class="w-full accent-blue-400" />
				</label>
			</div>
			<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
				<label class="flex flex-col gap-2">
					<span class="font-semibold">Min: {min}</span>
					<input type="range" bind:value={min} min="-20" max="0" class="w-full accent-red-400" />
				</label>
			</div>
			<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
				<label class="flex flex-col gap-2">
					<span class="font-semibold">Max: {max}</span>
					<input type="range" bind:value={max} min="0" max="20" class="w-full accent-green-400" />
				</label>
			</div>
		</div>

		<!-- Stats -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Statistics (Derived)</h2>
			<div class="grid grid-cols-4 gap-4 text-center">
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">{count}</div>
					<div class="text-sm text-gray-500">Current</div>
				</div>
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">{average.toFixed(1)}</div>
					<div class="text-sm text-gray-500">Average</div>
				</div>
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">{history.length}</div>
					<div class="text-sm text-gray-500">History</div>
				</div>
				<div class="bg-gray-900 p-4 rounded-lg">
					<div class="text-3xl font-bold text-blue-400">{max - min}</div>
					<div class="text-sm text-gray-500">Range</div>
				</div>
			</div>
		</div>

		<!-- History -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">History (Last 10)</h2>
			<div class="flex gap-2 flex-wrap">
				{#each history as value, i}
					<div
						class="px-4 py-2 rounded-lg font-bold"
						class:bg-blue-400={i === history.length - 1}
						class:text-black={i === history.length - 1}
						class:bg-gray-700={i !== history.length - 1}
						class:text-gray-400={i !== history.length - 1}
					>
						{value}
					</div>
				{/each}
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **All features combined**: $state, $derived, $derived.by, $effect
- **Limits & validation**: Min/max with derived checks
- **History tracking**: Array state with mutations
- **Persistence**: localStorage with effects
- **Auto-increment**: Interval with cleanup
- **Undo functionality**: History-based state management

> 💡 **Best Practice**: This component is 250+ lines. In production, split into smaller components:
>
> ```
> Counter.svelte (orchestrator)
> ├─ CounterDisplay.svelte (visual display)
> ├─ CounterControls.svelte (buttons)
> ├─ CounterSettings.svelte (sliders)
> └─ CounterHistory.svelte (history list)
> ```

**⚠️ Common Mistakes:**

- Don't put all logic in one component (this example is for learning)
- Don't forget to clear intervals in $effect cleanup
- Don't use $effect for derived values (use $derived instead)

**⚡ Performance Tips:**

- $derived is more efficient than $effect for computed values
- localStorage writes are synchronous - consider debouncing in production
- Keep effect logic minimal - move calculations to $derived

---

## 📝 Key Takeaways

✅ Svelte is a compiler, not a runtime framework  
✅ No virtual DOM = better performance  
✅ Less boilerplate than React  
✅ Pure functions = predictable code  
✅ Microtask queue batches updates automatically  
✅ Components have script, markup, and scoped styles  
✅ **$state()** creates reactive values with fine-grained updates  
✅ **$derived()** auto-recalculates computed values  
✅ **$derived.by()** handles complex multi-statement logic  
✅ **$effect()** runs side effects with automatic cleanup  
✅ Direct mutations work - no need for setState  
✅ Signals track dependencies automatically

---

## 🚀 Next Steps

After mastering these fundamentals, you'll be ready for:

- **Section 2**: Components, snippets, and styling patterns
- **Section 3**: Deep state management with nested objects
- **Section 4**: Advanced patterns like reactive classes and API integration
- Building production-ready reactive applications
