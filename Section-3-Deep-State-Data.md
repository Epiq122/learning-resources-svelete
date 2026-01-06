# Section 3: Deep State & Data Management

## 📚 Learning Objectives

By the end of this section, you will:

- Understand JavaScript Proxies and object setters/getters
- Master deep state reactivity in Svelte
- Debug Svelte applications effectively
- Display and manipulate lists with `#each` loops
- Handle array mutations correctly
- Work with immutable raw state
- Build a real-world Kanban board with drag-and-drop

---

## Table of Contents

- [Section 3: Deep State \& Data Management](#section-3-deep-state--data-management)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. JavaScript Proxies \& Setters/Getters](#1-javascript-proxies--settersgetters)
    - [What are Proxies and Setters/Getters?](#what-are-proxies-and-settersgetters)
    - [📁 Files to Create](#-files-to-create)
  - [2. Deep State Reactivity](#2-deep-state-reactivity)
    - [What is Deep State Reactivity?](#what-is-deep-state-reactivity)
    - [📁 Files to Create](#-files-to-create-1)
  - [3. Debugging in Svelte](#3-debugging-in-svelte)
    - [How to Debug Svelte Apps?](#how-to-debug-svelte-apps)
    - [📁 Files to Create](#-files-to-create-2)
  - [4. List Rendering with #each Loops](#4-list-rendering-with-each-loops)
    - [What are #each Loops?](#what-are-each-loops)
    - [📁 Files to Create](#-files-to-create-3)
  - [5. Array Mutations \& Reactivity](#5-array-mutations--reactivity)
    - [How Do Array Mutations Work?](#how-do-array-mutations-work)
    - [📁 Files to Create](#-files-to-create-4)
  - [6. $state.raw \& Immutable Patterns](#6-stateraw--immutable-patterns)
    - [When Should You Use $state.raw?](#when-should-you-use-stateraw)
    - [📁 Files to Create](#-files-to-create-5)
  - [7. Building a Real-World Kanban Board](#7-building-a-real-world-kanban-board)
    - [What is This?](#what-is-this)
    - [📁 Files to Create](#-files-to-create-6)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. JavaScript Proxies & Setters/Getters

### What are Proxies and Setters/Getters?

Proxies intercept operations on objects. Setters/getters let you run code when properties are accessed or modified. This is how Svelte's reactivity works under the hood!

**Real-World Scenario:** You're building a form that validates data as users type and tracks which fields have been touched.

**What it does:** Demonstrates how Proxies and setters/getters work.

### 📁 Files to Create

Create:

- `src/routes/proxies-demo/+page.svelte`

```svelte
<script lang="ts">
	// Example 1: Object with Getters and Setters
	interface UserForm {
		_email: string;
		_password: string;
		touched: Set<string>;
	}

	const formData = $state<UserForm>({
		_email: '',
		_password: '',
		touched: new Set()
	});

	// Computed properties with getters
	const emailValid = $derived(formData._email.includes('@') && formData._email.includes('.'));
	const passwordValid = $derived(formData._password.length >= 8);

	// Example 2: Proxy that logs all property access
	let accessLog = $state<string[]>([]);

	const proxyTarget = { name: 'John', age: 30, city: 'NYC' };

	const proxyHandler = {
		get(target: any, prop: string) {
			accessLog = [...accessLog, `READ: ${prop} = ${target[prop]}`];
			return target[prop];
		},
		set(target: any, prop: string, value: any) {
			accessLog = [...accessLog, `WRITE: ${prop} = ${value}`];
			target[prop] = value;
			return true;
		}
	};

	const monitoredObject = new Proxy(proxyTarget, proxyHandler);

	function accessName() {
		const name = monitoredObject.name;
		console.log(name);
	}

	function updateAge() {
		monitoredObject.age++;
	}

	// Example 3: Validation with Setters
	class ValidatedUser {
		private _email: string = '';
		private _age: number = 0;
		errors = $state<string[]>([]);

		get email() {
			return this._email;
		}

		set email(value: string) {
			this._email = value;
			this.validateEmail();
		}

		get age() {
			return this._age;
		}

		set age(value: number) {
			this._age = value;
			this.validateAge();
		}

		private validateEmail() {
			this.errors = this.errors.filter((e) => !e.includes('email'));
			if (!this._email.includes('@')) {
				this.errors = [...this.errors, 'Email must contain @'];
			}
		}

		private validateAge() {
			this.errors = this.errors.filter((e) => !e.includes('age'));
			if (this._age < 18) {
				this.errors = [...this.errors, 'Must be 18 or older'];
			}
		}
	}

	const validatedUser = new ValidatedUser();
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">
		🔍 Proxies & Setters/Getters Demo
	</h1>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 mb-6 max-w-4xl mx-auto">
		<h2 class="m-0 mb-5 text-blue-400 text-2xl">1. Getters with Validation</h2>
		<div class="flex flex-col gap-5">
			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Email:
				<input
					class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
					type="email"
					bind:value={formData._email}
					onblur={() => formData.touched.add('email')}
				/>
				{#if formData.touched.has('email')}
					<span
						class="text-sm font-semibold"
						class:text-green-400={emailValid}
						class:text-red-400={!emailValid}
					>
						{emailValid ? '✓ Valid' : '✗ Invalid email'}
					</span>
				{/if}
			</label>

			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Password:
				<input
					class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
					type="password"
					bind:value={formData._password}
					onblur={() => formData.touched.add('password')}
				/>
				{#if formData.touched.has('password')}
					<span
						class="text-sm font-semibold"
						class:text-green-400={passwordValid}
						class:text-red-400={!passwordValid}
					>
						{passwordValid ? '✓ Valid' : '✗ Must be 8+ characters'}
					</span>
				{/if}
			</label>
		</div>
	</div>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 mb-6 max-w-4xl mx-auto">
		<h2 class="m-0 mb-5 text-blue-400 text-2xl">2. Proxy Example (Monitor Access)</h2>
		<div class="flex gap-3 mb-5">
			<button
				class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
				onclick={accessName}>Read Name Property</button
			>
			<button
				class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
				onclick={updateAge}>Increment Age</button
			>
		</div>
		<div class="bg-gray-900 border border-gray-700 rounded-lg p-4 max-h-[200px] overflow-y-auto">
			<h3 class="m-0 mb-3 text-white text-base">Access Log:</h3>
			{#each accessLog as entry}
				<div class="text-green-400 font-mono text-xs py-1 border-b border-gray-800 last:border-b-0">
					{entry}
				</div>
			{/each}
		</div>
	</div>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 mb-6 max-w-4xl mx-auto">
		<h2 class="m-0 mb-5 text-blue-400 text-2xl">3. Class with Setters (Auto-Validation)</h2>
		<div class="flex flex-col gap-5">
			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Email:
				<input
					class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
					type="email"
					bind:value={validatedUser.email}
				/>
			</label>

			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Age:
				<input
					class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
					type="number"
					bind:value={validatedUser.age}
				/>
			</label>

			{#if validatedUser.errors.length > 0}
				<div class="bg-red-400/10 border-2 border-red-400 rounded-lg p-4">
					{#each validatedUser.errors as error}
						<div class="text-red-400 mb-2 last:mb-0">⚠️ {error}</div>
					{/each}
				</div>
			{:else}
				<div
					class="bg-green-400/10 border-2 border-green-400 rounded-lg p-4 text-green-400 font-semibold"
				>
					✅ All valid!
				</div>
			{/if}
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Proxy**: Intercepts object operations (get, set, delete, etc.)
- **Getters**: Computed properties that run code on read
- **Setters**: Run validation/logic when property is set
- **Svelte uses Proxies**: For fine-grained reactivity under the hood

---

## 2. Deep State Reactivity

### What is Deep State Reactivity?

Svelte 5 tracks nested object/array changes! Mutate deeply nested properties and the UI updates automatically.

**Real-World Scenario:** You're building a nested configuration editor where users can modify deeply nested settings.

**What it does:** Demonstrates that nested mutations trigger reactivity.

### 📁 Files to Create

Create:

- `src/routes/deep-state/+page.svelte`

**Best Practice:** Single-file demo showing deep reactivity concepts.

```svelte
<script lang="ts">
	interface Address {
		street: string;
		city: string;
		zip: string;
	}

	interface SocialLinks {
		twitter?: string;
		github?: string;
		linkedin?: string;
	}

	interface User {
		name: string;
		email: string;
		address: Address;
		tags: string[];
		social: SocialLinks;
		settings: {
			theme: 'light' | 'dark';
			notifications: {
				email: boolean;
				push: boolean;
				sms: boolean;
			};
		};
	}

	let user = $state<User>({
		name: 'Jane Doe',
		email: 'jane@example.com',
		address: {
			street: '123 Main St',
			city: 'San Francisco',
			zip: '94102'
		},
		tags: ['developer', 'designer'],
		social: {
			twitter: '@janedoe',
			github: 'janedoe'
		},
		settings: {
			theme: 'dark',
			notifications: {
				email: true,
				push: false,
				sms: false
			}
		}
	});

	let updateLog = $state<string[]>([]);

	function logUpdate(message: string) {
		updateLog = [...updateLog, `${new Date().toLocaleTimeString()}: ${message}`];
	}

	// Deep mutations
	function updateNestedAddress() {
		user.address.city = 'Los Angeles';
		user.address.zip = '90001';
		logUpdate('Updated nested address');
	}

	function toggleNotification(type: 'email' | 'push' | 'sms') {
		user.settings.notifications[type] = !user.settings.notifications[type];
		logUpdate(`Toggled ${type} notification`);
	}

	function addTag() {
		const newTag = `tag-${user.tags.length + 1}`;
		user.tags.push(newTag);
		logUpdate(`Added tag: ${newTag}`);
	}

	function removeTag(index: number) {
		const removed = user.tags[index];
		user.tags.splice(index, 1);
		logUpdate(`Removed tag: ${removed}`);
	}

	function updateSocial() {
		user.social.linkedin = 'jane-doe';
		logUpdate('Added LinkedIn profile');
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-8">🔄 Deep State Reactivity</h1>

	<div class="grid md:grid-cols-2 grid-cols-1 gap-6 max-w-7xl mx-auto">
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-blue-400 text-2xl font-bold">User Data</h2>
			<div class="data-view">
				<div class="mb-6 pb-6 border-b border-gray-700 last:border-b-0">
					<h3 class="mt-5 mb-3 text-white text-base first:mt-0">Basic Info</h3>
					<p class="my-2 text-gray-400"><strong>Name:</strong> {user.name}</p>
					<p class="my-2 text-gray-400"><strong>Email:</strong> {user.email}</p>
				</div>

				<div class="mb-6 pb-6 border-b border-gray-700 last:border-b-0">
					<h3 class="mt-5 mb-3 text-white text-base first:mt-0">Address (Nested)</h3>
					<p class="my-2 text-gray-400"><strong>Street:</strong> {user.address.street}</p>
					<p class="my-2 text-gray-400"><strong>City:</strong> {user.address.city}</p>
					<p class="my-2 text-gray-400"><strong>ZIP:</strong> {user.address.zip}</p>
				</div>

				<div class="mb-6 pb-6 border-b border-gray-700 last:border-b-0">
					<h3 class="mt-5 mb-3 text-white text-base first:mt-0">Tags (Array)</h3>
					<div class="flex flex-wrap gap-2">
						{#each user.tags as tag, i}
							<span
								class="bg-blue-400 text-black px-3 py-1.5 rounded-full text-sm font-semibold flex items-center gap-2"
							>
								{tag}
								<button
									class="bg-transparent border-none text-black cursor-pointer p-0 text-base leading-none"
									onclick={() => removeTag(i)}>✕</button
								>
							</span>
						{/each}
					</div>
				</div>

				<div class="mb-6 pb-6 border-b border-gray-700 last:border-b-0">
					<h3 class="mt-5 mb-3 text-white text-base first:mt-0">Social Links (Nested Object)</h3>
					{#if user.social.twitter}
						<p class="my-2 text-gray-400"><strong>Twitter:</strong> {user.social.twitter}</p>
					{/if}
					{#if user.social.github}
						<p class="my-2 text-gray-400"><strong>GitHub:</strong> {user.social.github}</p>
					{/if}
					{#if user.social.linkedin}
						<p class="my-2 text-gray-400"><strong>LinkedIn:</strong> {user.social.linkedin}</p>
					{/if}
				</div>

				<div class="mb-6 pb-6 border-b border-gray-700 last:border-b-0">
					<h3 class="mt-5 mb-3 text-white text-base first:mt-0">Settings (Deeply Nested)</h3>
					<p class="my-2 text-gray-400"><strong>Theme:</strong> {user.settings.theme}</p>
					<div class="flex flex-col gap-3 mt-3">
						<label class="flex items-center gap-2 cursor-pointer text-gray-400">
							<input
								class="w-5 h-5 cursor-pointer accent-blue-400"
								type="checkbox"
								checked={user.settings.notifications.email}
								onchange={() => toggleNotification('email')}
							/>
							Email Notifications
						</label>
						<label class="flex items-center gap-2 cursor-pointer text-gray-400">
							<input
								class="w-5 h-5 cursor-pointer accent-blue-400"
								type="checkbox"
								checked={user.settings.notifications.push}
								onchange={() => toggleNotification('push')}
							/>
							Push Notifications
						</label>
						<label class="flex items-center gap-2 cursor-pointer text-gray-400">
							<input
								class="w-5 h-5 cursor-pointer accent-blue-400"
								type="checkbox"
								checked={user.settings.notifications.sms}
								onchange={() => toggleNotification('sms')}
							/>
							SMS Notifications
						</label>
					</div>
				</div>
			</div>
		</div>

		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-blue-400 text-2xl font-bold">Actions</h2>
			<div class="flex flex-col gap-3 mb-6">
				<button
					class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
					onclick={updateNestedAddress}
				>
					Update Nested Address
				</button>
				<button
					class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
					onclick={addTag}
				>
					Add Tag
				</button>
				<button
					class="bg-blue-400 text-black border-none px-6 py-3 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
					onclick={updateSocial}
				>
					Add LinkedIn
				</button>
			</div>

			<h3 class="mt-5 mb-3 text-white text-base first:mt-0">Update Log</h3>
			<div class="bg-gray-900 border border-gray-700 rounded-lg p-4 max-h-80 overflow-y-auto">
				{#each updateLog as entry}
					<div
						class="text-green-400 font-mono text-xs py-1.5 border-b border-gray-800 last:border-b-0"
					>
						{entry}
					</div>
				{/each}
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Deep reactivity**: Nested mutations trigger updates
- **Direct mutation**: `user.address.city = 'LA'` works!
- **Array methods**: `push()`, `splice()` are reactive
- **No need for spreads**: Svelte tracks deeply

---

## 3. Debugging in Svelte

### How to Debug Svelte Apps?

Use `$inspect()` rune and browser DevTools to debug reactive state.

**Real-World Scenario:** You're tracking down why a computed value isn't updating correctly.

**What it does:** Shows debugging techniques for Svelte applications.

### 📁 Files to Create

Create:

- `src/routes/debugging-demo/+page.svelte`

```svelte
<script lang="ts">
	interface CartItem {
		id: number;
		name: string;
		price: number;
		quantity: number;
	}

	let cart = $state<CartItem[]>([
		{ id: 1, name: 'Laptop', price: 999, quantity: 1 },
		{ id: 2, name: 'Mouse', price: 49, quantity: 2 }
	]);

	let discount = $state(0.1); // 10%

	const subtotal = $derived(cart.reduce((sum, item) => sum + item.price * item.quantity, 0));

	const discountAmount = $derived(subtotal * discount);
	const total = $derived(subtotal - discountAmount);

	// 🔍 DEBUG: Inspect state changes
	$inspect(cart).with((type, value) => {
		console.log(`Cart ${type}:`, value);
	});

	$inspect(subtotal, total).with(() => {
		console.log(`Totals - Subtotal: $${subtotal}, Total: $${total}`);
	});

	let debugLog = $state<string[]>([]);

	function log(message: string) {
		debugLog = [...debugLog, `[${new Date().toLocaleTimeString()}] ${message}`];
		console.log(message);
	}

	function addItem() {
		const newItem: CartItem = {
			id: Date.now(),
			name: `Item ${cart.length + 1}`,
			price: Math.floor(Math.random() * 100) + 10,
			quantity: 1
		};
		cart.push(newItem);
		log(`Added: ${newItem.name}`);
	}

	function updateQuantity(id: number, delta: number) {
		const item = cart.find((i) => i.id === id);
		if (item) {
			item.quantity = Math.max(1, item.quantity + delta);
			log(`Updated ${item.name} quantity to ${item.quantity}`);
		}
	}

	function removeItem(id: number) {
		const item = cart.find((i) => i.id === id);
		if (item) {
			cart = cart.filter((i) => i.id !== id);
			log(`Removed: ${item.name}`);
		}
	}

	// Breakpoint helper
	$effect(() => {
		if (total > 1000) {
			console.warn('⚠️ Total exceeds $1000!');
			debugger; // Pauses in DevTools
		}
	});
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-6">🐛 Debugging Demo</h1>

	<div class="bg-blue-400/10 border-2 border-blue-400 rounded-xl p-5 mx-auto mb-8 max-w-4xl">
		<h3 class="m-0 mb-3 text-blue-400">💡 Debugging Tips:</h3>
		<ul class="m-0 pl-6">
			<li class="mb-2 leading-relaxed">Open browser DevTools console</li>
			<li class="mb-2 leading-relaxed">
				Watch <code class="bg-gray-800 text-green-400 px-1.5 py-0.5 rounded font-mono"
					>$inspect()</code
				> logs when state changes
			</li>
			<li class="mb-2 leading-relaxed">Total > $1000 triggers debugger statement</li>
			<li class="mb-2 leading-relaxed">Check debug log below for application events</li>
		</ul>
	</div>

	<div class="grid md:grid-cols-2 grid-cols-1 gap-6 max-w-7xl mx-auto">
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Shopping Cart</h2>

			<button
				onclick={addItem}
				class="w-full bg-green-400 text-black border-none p-3.5 rounded-lg font-bold cursor-pointer mb-5 transition-all duration-200 hover:bg-green-300 hover:-translate-y-0.5"
			>
				+ Add Random Item
			</button>

			<div class="flex flex-col gap-3 mb-6">
				{#each cart as item}
					<div class="bg-gray-900 border border-gray-700 rounded-lg p-4 flex items-center gap-3">
						<div class="flex-1">
							<h4 class="m-0 mb-1 text-white">{item.name}</h4>
							<p class="m-0 text-green-400 font-bold">${item.price.toFixed(2)}</p>
						</div>
						<div class="flex items-center gap-3">
							<button
								class="bg-gray-700 text-white border-none w-8 h-8 rounded-md cursor-pointer text-lg"
								onclick={() => updateQuantity(item.id, -1)}>−</button
							>
							<span class="font-bold min-w-8 text-center">{item.quantity}</span>
							<button
								class="bg-gray-700 text-white border-none w-8 h-8 rounded-md cursor-pointer text-lg"
								onclick={() => updateQuantity(item.id, 1)}>+</button
							>
						</div>
						<button
							onclick={() => removeItem(item.id)}
							class="bg-transparent border-none cursor-pointer text-xl p-2"
						>
							🗑️
						</button>
					</div>
				{/each}
			</div>

			<div class="bg-gray-900 border border-gray-700 rounded-lg p-5 mb-5">
				<div class="flex justify-between py-2 text-base">
					<span>Subtotal:</span>
					<span>${subtotal.toFixed(2)}</span>
				</div>
				<div class="flex justify-between py-2 text-base">
					<span>Discount ({(discount * 100).toFixed(0)}%):</span>
					<span class="text-green-400">-${discountAmount.toFixed(2)}</span>
				</div>
				<div
					class="flex justify-between border-t-2 border-gray-700 mt-3 pt-4 text-xl font-extrabold text-blue-400"
				>
					<span>Total:</span>
					<span>${total.toFixed(2)}</span>
				</div>
			</div>

			<label class="flex flex-col gap-2 text-gray-400 font-semibold">
				Discount: {(discount * 100).toFixed(0)}%
				<input
					type="range"
					class="w-full accent-blue-400"
					bind:value={discount}
					min="0"
					max="0.5"
					step="0.05"
				/>
			</label>
		</div>

		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">Debug Log</h2>
			<div
				class="bg-gray-900 border border-gray-700 rounded-lg p-4 max-h-96 overflow-y-auto mb-4 min-h-48"
			>
				{#each debugLog as entry}
					<div
						class="text-green-400 font-mono text-xs py-1.5 border-b border-gray-800 last:border-b-0"
					>
						{entry}
					</div>
				{/each}
				{#if debugLog.length === 0}
					<p class="text-gray-600 text-center py-10 px-5 italic">
						No events yet. Try adding or removing items!
					</p>
				{/if}
			</div>

			<button
				onclick={() => (debugLog = [])}
				class="w-full bg-gray-700 text-white border-none p-3 rounded-lg font-semibold cursor-pointer hover:bg-gray-600"
			>
				Clear Log
			</button>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **$inspect()**: Logs state changes automatically
- **.with()**: Custom logging function
- **console methods**: log, warn, error, table
- **debugger**: Pauses execution in DevTools
- **$effect()**: Run code when dependencies change

---

## 4. List Rendering with #each Loops

### What are #each Loops?

The `#each` directive renders a list of items by iterating over an array. It's reactive - when the array changes, the DOM updates automatically.

**Real-World Scenario:** You're building a todo list application where users can add, complete, and delete tasks.

**What it does:** Demonstrates different #each patterns including keys, indexes, destructuring, and conditional rendering.

### 📁 Files to Create

**Main Demo:**

- `src/routes/section-3/lists/+page.svelte` - List rendering examples

```svelte
<script lang="ts">
	interface Todo {
		id: number;
		text: string;
		completed: boolean;
		priority: 'low' | 'medium' | 'high';
		dueDate: Date;
	}

	let todos = $state<Todo[]>([
		{
			id: 1,
			text: 'Learn Svelte 5',
			completed: false,
			priority: 'high',
			dueDate: new Date('2024-12-31')
		},
		{
			id: 2,
			text: 'Build a project',
			completed: false,
			priority: 'medium',
			dueDate: new Date('2025-01-15')
		},
		{
			id: 3,
			text: 'Deploy to production',
			completed: true,
			priority: 'high',
			dueDate: new Date('2024-11-20')
		}
	]);

	let newTodoText = $state('');
	let filterStatus = $state<'all' | 'active' | 'completed'>('all');
	let sortBy = $state<'priority' | 'dueDate' | 'alphabetical'>('priority');

	// Computed filtered and sorted list
	const filteredTodos = $derived(() => {
		let filtered = todos;

		// Filter by status
		if (filterStatus === 'active') {
			filtered = filtered.filter((todo) => !todo.completed);
		} else if (filterStatus === 'completed') {
			filtered = filtered.filter((todo) => todo.completed);
		}

		// Sort
		const sorted = [...filtered];
		if (sortBy === 'priority') {
			const priorityOrder = { high: 0, medium: 1, low: 2 };
			sorted.sort((a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]);
		} else if (sortBy === 'dueDate') {
			sorted.sort((a, b) => a.dueDate.getTime() - b.dueDate.getTime());
		} else {
			sorted.sort((a, b) => a.text.localeCompare(b.text));
		}

		return sorted;
	});

	function addTodo() {
		if (!newTodoText.trim()) return;

		todos.push({
			id: Date.now(),
			text: newTodoText,
			completed: false,
			priority: 'medium',
			dueDate: new Date(Date.now() + 7 * 24 * 60 * 60 * 1000) // 7 days from now
		});

		newTodoText = '';
	}

	function toggleTodo(id: number) {
		const todo = todos.find((t) => t.id === id);
		if (todo) {
			todo.completed = !todo.completed;
		}
	}

	function deleteTodo(id: number) {
		todos = todos.filter((t) => t.id !== id);
	}

	function changePriority(id: number, priority: 'low' | 'medium' | 'high') {
		const todo = todos.find((t) => t.id === id);
		if (todo) {
			todo.priority = priority;
		}
	}

	// Stats
	const totalTodos = $derived(todos.length);
	const activeTodos = $derived(todos.filter((t) => !t.completed).length);
	const completedTodos = $derived(todos.filter((t) => t.completed).length);
	const completionRate = $derived(
		totalTodos > 0 ? Math.round((completedTodos / totalTodos) * 100) : 0
	);
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-indigo-100 p-8">
	<div class="max-w-4xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">Todo List with #each</h1>
		<p class="text-gray-600 mb-8">Learn list rendering, keys, filtering, and sorting in Svelte 5</p>

		<!-- Stats -->
		<div class="grid grid-cols-4 gap-4 mb-8">
			<div class="bg-white rounded-lg shadow p-4">
				<div class="text-sm text-gray-600 mb-1">Total</div>
				<div class="text-3xl font-bold text-gray-900">{totalTodos}</div>
			</div>
			<div class="bg-white rounded-lg shadow p-4">
				<div class="text-sm text-gray-600 mb-1">Active</div>
				<div class="text-3xl font-bold text-blue-600">{activeTodos}</div>
			</div>
			<div class="bg-white rounded-lg shadow p-4">
				<div class="text-sm text-gray-600 mb-1">Completed</div>
				<div class="text-3xl font-bold text-green-600">{completedTodos}</div>
			</div>
			<div class="bg-white rounded-lg shadow p-4">
				<div class="text-sm text-gray-600 mb-1">Rate</div>
				<div class="text-3xl font-bold text-purple-600">{completionRate}%</div>
			</div>
		</div>

		<!-- Add New Todo -->
		<div class="bg-white rounded-lg shadow p-6 mb-6">
			<h2 class="text-xl font-semibold text-gray-900 mb-4">Add New Todo</h2>
			<form
				onsubmit={(e) => {
					e.preventDefault();
					addTodo();
				}}
				class="flex gap-2"
			>
				<input
					type="text"
					bind:value={newTodoText}
					placeholder="What needs to be done?"
					class="flex-1 px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
				/>
				<button
					type="submit"
					class="px-6 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition font-medium"
				>
					Add
				</button>
			</form>
		</div>

		<!-- Filters and Sorting -->
		<div class="bg-white rounded-lg shadow p-6 mb-6">
			<div class="flex gap-6 items-center">
				<div class="flex-1">
					<label class="text-sm font-medium text-gray-700 mb-2 block">Filter</label>
					<div class="flex gap-2">
						<button
							onclick={() => (filterStatus = 'all')}
							class={`px-4 py-2 rounded-lg transition ${
								filterStatus === 'all'
									? 'bg-blue-500 text-white'
									: 'bg-gray-100 text-gray-700 hover:bg-gray-200'
							}`}
						>
							All
						</button>
						<button
							onclick={() => (filterStatus = 'active')}
							class={`px-4 py-2 rounded-lg transition ${
								filterStatus === 'active'
									? 'bg-blue-500 text-white'
									: 'bg-gray-100 text-gray-700 hover:bg-gray-200'
							}`}
						>
							Active
						</button>
						<button
							onclick={() => (filterStatus = 'completed')}
							class={`px-4 py-2 rounded-lg transition ${
								filterStatus === 'completed'
									? 'bg-blue-500 text-white'
									: 'bg-gray-100 text-gray-700 hover:bg-gray-200'
							}`}
						>
							Completed
						</button>
					</div>
				</div>
				<div class="flex-1">
					<label class="text-sm font-medium text-gray-700 mb-2 block">Sort By</label>
					<select
						bind:value={sortBy}
						class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
					>
						<option value="priority">Priority</option>
						<option value="dueDate">Due Date</option>
						<option value="alphabetical">Alphabetical</option>
					</select>
				</div>
			</div>
		</div>

		<!-- Todo List -->
		<div class="bg-white rounded-lg shadow p-6">
			<h2 class="text-xl font-semibold text-gray-900 mb-4">
				Tasks ({filteredTodos().length})
			</h2>

			{#if filteredTodos().length === 0}
				<div class="text-center py-12 text-gray-500">
					<div class="text-6xl mb-4">✓</div>
					<div class="text-lg">No tasks found</div>
					<div class="text-sm">
						{#if filterStatus === 'active'}
							All tasks are completed!
						{:else if filterStatus === 'completed'}
							No completed tasks yet.
						{:else}
							Add a task to get started.
						{/if}
					</div>
				</div>
			{:else}
				<div class="space-y-3">
					<!-- IMPORTANT: Use (item.id) for keys -->
					{#each filteredTodos() as todo (todo.id)}
						<div
							class={`border rounded-lg p-4 transition ${
								todo.completed ? 'bg-gray-50 border-gray-200' : 'bg-white border-gray-300'
							}`}
						>
							<div class="flex items-start gap-3">
								<!-- Checkbox -->
								<input
									type="checkbox"
									checked={todo.completed}
									onchange={() => toggleTodo(todo.id)}
									class="mt-1 w-5 h-5 text-blue-500 rounded"
								/>

								<!-- Content -->
								<div class="flex-1">
									<div
										class={`text-lg ${todo.completed ? 'line-through text-gray-500' : 'text-gray-900'}`}
									>
										{todo.text}
									</div>
									<div class="flex items-center gap-4 mt-2 text-sm">
										<div class="flex items-center gap-2">
											<span class="text-gray-600">Priority:</span>
											<select
												value={todo.priority}
												onchange={(e) =>
													changePriority(
														todo.id,
														e.currentTarget.value as 'low' | 'medium' | 'high'
													)}
												class={`px-2 py-1 rounded text-xs font-medium ${
													todo.priority === 'high'
														? 'bg-red-100 text-red-700'
														: todo.priority === 'medium'
															? 'bg-yellow-100 text-yellow-700'
															: 'bg-green-100 text-green-700'
												}`}
											>
												<option value="low">Low</option>
												<option value="medium">Medium</option>
												<option value="high">High</option>
											</select>
										</div>
										<div class="text-gray-600">
											Due: {todo.dueDate.toLocaleDateString()}
										</div>
									</div>
								</div>

								<!-- Delete Button -->
								<button
									onclick={() => deleteTodo(todo.id)}
									class="text-red-500 hover:text-red-700 transition px-3 py-1 rounded hover:bg-red-50"
								>
									Delete
								</button>
							</div>
						</div>
					{/each}
				</div>
			{/if}
		</div>

		<!-- #each Patterns Reference -->
		<div class="bg-white rounded-lg shadow p-6 mt-6">
			<h2 class="text-xl font-semibold text-gray-900 mb-4">#each Patterns</h2>
			<div class="space-y-4 text-sm">
				<div class="border-l-4 border-blue-500 pl-4">
					<div class="font-mono text-blue-600 mb-1">{`{#each items as item (item.id)}`}</div>
					<div class="text-gray-600">Always use keys for lists that change</div>
				</div>
				<div class="border-l-4 border-purple-500 pl-4">
					<div class="font-mono text-purple-600 mb-1">
						{`{#each items as item, index}`}
					</div>
					<div class="text-gray-600">Access the index of each item</div>
				</div>
				<div class="border-l-4 border-green-500 pl-4">
					<div class="font-mono text-green-600 mb-1">
						{`{#each items as { id, name }}`}
					</div>
					<div class="text-gray-600">Destructure properties directly</div>
				</div>
				<div class="border-l-4 border-orange-500 pl-4">
					<div class="font-mono text-orange-600 mb-1">
						{`{#each items as item} ... {:else} ... {/each}`}
					</div>
					<div class="text-gray-600">Show fallback when list is empty</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Keys**: Use `(item.id)` to help Svelte track items efficiently
- **Index**: Access with `{#each items as item, index}`
- **Destructuring**: `{#each items as { id, name }}`
- **Empty state**: Use `{:else}` for empty lists
- **Filtering**: Compute filtered lists with $derived
- **Sorting**: Sort arrays before rendering

---

## 5. Array Mutations & Reactivity

### How Do Array Mutations Work?

In Svelte 5, direct array mutations (push, pop, splice, etc.) are automatically reactive. You don't need to reassign the array.

**Real-World Scenario:** You're building a real-time chat application where messages are added and removed frequently.

**What it does:** Demonstrates all array mutation methods and how they trigger reactivity in Svelte 5.

### 📁 Files to Create

**Main Demo:**

- `src/routes/section-3/array-mutations/+page.svelte` - Array mutation examples

```svelte
<script lang="ts">
	interface Message {
		id: number;
		text: string;
		author: string;
		timestamp: Date;
		edited?: boolean;
	}

	let messages = $state<Message[]>([
		{
			id: 1,
			text: 'Welcome to the chat!',
			author: 'System',
			timestamp: new Date('2024-01-01T10:00:00')
		},
		{
			id: 2,
			text: 'Hello everyone!',
			author: 'Alice',
			timestamp: new Date('2024-01-01T10:05:00')
		},
		{
			id: 3,
			text: 'Hey Alice! 👋',
			author: 'Bob',
			timestamp: new Date('2024-01-01T10:06:00')
		}
	]);

	let newMessageText = $state('');
	let currentUser = $state('Charlie');
	let actionLog = $state<string[]>([]);

	function logAction(action: string) {
		actionLog.push(`[${new Date().toLocaleTimeString()}] ${action}`);
	}

	// PUSH - Add to end
	function addMessage() {
		if (!newMessageText.trim()) return;

		messages.push({
			id: Date.now(),
			text: newMessageText,
			author: currentUser,
			timestamp: new Date()
		});

		logAction(`PUSH: Added message "${newMessageText}"`);
		newMessageText = '';
	}

	// POP - Remove from end
	function removeLastMessage() {
		const removed = messages.pop();
		if (removed) {
			logAction(`POP: Removed message from ${removed.author}`);
		}
	}

	// UNSHIFT - Add to beginning
	function addSystemMessage() {
		messages.unshift({
			id: Date.now(),
			text: 'New system announcement',
			author: 'System',
			timestamp: new Date()
		});
		logAction('UNSHIFT: Added message to beginning');
	}

	// SHIFT - Remove from beginning
	function removeFirstMessage() {
		const removed = messages.shift();
		if (removed) {
			logAction(`SHIFT: Removed first message from ${removed.author}`);
		}
	}

	// SPLICE - Remove specific index
	function deleteMessage(id: number) {
		const index = messages.findIndex((m) => m.id === id);
		if (index !== -1) {
			const removed = messages.splice(index, 1)[0];
			logAction(`SPLICE: Deleted message from ${removed.author}`);
		}
	}

	// SPLICE - Insert at specific index
	function insertMessageAt(index: number) {
		messages.splice(index, 0, {
			id: Date.now(),
			text: `Inserted at position ${index}`,
			author: currentUser,
			timestamp: new Date()
		});
		logAction(`SPLICE: Inserted message at index ${index}`);
	}

	// Direct index assignment
	function editMessage(id: number, newText: string) {
		const index = messages.findIndex((m) => m.id === id);
		if (index !== -1) {
			messages[index] = {
				...messages[index],
				text: newText,
				edited: true
			};
			logAction(`INDEX ASSIGNMENT: Edited message at index ${index}`);
		}
	}

	// SORT - Mutates array
	function sortByTimestamp() {
		messages.sort((a, b) => a.timestamp.getTime() - b.timestamp.getTime());
		logAction('SORT: Sorted messages by timestamp');
	}

	// REVERSE - Mutates array
	function reverseMessages() {
		messages.reverse();
		logAction('REVERSE: Reversed message order');
	}

	// Clear all
	function clearMessages() {
		messages.length = 0; // Another way to clear
		// Alternative: messages = [];
		logAction('LENGTH = 0: Cleared all messages');
	}

	const messageCount = $derived(messages.length);
</script>

<div class="min-h-screen bg-gradient-to-br from-purple-50 to-pink-100 p-8">
	<div class="max-w-6xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">Array Mutations in Svelte 5</h1>
		<p class="text-gray-600 mb-8">
			All array mutations are automatically reactive - no reassignment needed!
		</p>

		<div class="grid grid-cols-2 gap-6">
			<!-- Chat Messages -->
			<div class="bg-white rounded-lg shadow p-6">
				<div class="flex items-center justify-between mb-4">
					<h2 class="text-xl font-semibold text-gray-900">
						Messages ({messageCount})
					</h2>
					<button
						onclick={clearMessages}
						class="px-3 py-1 bg-red-500 text-white text-sm rounded hover:bg-red-600 transition"
					>
						Clear All
					</button>
				</div>

				<!-- Message List -->
				<div class="space-y-2 mb-4 max-h-96 overflow-y-auto">
					{#each messages as message, index (message.id)}
						<div
							class={`p-3 rounded-lg ${
								message.author === 'System' ? 'bg-blue-50 border border-blue-200' : 'bg-gray-50'
							}`}
						>
							<div class="flex items-start justify-between">
								<div class="flex-1">
									<div class="text-sm font-medium text-gray-900">
										{message.author}
										{#if message.edited}
											<span class="text-xs text-gray-500">(edited)</span>
										{/if}
									</div>
									<div class="text-gray-700">{message.text}</div>
									<div class="text-xs text-gray-500 mt-1">
										{message.timestamp.toLocaleTimeString()} • Index: {index}
									</div>
								</div>
								<button
									onclick={() => deleteMessage(message.id)}
									class="text-red-500 hover:text-red-700 text-sm ml-2"
								>
									Delete
								</button>
							</div>
						</div>
					{/each}
				</div>

				<!-- Add Message -->
				<form
					onsubmit={(e) => {
						e.preventDefault();
						addMessage();
					}}
					class="flex gap-2"
				>
					<input
						type="text"
						bind:value={newMessageText}
						placeholder="Type a message..."
						class="flex-1 px-3 py-2 border border-gray-300 rounded focus:outline-none focus:ring-2 focus:ring-purple-500 text-sm"
					/>
					<button
						type="submit"
						class="px-4 py-2 bg-purple-500 text-white rounded hover:bg-purple-600 transition text-sm font-medium"
					>
						Send
					</button>
				</form>
			</div>

			<!-- Controls -->
			<div class="space-y-4">
				<!-- Array Methods -->
				<div class="bg-white rounded-lg shadow p-6">
					<h3 class="text-lg font-semibold text-gray-900 mb-4">Array Methods</h3>
					<div class="grid grid-cols-2 gap-2">
						<button
							onclick={addMessage}
							class="px-4 py-2 bg-green-500 text-white rounded hover:bg-green-600 transition text-sm font-medium"
						>
							.push() - Add End
						</button>
						<button
							onclick={removeLastMessage}
							class="px-4 py-2 bg-orange-500 text-white rounded hover:bg-orange-600 transition text-sm font-medium"
						>
							.pop() - Remove End
						</button>
						<button
							onclick={addSystemMessage}
							class="px-4 py-2 bg-blue-500 text-white rounded hover:bg-blue-600 transition text-sm font-medium"
						>
							.unshift() - Add Start
						</button>
						<button
							onclick={removeFirstMessage}
							class="px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 transition text-sm font-medium"
						>
							.shift() - Remove Start
						</button>
						<button
							onclick={() => insertMessageAt(1)}
							class="px-4 py-2 bg-purple-500 text-white rounded hover:bg-purple-600 transition text-sm font-medium"
						>
							.splice() - Insert at 1
						</button>
						<button
							onclick={sortByTimestamp}
							class="px-4 py-2 bg-indigo-500 text-white rounded hover:bg-indigo-600 transition text-sm font-medium"
						>
							.sort() - Sort
						</button>
						<button
							onclick={reverseMessages}
							class="px-4 py-2 bg-pink-500 text-white rounded hover:bg-pink-600 transition text-sm font-medium"
						>
							.reverse() - Reverse
						</button>
						<button
							onclick={clearMessages}
							class="px-4 py-2 bg-gray-500 text-white rounded hover:bg-gray-600 transition text-sm font-medium"
						>
							.length = 0 - Clear
						</button>
					</div>
				</div>

				<!-- Action Log -->
				<div class="bg-white rounded-lg shadow p-6">
					<div class="flex items-center justify-between mb-4">
						<h3 class="text-lg font-semibold text-gray-900">Action Log</h3>
						<button
							onclick={() => (actionLog = [])}
							class="text-sm text-gray-600 hover:text-gray-800"
						>
							Clear
						</button>
					</div>
					<div class="space-y-1 max-h-64 overflow-y-auto text-sm font-mono">
						{#each actionLog as log (log)}
							<div class="text-gray-700 text-xs">{log}</div>
						{/each}
					</div>
				</div>

				<!-- Cheat Sheet -->
				<div class="bg-white rounded-lg shadow p-6">
					<h3 class="text-lg font-semibold text-gray-900 mb-4">Reactivity Cheat Sheet</h3>
					<div class="space-y-2 text-sm">
						<div class="border-l-4 border-green-500 pl-3">
							<div class="font-mono text-green-600">arr.push(item)</div>
							<div class="text-gray-600">✅ Reactive in Svelte 5</div>
						</div>
						<div class="border-l-4 border-green-500 pl-3">
							<div class="font-mono text-green-600">arr[0] = newValue</div>
							<div class="text-gray-600">✅ Reactive in Svelte 5</div>
						</div>
						<div class="border-l-4 border-green-500 pl-3">
							<div class="font-mono text-green-600">arr.splice(1, 1)</div>
							<div class="text-gray-600">✅ Reactive in Svelte 5</div>
						</div>
						<div class="border-l-4 border-green-500 pl-3">
							<div class="font-mono text-green-600">arr.sort()</div>
							<div class="text-gray-600">✅ Reactive in Svelte 5</div>
						</div>
						<div class="border-l-4 border-red-500 pl-3">
							<div class="font-mono text-red-600">arr.map()</div>
							<div class="text-gray-600">❌ Returns new array (not mutation)</div>
						</div>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **push()**: Add to end - automatically reactive
- **pop()**: Remove from end - automatically reactive
- **unshift()**: Add to beginning - automatically reactive
- **shift()**: Remove from beginning - automatically reactive
- **splice()**: Insert/remove at index - automatically reactive
- **Direct assignment**: `arr[i] = value` - automatically reactive
- **sort()/reverse()**: Mutate array - automatically reactive
- **Non-mutating methods**: map(), filter(), slice() return NEW arrays

---

## 6. $state.raw & Immutable Patterns

### When Should You Use $state.raw?

Use `$state.raw()` for large data structures where you don't need deep reactivity. It improves performance by skipping Proxy wrapping.

**Real-World Scenario:** You're building a data visualization dashboard that loads thousands of data points that rarely change.

**What it does:** Compares reactive state vs raw state and shows when to use each pattern.

### 📁 Files to Create

**Main Demo:**

- `src/routes/section-3/raw-state/+page.svelte` - $state.raw comparison

```svelte
<script lang="ts">
	interface DataPoint {
		id: number;
		timestamp: Date;
		value: number;
		category: string;
	}

	// Regular reactive state (Proxied)
	let reactiveData = $state<DataPoint[]>([]);

	// Raw state (NOT Proxied - better performance)
	let rawData = $state.raw<DataPoint[]>([]);

	// Metadata that needs reactivity
	let reactiveMetadata = $state({
		lastUpdate: new Date(),
		totalRecords: 0,
		isLoading: false
	});

	// Raw metadata (no reactivity needed)
	let rawMetadata = $state.raw({
		dataSource: 'API',
		version: '1.0',
		staticConfig: {
			maxRecords: 10000,
			refreshInterval: 5000
		}
	});

	let operationTimes = $state<{ operation: string; reactive: number; raw: number }[]>([]);

	// Generate sample data
	function generateData(count: number): DataPoint[] {
		return Array.from({ length: count }, (_, i) => ({
			id: i,
			timestamp: new Date(Date.now() - Math.random() * 86400000),
			value: Math.random() * 100,
			category: ['A', 'B', 'C', 'D'][Math.floor(Math.random() * 4)]
		}));
	}

	// Benchmark: Loading data
	function benchmarkLoad(count: number) {
		// Reactive version
		const reactiveStart = performance.now();
		reactiveData = generateData(count);
		const reactiveTime = performance.now() - reactiveStart;

		// Raw version
		const rawStart = performance.now();
		rawData = generateData(count);
		const rawTime = performance.now() - rawStart;

		operationTimes.push({
			operation: `Load ${count} records`,
			reactive: reactiveTime,
			raw: rawTime
		});

		reactiveMetadata.lastUpdate = new Date();
		reactiveMetadata.totalRecords = count;
	}

	// Benchmark: Reading data
	function benchmarkRead() {
		// Reactive version
		const reactiveStart = performance.now();
		let sum = 0;
		for (let i = 0; i < reactiveData.length; i++) {
			sum += reactiveData[i].value;
		}
		const reactiveTime = performance.now() - reactiveStart;

		// Raw version
		const rawStart = performance.now();
		sum = 0;
		for (let i = 0; i < rawData.length; i++) {
			sum += rawData[i].value;
		}
		const rawTime = performance.now() - rawStart;

		operationTimes.push({
			operation: `Read ${reactiveData.length} records`,
			reactive: reactiveTime,
			raw: rawTime
		});
	}

	// Benchmark: Mutating data
	function benchmarkMutate() {
		if (reactiveData.length === 0) return;

		// Reactive version - WILL trigger updates
		const reactiveStart = performance.now();
		reactiveData[0].value = Math.random() * 100;
		const reactiveTime = performance.now() - reactiveStart;

		// Raw version - WON'T trigger updates automatically
		const rawStart = performance.now();
		rawData[0].value = Math.random() * 100;
		const rawTime = performance.now() - rawStart;

		operationTimes.push({
			operation: 'Mutate single record',
			reactive: reactiveTime,
			raw: rawTime
		});
	}

	// When you need to update UI with raw data, reassign the array
	function updateRawDataAndRefresh() {
		rawData[0].value = Math.random() * 100;
		rawData = rawData; // Force re-render
	}

	const avgReactive = $derived(
		operationTimes.length > 0
			? operationTimes.reduce((sum, t) => sum + t.reactive, 0) / operationTimes.length
			: 0
	);

	const avgRaw = $derived(
		operationTimes.length > 0
			? operationTimes.reduce((sum, t) => sum + t.raw, 0) / operationTimes.length
			: 0
	);

	const performanceGain = $derived(
		avgReactive > 0 ? ((avgReactive - avgRaw) / avgReactive) * 100 : 0
	);
</script>

<div class="min-h-screen bg-gradient-to-br from-green-50 to-teal-100 p-8">
	<div class="max-w-6xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">$state.raw vs $state</h1>
		<p class="text-gray-600 mb-8">
			Compare performance and understand when to use raw state for optimization
		</p>

		<!-- Performance Stats -->
		<div class="grid grid-cols-3 gap-4 mb-6">
			<div class="bg-white rounded-lg shadow p-6">
				<div class="text-sm text-gray-600 mb-1">Avg Reactive Time</div>
				<div class="text-3xl font-bold text-blue-600">{avgReactive.toFixed(2)}ms</div>
			</div>
			<div class="bg-white rounded-lg shadow p-6">
				<div class="text-sm text-gray-600 mb-1">Avg Raw Time</div>
				<div class="text-3xl font-bold text-green-600">{avgRaw.toFixed(2)}ms</div>
			</div>
			<div class="bg-white rounded-lg shadow p-6">
				<div class="text-sm text-gray-600 mb-1">Performance Gain</div>
				<div class="text-3xl font-bold text-purple-600">{performanceGain.toFixed(1)}%</div>
			</div>
		</div>

		<div class="grid grid-cols-2 gap-6">
			<!-- Controls -->
			<div class="bg-white rounded-lg shadow p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">Benchmark Operations</h2>
				<div class="space-y-2">
					<button
						onclick={() => benchmarkLoad(1000)}
						class="w-full px-4 py-3 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition font-medium"
					>
						Load 1,000 Records
					</button>
					<button
						onclick={() => benchmarkLoad(10000)}
						class="w-full px-4 py-3 bg-blue-600 text-white rounded-lg hover:bg-blue-700 transition font-medium"
					>
						Load 10,000 Records
					</button>
					<button
						onclick={benchmarkRead}
						class="w-full px-4 py-3 bg-green-500 text-white rounded-lg hover:bg-green-600 transition font-medium"
					>
						Read All Records
					</button>
					<button
						onclick={benchmarkMutate}
						class="w-full px-4 py-3 bg-purple-500 text-white rounded-lg hover:bg-purple-600 transition font-medium"
					>
						Mutate Single Record
					</button>
					<button
						onclick={() => (operationTimes = [])}
						class="w-full px-4 py-3 bg-gray-500 text-white rounded-lg hover:bg-gray-600 transition font-medium"
					>
						Clear Benchmarks
					</button>
				</div>

				<!-- Current Data -->
				<div class="mt-6 p-4 bg-gray-50 rounded-lg">
					<div class="text-sm text-gray-600 mb-2">Current Data</div>
					<div class="space-y-1 text-sm">
						<div class="flex justify-between">
							<span class="text-gray-700">Reactive Records:</span>
							<span class="font-mono">{reactiveData.length}</span>
						</div>
						<div class="flex justify-between">
							<span class="text-gray-700">Raw Records:</span>
							<span class="font-mono">{rawData.length}</span>
						</div>
						<div class="flex justify-between">
							<span class="text-gray-700">Last Update:</span>
							<span class="font-mono text-xs">
								{reactiveMetadata.lastUpdate.toLocaleTimeString()}
							</span>
						</div>
					</div>
				</div>
			</div>

			<!-- Benchmark Results -->
			<div class="bg-white rounded-lg shadow p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">Benchmark Results</h2>
				{#if operationTimes.length === 0}
					<div class="text-center py-12 text-gray-500">
						<div class="text-4xl mb-2">⚡</div>
						<div>Run benchmarks to see performance comparison</div>
					</div>
				{:else}
					<div class="space-y-2 max-h-96 overflow-y-auto">
						{#each operationTimes as result (result.operation + Math.random())}
							<div class="border border-gray-200 rounded-lg p-4">
								<div class="font-medium text-gray-900 mb-2">{result.operation}</div>
								<div class="space-y-1">
									<div class="flex items-center gap-2">
										<div class="w-24 text-sm text-gray-600">Reactive:</div>
										<div class="flex-1 bg-blue-100 rounded-full h-6 relative">
											<div
												class="bg-blue-500 h-6 rounded-full flex items-center justify-end pr-2"
												style={`width: ${(result.reactive / Math.max(result.reactive, result.raw)) * 100}%`}
											>
												<span class="text-xs text-white font-medium">
													{result.reactive.toFixed(2)}ms
												</span>
											</div>
										</div>
									</div>
									<div class="flex items-center gap-2">
										<div class="w-24 text-sm text-gray-600">Raw:</div>
										<div class="flex-1 bg-green-100 rounded-full h-6 relative">
											<div
												class="bg-green-500 h-6 rounded-full flex items-center justify-end pr-2"
												style={`width: ${(result.raw / Math.max(result.reactive, result.raw)) * 100}%`}
											>
												<span class="text-xs text-white font-medium">
													{result.raw.toFixed(2)}ms
												</span>
											</div>
										</div>
									</div>
									<div class="text-xs text-gray-600 mt-2">
										Raw is {((result.reactive / result.raw - 1) * 100).toFixed(1)}% faster
									</div>
								</div>
							</div>
						{/each}
					</div>
				{/if}
			</div>
		</div>

		<!-- When to Use Guide -->
		<div class="grid grid-cols-2 gap-6 mt-6">
			<div class="bg-white rounded-lg shadow p-6">
				<h3 class="text-lg font-semibold text-gray-900 mb-4">✅ Use $state (Reactive)</h3>
				<div class="space-y-3 text-sm">
					<div class="border-l-4 border-blue-500 pl-3">
						<div class="font-medium text-gray-900">UI-bound data</div>
						<div class="text-gray-600">Data that's displayed and changes frequently</div>
					</div>
					<div class="border-l-4 border-blue-500 pl-3">
						<div class="font-medium text-gray-900">Small to medium datasets</div>
						<div class="text-gray-600">Under 1,000 items typically</div>
					</div>
					<div class="border-l-4 border-blue-500 pl-3">
						<div class="font-medium text-gray-900">Form inputs</div>
						<div class="text-gray-600">User input that needs immediate feedback</div>
					</div>
					<div class="border-l-4 border-blue-500 pl-3">
						<div class="font-medium text-gray-900">Nested objects</div>
						<div class="text-gray-600">When you need deep reactivity</div>
					</div>
				</div>
			</div>

			<div class="bg-white rounded-lg shadow p-6">
				<h3 class="text-lg font-semibold text-gray-900 mb-4">⚡ Use $state.raw (Performance)</h3>
				<div class="space-y-3 text-sm">
					<div class="border-l-4 border-green-500 pl-3">
						<div class="font-medium text-gray-900">Large datasets</div>
						<div class="text-gray-600">Thousands of records that rarely change</div>
					</div>
					<div class="border-l-4 border-green-500 pl-3">
						<div class="font-medium text-gray-900">Read-only data</div>
						<div class="text-gray-600">Data loaded once and displayed</div>
					</div>
					<div class="border-l-4 border-green-500 pl-3">
						<div class="font-medium text-gray-900">Configuration objects</div>
						<div class="text-gray-600">Static settings that don't change</div>
					</div>
					<div class="border-l-4 border-green-500 pl-3">
						<div class="font-medium text-gray-900">Performance critical code</div>
						<div class="text-gray-600">When you measure a bottleneck</div>
					</div>
				</div>
			</div>
		</div>

		<!-- Code Examples -->
		<div class="bg-white rounded-lg shadow p-6 mt-6">
			<h3 class="text-lg font-semibold text-gray-900 mb-4">Code Patterns</h3>
			<div class="space-y-4 text-sm font-mono">
				<div class="border-l-4 border-blue-500 pl-4 bg-blue-50 p-3 rounded">
					<div class="text-blue-900 mb-2">// Reactive state (automatic updates)</div>
					<div class="text-blue-700">let items = $state([1, 2, 3]);</div>
					<div class="text-blue-700">items.push(4); // ✅ UI updates automatically</div>
				</div>
				<div class="border-l-4 border-green-500 pl-4 bg-green-50 p-3 rounded">
					<div class="text-green-900 mb-2">// Raw state (manual updates)</div>
					<div class="text-green-700">let items = $state.raw([1, 2, 3]);</div>
					<div class="text-green-700">items.push(4);</div>
					<div class="text-green-700">items = items; // ⚠️ Must reassign to update UI</div>
				</div>
				<div class="border-l-4 border-purple-500 pl-4 bg-purple-50 p-3 rounded">
					<div class="text-purple-900 mb-2">// Hybrid approach</div>
					<div class="text-purple-700">let data = $state.raw(largeDataset); // Raw data</div>
					<div class="text-purple-700">
						let metadata = $state({'{ count: 0 }'}); // Reactive metadata
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **$state.raw()**: Skips Proxy wrapping for better performance
- **When to use raw**: Large datasets, read-only data, static config
- **When to use reactive**: UI-bound data, forms, small datasets
- **Manual updates**: Raw state requires reassignment to trigger UI updates
- **Hybrid approach**: Use raw for data, reactive for metadata
- **Measure first**: Only optimize when you have a proven bottleneck

---

## 7. Building a Real-World Kanban Board

### What is This?

A practical example combining deep state reactivity, array mutations, and drag-and-drop to build a project management Kanban board.

**Real-World Scenario:** You're building a project management tool where teams can organize tasks across different stages (To Do, In Progress, Done).

**What it does:** Creates a fully functional Kanban board with drag-and-drop, task management, and deep state updates.

### 📁 Files to Create

**Main Page:**

- `src/routes/kanban/+page.svelte` - Main Kanban board with all logic

**Optional: Break into Components** (for larger apps):

- `src/lib/components/KanbanColumn.svelte` - Individual column component
- `src/lib/components/TaskCard.svelte` - Individual task card component
- `src/lib/types/kanban.ts` - Shared TypeScript interfaces

**Best Practices:**

- Start with a single-file implementation (easier to learn)
- Break into components later if needed for reusability
- Keep state management in the parent page
- Pass data down as props, events up as callbacks

```svelte
<script lang="ts">
	interface Task {
		id: number;
		title: string;
		description: string;
		priority: 'low' | 'medium' | 'high';
		assignee?: string;
		createdAt: Date;
	}

	interface Column {
		id: string;
		title: string;
		tasks: Task[];
	}

	let columns = $state<Column[]>([
		{
			id: 'todo',
			title: 'To Do',
			tasks: [
				{
					id: 1,
					title: 'Design new landing page',
					description: 'Create mockups for the new product landing page',
					priority: 'high',
					assignee: 'Sarah',
					createdAt: new Date('2024-01-15')
				},
				{
					id: 2,
					title: 'Update documentation',
					description: 'Add examples for new API endpoints',
					priority: 'medium',
					assignee: 'Mike',
					createdAt: new Date('2024-01-16')
				}
			]
		},
		{
			id: 'in-progress',
			title: 'In Progress',
			tasks: [
				{
					id: 3,
					title: 'Fix authentication bug',
					description: 'Users getting logged out unexpectedly',
					priority: 'high',
					assignee: 'John',
					createdAt: new Date('2024-01-14')
				}
			]
		},
		{
			id: 'done',
			title: 'Done',
			tasks: [
				{
					id: 4,
					title: 'Deploy staging environment',
					description: 'Set up new staging server',
					priority: 'medium',
					assignee: 'Sarah',
					createdAt: new Date('2024-01-10')
				}
			]
		}
	]);

	let draggedTask: Task | null = null;
	let draggedFromColumn: string | null = null;
	let showNewTaskModal = $state(false);
	let newTaskColumn = $state<string>('todo');
	let newTask = $state({
		title: '',
		description: '',
		priority: 'medium' as 'low' | 'medium' | 'high',
		assignee: ''
	});

	// Computed stats
	const totalTasks = $derived(columns.reduce((sum, col) => sum + col.tasks.length, 0));
	const completedTasks = $derived(columns.find((col) => col.id === 'done')?.tasks.length || 0);
	const inProgressTasks = $derived(
		columns.find((col) => col.id === 'in-progress')?.tasks.length || 0
	);

	function handleDragStart(task: Task, columnId: string) {
		draggedTask = task;
		draggedFromColumn = columnId;
	}

	function handleDragOver(e: DragEvent) {
		e.preventDefault();
	}

	function handleDrop(targetColumnId: string) {
		if (!draggedTask || !draggedFromColumn) return;

		// Remove from source column
		const sourceColumn = columns.find((col) => col.id === draggedFromColumn);
		if (sourceColumn) {
			const taskIndex = sourceColumn.tasks.findIndex((t) => t.id === draggedTask!.id);
			if (taskIndex !== -1) {
				sourceColumn.tasks.splice(taskIndex, 1);
			}
		}

		// Add to target column
		const targetColumn = columns.find((col) => col.id === targetColumnId);
		if (targetColumn) {
			targetColumn.tasks.push(draggedTask);
		}

		draggedTask = null;
		draggedFromColumn = null;
	}

	function addTask() {
		if (!newTask.title.trim()) return;

		const column = columns.find((col) => col.id === newTaskColumn);
		if (column) {
			column.tasks.push({
				id: Date.now(),
				title: newTask.title,
				description: newTask.description,
				priority: newTask.priority,
				assignee: newTask.assignee || undefined,
				createdAt: new Date()
			});
		}

		// Reset form
		newTask = { title: '', description: '', priority: 'medium', assignee: '' };
		showNewTaskModal = false;
	}

	function deleteTask(columnId: string, taskId: number) {
		const column = columns.find((col) => col.id === columnId);
		if (column) {
			column.tasks = column.tasks.filter((t) => t.id !== taskId);
		}
	}

	function getPriorityColor(priority: string) {
		return (
			{
				low: 'rgb(74 222 128)',
				medium: 'rgb(249 115 22)',
				high: 'rgb(239 68 68)'
			}[priority] || 'rgb(156 163 175)'
		);
	}
</script>

<div class="bg-gray-900 min-h-screen p-10 text-gray-200">
	<div class="max-w-screen-2xl mx-auto">
		<!-- Header -->
		<div class="flex justify-between items-center mb-8">
			<div>
				<h1 class="text-blue-400 text-4xl font-extrabold m-0 mb-2">🚀 Project Board</h1>
				<p class="text-gray-500 m-0">Manage your team's workflow</p>
			</div>
			<button
				class="bg-blue-400 text-black px-6 py-3 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5"
				onclick={() => (showNewTaskModal = true)}
			>
				+ New Task
			</button>
		</div>

		<!-- Stats -->
		<div class="grid grid-cols-3 gap-4 mb-8">
			<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 text-center">
				<div class="text-4xl font-extrabold text-blue-400">{totalTasks}</div>
				<div class="text-gray-400 mt-2">Total Tasks</div>
			</div>
			<div class="bg-[#2a2a2a] border-2 border-[#3a3a3a] rounded-xl p-6 text-center">
				<div class="text-4xl font-extrabold text-orange-500">{inProgressTasks}</div>
				<div class="text-gray-400 mt-2">In Progress</div>
			</div>
			<div class="bg-[#2a2a2a] border-2 border-[#3a3a3a] rounded-xl p-6 text-center">
				<div class="text-4xl font-extrabold text-green-400">{completedTasks}</div>
				<div class="text-gray-400 mt-2">Completed</div>
			</div>
		</div>

		<!-- Kanban Columns -->
		<div class="grid grid-cols-3 gap-6">
			{#each columns as column}
				<div
					class="bg-gray-800 border-2 border-gray-700 rounded-xl p-4 min-h-[32rem]"
					ondragover={handleDragOver}
					ondrop={() => handleDrop(column.id)}
				>
					<div class="flex justify-between items-center mb-4">
						<h2 class="text-white text-xl font-bold m-0">{column.title}</h2>
						<span class="bg-gray-700 text-gray-400 px-3 py-1 rounded-full text-sm font-semibold">
							{column.tasks.length}
						</span>
					</div>

					<div class="flex flex-col gap-3">
						{#each column.tasks as task (task.id)}
							<div
								class="bg-gray-900 border border-gray-700 rounded-lg p-4 cursor-move transition-all duration-200 hover:border-blue-400 hover:shadow-xl"
								draggable="true"
								ondragstart={() => handleDragStart(task, column.id)}
							>
								<div class="flex justify-between items-start mb-3">
									<h3 class="text-white font-semibold text-base m-0 flex-1">{task.title}</h3>
									<button
										class="text-gray-600 hover:text-red-400 text-lg bg-transparent border-none cursor-pointer p-0 ml-2"
										onclick={() => deleteTask(column.id, task.id)}
									>
										✕
									</button>
								</div>

								<p class="text-gray-400 text-sm m-0 mb-3 leading-relaxed">
									{task.description}
								</p>

								<div class="flex justify-between items-center">
									<span
										class="px-2 py-1 rounded text-xs font-bold uppercase"
										style="background: {getPriorityColor(task.priority)}; color: rgb(0 0 0);"
									>
										{task.priority}
									</span>

									{#if task.assignee}
										<span class="text-gray-400 text-xs">👤 {task.assignee}</span>
									{/if}
								</div>
							</div>
						{/each}
					</div>
				</div>
			{/each}
		</div>
	</div>

	<!-- New Task Modal -->
	{#if showNewTaskModal}
		<div
			class="fixed inset-0 bg-black/70 flex items-center justify-center z-[1000] backdrop-blur"
			onclick={() => (showNewTaskModal = false)}
		>
			<div
				class="bg-gray-800 border-2 border-gray-700 rounded-2xl max-w-lg w-[90%] p-6"
				onclick={(e) => e.stopPropagation()}
			>
				<h2 class="text-blue-400 text-2xl font-bold m-0 mb-6">Create New Task</h2>

				<div class="flex flex-col gap-4">
					<label class="flex flex-col gap-2">
						<span class="text-gray-400 font-semibold">Title *</span>
						<input
							class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg focus:outline-none focus:border-blue-400"
							type="text"
							bind:value={newTask.title}
							placeholder="Enter task title"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="text-gray-400 font-semibold">Description</span>
						<textarea
							class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg focus:outline-none focus:border-blue-400 min-h-24 resize-none"
							bind:value={newTask.description}
							placeholder="Enter task description"
						></textarea>
					</label>

					<label class="flex flex-col gap-2">
						<span class="text-gray-400 font-semibold">Priority</span>
						<select
							class="bg-[#1a1a1a] border-2 border-[#3a3a3a] text-white p-3 rounded-lg focus:outline-none focus:border-blue-400"
							bind:value={newTask.priority}
						>
							<option value="low">Low</option>
							<option value="medium">Medium</option>
							<option value="high">High</option>
						</select>
					</label>

					<label class="flex flex-col gap-2">
						<span class="text-gray-400 font-semibold">Assignee</span>
						<input
							class="bg-[#1a1a1a] border-2 border-[#3a3a3a] text-white p-3 rounded-lg focus:outline-none focus:border-blue-400"
							type="text"
							bind:value={newTask.assignee}
							placeholder="Assign to team member"
						/>
					</label>

					<label class="flex flex-col gap-2">
						<span class="text-gray-400 font-semibold">Column</span>
						<select
							class="bg-[#1a1a1a] border-2 border-[#3a3a3a] text-white p-3 rounded-lg focus:outline-none focus:border-blue-400"
							bind:value={newTaskColumn}
						>
							{#each columns as column}
								<option value={column.id}>{column.title}</option>
							{/each}
						</select>
					</label>
				</div>

				<div class="flex gap-3 mt-6">
					<button
						class="flex-1 bg-[#3a3a3a] text-white px-6 py-3 rounded-lg font-bold cursor-pointer hover:bg-[#4a4a4a]"
						onclick={() => (showNewTaskModal = false)}
					>
						Cancel
					</button>
					<button
						class="flex-1 bg-blue-400 text-black px-6 py-3 rounded-lg font-bold cursor-pointer hover:bg-[#6ab0ff]"
						onclick={addTask}
					>
						Create Task
					</button>
				</div>
			</div>
		</div>
	{/if}
</div>
```

**Key Concepts:**

- **Drag and Drop**: Native HTML5 drag events (`draggable`, `ondragstart`, `ondrop`)
- **Deep mutations**: Direct array manipulation with `push()`, `splice()`, `filter()`
- **$derived stats**: Computed values automatically update
- **Nested state**: Tasks inside columns, all reactive
- **Real-world patterns**: Modal dialogs, forms, CRUD operations
- **Visual feedback**: Priority badges, hover states, transitions

**What Makes This Realistic:**

1. **Project Management**: Teams actually use Kanban boards daily
2. **Drag-and-drop**: Intuitive UX for moving tasks between stages
3. **Task metadata**: Priority, assignee, descriptions - real data points
4. **Statistics**: Dashboard showing progress at a glance
5. **CRUD operations**: Create, read, update (move), delete tasks
6. **Form validation**: Required fields, modal dialogs
7. **Scalable**: Can easily add more columns, filters, search, etc.

> 💡 **Best Practice**: This 500+ line example is intentionally monolithic for learning. In production, split into:
>
> ```
> KanbanBoard.svelte (orchestrator - 100 lines)
> ├─ KanbanColumn.svelte (column container - 80 lines)
> ├─ TaskCard.svelte (individual task - 60 lines)
> ├─ TaskForm.svelte (create/edit modal - 100 lines)
> ├─ KanbanStats.svelte (statistics display - 40 lines)
> └─ types/kanban.ts (TypeScript definitions)
> ```

**⚠️ Common Mistakes:**

- Don't forget drag event `preventDefault()` - it breaks drag-and-drop
- Don't mutate state during drag events - wait for drop
- Don't skip the key attribute in #each loops - causes render bugs
- Don't forget to validate form inputs before adding tasks

**⚡ Performance Tips:**

- Drag-and-drop with large lists? Consider virtual scrolling
- Deep state is efficient, but avoid unnecessary nested objects
- Use `$state.raw()` for large datasets that don't need reactivity

---

## 📝 Key Takeaways

✅ Proxies and setters/getters enable reactivity
✅ Deep state mutations work automatically
✅ Use $inspect() for debugging reactive state
✅ #each loops display lists with keys
✅ Direct array mutations are reactive
✅ Understand when to use immutable patterns
✅ Complex components combine all concepts

---

## 🚀 Next Steps

After mastering deep state management, you'll be ready for:

- **Section 4**: Advanced state patterns with classes
- **Section 5**: Universal reactivity and shared state
- Building data-intensive applications
