# Section 4: Advanced State Patterns

## 📚 Learning Objectives

By the end of this section, you will:

- Build reactive applications with API integration
- Master async/await patterns with {#await} blocks
- Create reactive classes with methods
- Understand writable derived state
- Work with built-in reactive classes
- Build a complete currency converter application

---

## Table of Contents

- [Section 4: Advanced State Patterns](#section-4-advanced-state-patterns)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Currency Converter - Part 1: Setup \& API](#1-currency-converter---part-1-setup--api)
    - [What are We Building?](#what-are-we-building)
    - [📁 Files to Create](#-files-to-create)
  - [2. Currency Converter - Part 2: UI \& State](#2-currency-converter---part-2-ui--state)
  - [3. Writable Derived State with Getters/Setters](#3-writable-derived-state-with-getterssetters)
    - [What is Writable Derived State?](#what-is-writable-derived-state)
    - [📁 Files to Create](#-files-to-create-1)
  - [4. {#await} Blocks for Async Data](#4-await-blocks-for-async-data)
    - [What are {#await} Blocks?](#what-are-await-blocks)
  - [5. Reactive Classes - Part 1: Basics](#5-reactive-classes---part-1-basics)
    - [What are Reactive Classes?](#what-are-reactive-classes)
  - [6. Reactive Classes - Part 2: Shopping Cart](#6-reactive-classes---part-2-shopping-cart)
    - [Building a Complex Reactive Class](#building-a-complex-reactive-class)
    - [📁 Files to Create](#-files-to-create-2)
  - [7. Built-in Reactive Classes (Set, Map, Date)](#7-built-in-reactive-classes-set-map-date)
    - [What are Built-in Reactive Classes?](#what-are-built-in-reactive-classes)
    - [📁 Files to Create](#-files-to-create-3)
  - [8. API Integration Patterns](#8-api-integration-patterns)
    - [How to Structure API Calls?](#how-to-structure-api-calls)
    - [📁 Files to Create](#-files-to-create-4)
  - [10. 🚀 End-of-Section Project: E-Commerce Shopping Cart](#10--end-of-section-project-e-commerce-shopping-cart)
    - [Project Overview](#project-overview)
    - [📁 Files to Create](#-files-to-create-5)
    - [What This Project Demonstrates](#what-this-project-demonstrates)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Currency Converter - Part 1: Setup & API

### What are We Building?

A real-time currency converter that fetches exchange rates from an API and updates conversions automatically.

**Real-World Scenario:** You're building a multi-currency e-commerce platform that needs live exchange rates.

**What it does:** Sets up API integration and reactive state management.

### 📁 Files to Create

**API Service** (reusable utility):

- `src/lib/services/currencyAPI.ts` - API service with types and caching

**Page:**

- `src/routes/currency-converter/+page.svelte` - Main converter UI

**Best Practices:**

- Keep API logic separate from components in `src/lib/services/`
- Export types and functions from the service module
- Use singleton pattern for services (one instance)
- Cache API responses to reduce network calls

```typescript
// lib/services/currencyAPI.ts
export interface ExchangeRates {
  base: string;
  rates: Record<string, number>;
  timestamp: number;
}

export interface Currency {
  code: string;
  name: string;
  symbol: string;
}

export const popularCurrencies: Currency[] = [
  { code: "USD", name: "US Dollar", symbol: "$" },
  { code: "EUR", name: "Euro", symbol: "€" },
  { code: "GBP", name: "British Pound", symbol: "£" },
  { code: "JPY", name: "Japanese Yen", symbol: "¥" },
  { code: "AUD", name: "Australian Dollar", symbol: "A$" },
  { code: "CAD", name: "Canadian Dollar", symbol: "C$" },
  { code: "CHF", name: "Swiss Franc", symbol: "CHF" },
  { code: "CNY", name: "Chinese Yuan", symbol: "¥" },
  { code: "INR", name: "Indian Rupee", symbol: "₹" },
  { code: "MXN", name: "Mexican Peso", symbol: "Mex$" },
];

class CurrencyService {
  // Map stores cached data with the base currency as key
  private cache: Map<string, ExchangeRates> = new Map();
  // Cache expires after 5 minutes to keep data fresh
  private cacheExpiry = 5 * 60 * 1000; // 5 minutes

  async fetchRates(base: string): Promise<ExchangeRates> {
    // Check cache first to avoid unnecessary API calls
    const cached = this.cache.get(base);
    // If cached data exists and hasn't expired, return it
    if (cached && Date.now() - cached.timestamp < this.cacheExpiry) {
      console.log("📦 Using cached rates for", base);
      return cached;
    }

    console.log("🌐 Fetching rates for", base);

    // Using exchangerate-api.com (free tier)
    const response = await fetch(
      `https://api.exchangerate-api.com/v4/latest/${base}`
    );

    if (!response.ok) {
      throw new Error(`Failed to fetch rates: ${response.statusText}`);
    }

    const data = await response.json();
    const rates: ExchangeRates = {
      base: data.base,
      rates: data.rates,
      timestamp: Date.now(),
    };

    // Cache the result for future requests
    this.cache.set(base, rates);

    return rates;
  }

  async convert(amount: number, from: string, to: string): Promise<number> {
    if (from === to) return amount;

    const rates = await this.fetchRates(from);
    const rate = rates.rates[to];

    if (!rate) {
      throw new Error(`Exchange rate not found for ${to}`);
    }

    return amount * rate;
  }
}

export const currencyService = new CurrencyService();
```

```svelte
<!-- routes/currency-converter-basic.svelte -->
<script lang="ts">
	import { currencyService, popularCurrencies, type Currency } from '$lib/currencyAPI';

	let amount = $state(100);
	let fromCurrency = $state('USD');
	let toCurrency = $state('EUR');
	let converting = $state(false);
	let result = $state<number | null>(null);
	let error = $state<string | null>(null);

	async function convert() {
		if (amount <= 0) {
			error = 'Amount must be greater than 0';
			return;
		}

		converting = true;
		error = null;
		result = null;

		try {
			const converted = await currencyService.convert(amount, fromCurrency, toCurrency);
			result = converted;
		} catch (err) {
			error = err instanceof Error ? err.message : 'Conversion failed';
		} finally {
			converting = false;
		}
	}

	function swap() {
		const temp = fromCurrency;
		fromCurrency = toCurrency;
		toCurrency = temp;
	}

	const fromSymbol = $derived(popularCurrencies.find((c) => c.code === fromCurrency)?.symbol || '');
	const toSymbol = $derived(popularCurrencies.find((c) => c.code === toCurrency)?.symbol || '');
</script>

<div
	class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200 flex flex-col items-center justify-center"
>
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-8">💱 Currency Converter</h1>

	<div class="bg-gray-800 border-2 border-gray-700 rounded-2xl p-8 max-w-lg w-full">
		<div class="mb-6">
			<label class="block text-gray-300 font-semibold mb-2">From</label>
			<div class="relative mb-3">
				<span class="absolute left-4 top-1/2 -translate-y-1/2 text-gray-500 font-bold text-lg"
					>{fromSymbol}</span
				>
				<input
					type="number"
					bind:value={amount}
					min="0"
					step="0.01"
					class="w-full bg-gray-900 border-2 border-gray-700 text-white py-3.5 px-4 pl-10 rounded-lg text-lg font-bold focus:outline-none focus:border-blue-400"
				/>
			</div>
			<select
				bind:value={fromCurrency}
				class="w-full bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-sm cursor-pointer focus:outline-none focus:border-blue-400"
			>
				{#each popularCurrencies as currency}
					<option value={currency.code}>
						{currency.code} - {currency.name}
					</option>
				{/each}
			</select>
		</div>

		<button
			onclick={swap}
			aria-label="Swap currencies"
			class="w-full bg-gray-700 text-white border-none p-3 rounded-lg text-2xl cursor-pointer mb-6 transition-all duration-200 hover:bg-gray-600 hover:rotate-180"
		>
			⇄
		</button>

		<div class="mb-6">
			<label class="block text-gray-300 font-semibold mb-2">To</label>
			<div class="relative mb-3">
				<span class="absolute left-4 top-1/2 -translate-y-1/2 text-gray-500 font-bold text-lg"
					>{toSymbol}</span
				>
				<input
					type="number"
					value={result?.toFixed(2) || ''}
					readonly
					placeholder="Result"
					class="w-full bg-gray-900 border-2 border-gray-700 text-green-400 py-3.5 px-4 pl-10 rounded-lg text-lg font-bold focus:outline-none focus:border-blue-400"
				/>
			</div>
			<select
				bind:value={toCurrency}
				class="w-full bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-sm cursor-pointer focus:outline-none focus:border-blue-400"
			>
				{#each popularCurrencies as currency}
					<option value={currency.code}>
						{currency.code} - {currency.name}
					</option>
				{/each}
			</select>
		</div>

		<button
			onclick={convert}
			disabled={converting}
			class="w-full bg-blue-400 text-black border-none p-4 rounded-lg text-lg font-bold cursor-pointer transition-all duration-200 mb-4 hover:bg-blue-300 hover:-translate-y-0.5 disabled:opacity-50 disabled:cursor-not-allowed"
		>
			{converting ? 'Converting...' : 'Convert'}
		</button>

		{#if error}
			<div class="bg-red-400/10 border-2 border-red-400 text-red-400 p-4 rounded-lg font-semibold">
				⚠️ {error}
			</div>
		{/if}

		{#if result !== null && !error}
			<div
				class="bg-green-400/10 border-2 border-green-400 text-green-400 p-4 rounded-lg font-bold text-center text-lg"
			>
				✅ {fromSymbol}{amount.toFixed(2)} = {toSymbol}{result.toFixed(2)}
			</div>
		{/if}
	</div>
</div>
```

**Key Concepts:**

- **Service class**: Encapsulates API logic
- **Caching**: Reduces API calls
- **Error handling**: Try/catch for failures
- **Loading states**: Track async operations

---

## 2. Currency Converter - Part 2: UI & State

_(Complete interactive version with real-time conversion)_

---

## 3. Writable Derived State with Getters/Setters

### What is Writable Derived State?

Create derived values that can also be set, with custom get/set logic.

**Real-World Scenario:** You're building a temperature converter where you can edit either Celsius or Fahrenheit.

**What it does:** Demonstrates bidirectional derived state.

### 📁 Files to Create

Create:

- `src/routes/writable-derived/+page.svelte`

```svelte
<script lang="ts">
	// Temperature converter with writable derived state
	let celsius = $state(20);

	// Writable derived: can get AND set (bidirectional binding)
	// When you READ fahrenheit.value, it computes from celsius
	// When you WRITE to fahrenheit.value, it updates celsius
	let fahrenheit = {
		get value() {
			// Convert celsius to fahrenheit when reading
			return (celsius * 9) / 5 + 32;
		},
		set value(f: number) {
			// Convert fahrenheit back to celsius when writing
			celsius = ((f - 32) * 5) / 9;
		}
	};

	// Distance converter
	let kilometers = $state(10);

	let miles = {
		get value() {
			return kilometers * 0.621371;
		},
		set value(m: number) {
			kilometers = m / 0.621371;
		}
	};

	// Price with tax
	let basePrice = $state(100);
	let taxRate = $state(0.2); // 20%

	let totalPrice = {
		get value() {
			return basePrice * (1 + taxRate);
		},
		set value(total: number) {
			basePrice = total / (1 + taxRate);
		}
	};
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 text-4xl font-bold m-0 mb-10">🔄 Writable Derived State</h1>

	<div class="max-w-7xl mx-auto grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">🌡️ Temperature</h2>
			<div class="flex items-center gap-4 mb-4">
				<div class="flex-1 relative">
					<label class="block text-gray-300 text-sm mb-2 font-semibold">Celsius</label>
					<input
						type="number"
						bind:value={celsius}
						step="0.1"
						class="w-full bg-gray-900 border-2 border-gray-700 text-white py-3 pr-12 pl-3 rounded-lg text-xl font-bold focus:outline-none focus:border-blue-400"
					/>
					<span class="absolute right-3 bottom-3 text-gray-500 font-bold">°C</span>
				</div>
				<div class="text-blue-400 text-2xl font-bold">⇄</div>
				<div class="flex-1 relative">
					<label class="block text-gray-300 text-sm mb-2 font-semibold">Fahrenheit</label>
					<input
						type="number"
						bind:value={fahrenheit.value}
						step="0.1"
						class="w-full bg-gray-900 border-2 border-gray-700 text-white py-3 pr-12 pl-3 rounded-lg text-xl font-bold focus:outline-none focus:border-blue-400"
					/>
					<span class="absolute right-3 bottom-3 text-gray-500 font-bold">°F</span>
				</div>
			</div>
			<p class="text-gray-500 text-sm italic text-center m-0">
				Edit either value - they stay in sync!
			</p>
		</div>

		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">📏 Distance</h2>
			<div class="flex items-center gap-4 mb-4">
				<div class="flex-1 relative">
					<label class="block text-gray-300 text-sm mb-2 font-semibold">Kilometers</label>
					<input
						type="number"
						bind:value={kilometers}
						step="0.1"
						class="w-full bg-gray-900 border-2 border-gray-700 text-white py-3 pr-12 pl-3 rounded-lg text-xl font-bold focus:outline-none focus:border-blue-400"
					/>
					<span class="absolute right-3 bottom-3 text-gray-500 font-bold">km</span>
				</div>
				<div class="text-blue-400 text-2xl font-bold">⇄</div>
				<div class="flex-1 relative">
					<label class="block text-gray-300 text-sm mb-2 font-semibold">Miles</label>
					<input
						type="number"
						bind:value={miles.value}
						step="0.1"
						class="w-full bg-gray-900 border-2 border-gray-700 text-white py-3 pr-12 pl-3 rounded-lg text-xl font-bold focus:outline-none focus:border-blue-400"
					/>
					<span class="absolute right-3 bottom-3 text-gray-500 font-bold">mi</span>
				</div>
			</div>
		</div>

		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-2xl font-bold">💰 Price with Tax</h2>
			<div class="flex flex-col gap-5">
				<label class="flex flex-col gap-2 text-gray-300 font-semibold">
					Base Price:
					<input
						type="number"
						bind:value={basePrice}
						step="1"
						class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base"
					/>
				</label>

				<label class="flex flex-col gap-2 text-gray-300 font-semibold">
					Tax Rate: {(taxRate * 100).toFixed(0)}%
					<input
						type="range"
						bind:value={taxRate}
						min="0"
						max="0.5"
						step="0.01"
						class="w-full cursor-pointer accent-blue-400"
					/>
				</label>

				<label class="flex flex-col gap-2 text-green-400 font-semibold">
					Total (editable):
					<input
						type="number"
						bind:value={totalPrice.value}
						step="0.01"
						class="bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base"
					/>
				</label>

				<div class="bg-gray-900 border border-gray-700 rounded-lg p-4">
					<div class="flex justify-between py-2 border-b border-gray-800">
						<span>Base:</span>
						<span>${basePrice.toFixed(2)}</span>
					</div>
					<div class="flex justify-between py-2 border-b border-gray-800">
						<span>Tax:</span>
						<span>${(basePrice * taxRate).toFixed(2)}</span>
					</div>
					<div
						class="flex justify-between py-2 mt-2 pt-4 border-t-2 border-gray-700 text-xl font-extrabold text-blue-400"
					>
						<span>Total:</span>
						<span>${totalPrice.value.toFixed(2)}</span>
					</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Bidirectional binding**: Can read and write derived values
- **Getters**: Compute value from source
- **Setters**: Update source from computed value
- **Synchronized state**: Edit either side, both update

---

## 4. {#await} Blocks for Async Data

### What are {#await} Blocks?

Handle promises in templates with loading, success, and error states.

**Real-World Scenario:** You're building a user profile page that loads data from an API.

**What it does:** Shows all {#await} patterns for async data fetching.

```svelte
<script lang="ts">
	interface User {
		id: number;
		name: string;
		email: string;
		avatar: string;
		bio: string;
	}

	interface Post {
		id: number;
		title: string;
		content: string;
		likes: number;
	}

	// Simulate API calls
	function delay(ms: number) {
		return new Promise(resolve => setTimeout(resolve, ms));
	}

	async function fetchUser(id: number): Promise<User> {
		await delay(1500);
		if (Math.random() < 0.1) throw new Error('Failed to load user');
		return {
			id,
			name: 'Jane Developer',
			email: 'jane@example.com',
			avatar: '👩‍💻',
			bio: 'Full-stack developer passionate about web technologies'
		};
	}

	async function fetchPosts(userId: number): Promise<Post[]> {
		await delay(2000);
		if (Math.random() < 0.1) throw new Error('Failed to load posts');
		return [
			{ id: 1, title: 'My First Post', content: 'Hello world!', likes: 42 },
			{ id: 2, title: 'Svelte is Amazing', content: 'Love the reactivity!', likes: 128 },
			{ id: 3, title: 'Tips for Beginners', content: 'Start small...', likes: 73 }
		];
	}

	let userId = $state(1);
	let userPromise = $state(fetchUser(userId));
	let postsPromise = $state(fetchPosts(userId));

	function reload() {
		userPromise = fetchUser(userId);
		postsPromise = fetchPosts(userId);
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-6">⏳ {#await} Blocks Demo</h1>

	<button onclick={reload} class="block mx-auto mb-8 bg-blue-400 text-black border-none py-3 px-8 rounded-lg font-bold cursor-pointer transition-all duration-200 hover:bg-blue-300 hover:-translate-y-0.5">
		🔄 Reload Data
	</button>

	<div class="max-w-7xl mx-auto grid grid-cols-[repeat(auto-fit,minmax(300px,1fr))] gap-6">
		<!-- Pattern 1: Just the resolved value -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-lg">Pattern 1: Success Only</h2>
			{#await userPromise then user}
				<div class="flex items-center gap-3 text-lg">
					<span class="text-3xl">{user.avatar}</span>
					<span>{user.name}</span>
				</div>
			{/await}
		</div>

		<!-- Pattern 2: Pending and resolved -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-lg">Pattern 2: Loading + Success</h2>
			{#await userPromise}
				<div class="flex items-center justify-center gap-3 py-8 text-gray-500 italic">Loading user...</div>
			{:then user}
				<div class="text-center">
					<div class="text-6xl mb-4">{user.avatar}</div>
					<h3 class="m-0 mb-2 text-white">{user.name}</h3>
					<p class="text-gray-500 my-2">{user.email}</p>
				</div>
			{/await}
		</div>

		<!-- Pattern 3: All states -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
			<h2 class="m-0 mb-5 text-white text-lg">Pattern 3: All States</h2>
			{#await userPromise}
				<div class="flex items-center justify-center gap-3 py-8 text-gray-500 italic">
					<div class="w-6 h-6 border-4 border-gray-700 border-t-blue-400 rounded-full animate-spin"></div>
					Loading user profile...
				</div>
			{:then user}
				<div class="text-center">
					<div class="text-6xl mb-4">{user.avatar}</div>
					<h3 class="m-0 mb-2 text-white">{user.name}</h3>
					<p class="text-gray-500 my-2">{user.email}</p>
					<p class="text-gray-300 mt-4 mb-0 leading-relaxed">{user.bio}</p>
				</div>
			{:catch error}
				<div class="bg-red-400/10 border-2 border-red-400 text-red-400 p-4 rounded-lg font-semibold">
					⚠️ {error.message}
				</div>
			{/await}
		</div>

		<!-- Pattern 4: Multiple awaits -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 col-span-full">
			<h2 class="m-0 mb-5 text-white text-lg">Pattern 4: Multiple Promises</h2>
			<div class="grid grid-cols-2 gap-6">
				<div>
					<h3 class="m-0 mb-2 text-white">User</h3>
					{#await userPromise}
						<div class="flex items-center justify-center gap-3 p-4 text-gray-500 italic">Loading...</div>
					{:then user}
						<div class="flex flex-col gap-1">
							<strong>{user.name}</strong>
							<small class="text-gray-500 text-xs">{user.email}</small>
						</div>
					{:catch}
						<div class="bg-red-400/10 border-2 border-red-400 text-red-400 p-2 rounded-lg font-semibold text-sm">Failed</div>
					{/await}
				</div>

				<div>
					<h3 class="m-0 mb-2 text-white">Posts</h3>
					{#await postsPromise}
						<div class="flex items-center justify-center gap-3 p-4 text-gray-500 italic">Loading...</div>
					{:then posts}
						<div class="bg-blue-400 text-black p-3 rounded-lg font-bold text-center">
							📝 {posts.length} posts
						</div>
					{:catch}
						<div class="bg-red-400/10 border-2 border-red-400 text-red-400 p-2 rounded-lg font-semibold text-sm">Failed</div>
					{/await}
				</div>
			</div>
		</div>

		<!-- Pattern 5: Detailed posts list -->
		<div class="bg-gray-800 border-2 border-gray-700 rounded-xl p-6 col-span-full">
			<h2 class="m-0 mb-5 text-white text-lg">User Posts</h2>
			{#await postsPromise}
				<div class="flex items-center justify-center gap-3 py-8 text-gray-500 italic">
					<div class="w-6 h-6 border-4 border-gray-700 border-t-blue-400 rounded-full animate-spin"></div>
					Loading posts...
				</div>
			{:then posts}
				<div class="flex flex-col gap-4">
					{#each posts as post}
						<div class="bg-gray-900 border border-gray-700 rounded-lg p-4">
							<h4 class="m-0 mb-2 text-blue-400">{post.title}</h4>
							<p class="text-gray-300 my-2">{post.content}</p>
							<div class="text-red-400 font-bold mt-3">❤️ {post.likes}</div>
						</div>
					{/each}
				</div>
			{:catch error}
				<div class="bg-red-400/10 border-2 border-red-400 text-red-400 p-4 rounded-lg font-semibold">
					⚠️ {error.message}
					<button onclick={reload} class="mt-3 bg-red-400 text-white border-none py-2 px-4 rounded-md cursor-pointer">Try Again</button>
				</div>
			{/await}
		</div>
	</div>
</div>

<style>
	@keyframes spin {
		to { transform: rotate(360deg); }
	}
</style>
```

**Key Concepts:**

- **{#await promise}**: Pending state
- **{:then value}**: Success state
- **{:catch error}**: Error state
- **Multiple awaits**: Handle multiple promises
- **Reactive promises**: Re-run when promise changes

---

## 5. Reactive Classes - Part 1: Basics

### What are Reactive Classes?

Classes with reactive properties that trigger UI updates when methods are called.

**Real-World Scenario:** You're building a todo list with methods to add, complete, and filter tasks.

**What it does:** Shows how to make class instances reactive.

```svelte
<script lang="ts">
	interface TodoItem {
		id: number;
		text: string;
		completed: boolean;
		createdAt: Date;
	}

	class TodoList {
		// $state makes properties reactive - UI updates when they change
		items = $state<TodoItem[]>([]);
		filter = $state<'all' | 'active' | 'completed'>('all');

		// Computed property: auto-recalculates when items or filter changes
		get filtered() {
			switch (this.filter) {
				case 'active':
					return this.items.filter((t) => !t.completed);
				case 'completed':
					return this.items.filter((t) => t.completed);
				default:
					return this.items;
			}
		}

		// Another computed property for statistics
		get stats() {
			return {
				total: this.items.length,
				active: this.items.filter((t) => !t.completed).length,
				completed: this.items.filter((t) => t.completed).length
			};
		}

		// Method to add a new todo - UI updates automatically when items array changes
		add(text: string) {
			if (!text.trim()) return;

			// Array.push() is reactive - triggers UI update
			this.items.push({
				id: Date.now(),
				text: text.trim(),
				completed: false,
				createdAt: new Date()
			});
		}

		toggle(id: number) {
			const item = this.items.find((t) => t.id === id);
			if (item) {
				item.completed = !item.completed;
			}
		}

		remove(id: number) {
			this.items = this.items.filter((t) => t.id !== id);
		}

		clearCompleted() {
			this.items = this.items.filter((t) => !t.completed);
		}
	}

	const todos = new TodoList();
	let newTodo = $state('');

	function handleAdd() {
		todos.add(newTodo);
		newTodo = '';
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-8">✅ Reactive Class Todo List</h1>

	<div class="max-w-2xl mx-auto bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
		<div class="flex gap-3 mb-6">
			<input
				bind:value={newTodo}
				placeholder="What needs to be done?"
				onkeydown={(e) => e.key === 'Enter' && handleAdd()}
				class="flex-1 bg-gray-900 border-2 border-gray-700 text-white py-3.5 px-3.5 rounded-lg text-base focus:outline-none focus:border-blue-400"
			/>
			<button
				onclick={handleAdd}
				class="bg-blue-400 text-black border-none py-3.5 px-7 rounded-lg font-bold cursor-pointer"
				>Add</button
			>
		</div>

		<div class="flex gap-2 mb-5">
			<button
				class="flex-1 bg-gray-700 text-gray-300 border-none py-2.5 px-2.5 rounded-md cursor-pointer transition-all duration-200"
				class:!bg-blue-400={todos.filter === 'all'}
				class:!text-black={todos.filter === 'all'}
				class:!font-bold={todos.filter === 'all'}
				onclick={() => (todos.filter = 'all')}
			>
				All ({todos.stats.total})
			</button>
			<button
				class="flex-1 bg-gray-700 text-gray-300 border-none py-2.5 px-2.5 rounded-md cursor-pointer transition-all duration-200"
				class:!bg-blue-400={todos.filter === 'active'}
				class:!text-black={todos.filter === 'active'}
				class:!font-bold={todos.filter === 'active'}
				onclick={() => (todos.filter = 'active')}
			>
				Active ({todos.stats.active})
			</button>
			<button
				class="flex-1 bg-gray-700 text-gray-300 border-none py-2.5 px-2.5 rounded-md cursor-pointer transition-all duration-200"
				class:!bg-blue-400={todos.filter === 'completed'}
				class:!text-black={todos.filter === 'completed'}
				class:!font-bold={todos.filter === 'completed'}
				onclick={() => (todos.filter = 'completed')}
			>
				Completed ({todos.stats.completed})
			</button>
		</div>

		<div class="flex flex-col gap-2 mb-5 min-h-48">
			{#each todos.filtered as todo}
				<div
					class="bg-gray-900 border border-gray-700 rounded-lg p-4 flex items-center gap-3 transition-all duration-200"
					class:opacity-60={todo.completed}
				>
					<input
						type="checkbox"
						checked={todo.completed}
						onchange={() => todos.toggle(todo.id)}
						class="w-5 h-5 cursor-pointer accent-green-400"
					/>
					<span
						class="flex-1"
						class:line-through={todo.completed}
						class:text-gray-600={todo.completed}>{todo.text}</span
					>
					<button
						onclick={() => todos.remove(todo.id)}
						class="bg-transparent border-none cursor-pointer text-lg opacity-70 transition-opacity duration-200 hover:opacity-100"
					>
						🗑️
					</button>
				</div>
			{:else}
				<p class="text-center text-gray-600 py-10 italic">
					No tasks {todos.filter !== 'all' ? `(${todos.filter})` : ''}
				</p>
			{/each}
		</div>

		{#if todos.stats.completed > 0}
			<button
				onclick={() => todos.clearCompleted()}
				class="w-full bg-red-400 text-white border-none p-3 rounded-lg font-bold cursor-pointer"
			>
				Clear Completed
			</button>
		{/if}
	</div>
</div>
```

**Key Concepts:**

- **$state in classes**: Make class properties reactive
- **Getters**: Computed properties (filtered, stats)
- **Methods**: Modify state and UI updates
- **Encapsulation**: All logic in one place

---

## 6. Reactive Classes - Part 2: Shopping Cart

### Building a Complex Reactive Class

A more advanced example showing a shopping cart with multiple methods, computed properties, and complex state management.

**Real-World Scenario:** You're building an e-commerce platform that needs a shopping cart with quantity management, discounts, and tax calculations.

**What it does:** Creates a full-featured ShoppingCart class with reactive state.

### 📁 Files to Create

**Main Demo:**

- `src/routes/section-4/shopping-cart/+page.svelte` - Shopping cart with reactive class

```svelte
<script lang="ts">
	interface Product {
		id: number;
		name: string;
		price: number;
		image: string;
		category: string;
	}

	interface CartItem {
		product: Product;
		quantity: number;
	}

	class ShoppingCart {
		// Reactive state: cart items update UI automatically
		items = $state<CartItem[]>([]);
		discountCode = $state<string>('');
		taxRate = $state<number>(0.08); // 8% tax

		// Available products (not reactive - static data)
		products: Product[] = [
			{
				id: 1,
				name: 'Wireless Headphones',
				price: 79.99,
				image: '🎧',
				category: 'Electronics'
			},
			{
				id: 2,
				name: 'Smart Watch',
				price: 199.99,
				image: '⌚',
				category: 'Electronics'
			},
			{
				id: 3,
				name: 'Laptop Stand',
				price: 49.99,
				image: '💻',
				category: 'Accessories'
			},
			{
				id: 4,
				name: 'Mechanical Keyboard',
				price: 129.99,
				image: '⌨️',
				category: 'Electronics'
			},
			{
				id: 5,
				name: 'USB-C Hub',
				price: 39.99,
				image: '🔌',
				category: 'Accessories'
			},
			{
				id: 6,
				name: 'Desk Lamp',
				price: 34.99,
				image: '💡',
				category: 'Furniture'
			}
		];

		// Computed: Subtotal before tax and discount
		// Automatically recalculates when items array changes
		get subtotal(): number {
			return this.items.reduce((sum, item) => sum + item.product.price * item.quantity, 0);
		}

		// Computed: Discount amount based on discount code
		// Updates automatically when discountCode or subtotal changes
		get discount(): number {
			const validCodes: Record<string, number> = {
				SAVE10: 0.1,
				SAVE20: 0.2,
				SAVE50: 0.5
			};

			const rate = validCodes[this.discountCode.toUpperCase()] || 0;
			return this.subtotal * rate;
		}

		// Computed: Subtotal after discount
		get discountedSubtotal(): number {
			return this.subtotal - this.discount;
		}

		// Computed: Tax amount
		get tax(): number {
			return this.discountedSubtotal * this.taxRate;
		}

		// Computed: Final total
		get total(): number {
			return this.discountedSubtotal + this.tax;
		}

		// Computed: Total items count
		get itemCount(): number {
			return this.items.reduce((sum, item) => sum + item.quantity, 0);
		}

		// Computed: Is discount valid
		get isDiscountValid(): boolean {
			return this.discount > 0;
		}

		// Add product to cart
		addProduct(product: Product) {
			const existingItem = this.items.find((item) => item.product.id === product.id);

			if (existingItem) {
				existingItem.quantity++;
			} else {
				this.items.push({
					product,
					quantity: 1
				});
			}
		}

		// Remove product from cart
		removeProduct(productId: number) {
			this.items = this.items.filter((item) => item.product.id !== productId);
		}

		// Update quantity
		updateQuantity(productId: number, quantity: number) {
			const item = this.items.find((item) => item.product.id === productId);
			if (item) {
				if (quantity <= 0) {
					this.removeProduct(productId);
				} else {
					item.quantity = quantity;
				}
			}
		}

		// Clear cart
		clear() {
			this.items = [];
			this.discountCode = '';
		}

		// Apply discount code
		applyDiscount(code: string) {
			this.discountCode = code;
		}

		// Get items by category
		getItemsByCategory(): Record<string, CartItem[]> {
			const grouped: Record<string, CartItem[]> = {};

			for (const item of this.items) {
				const category = item.product.category;
				if (!grouped[category]) {
					grouped[category] = [];
				}
				grouped[category].push(item);
			}

			return grouped;
		}
	}

	const cart = new ShoppingCart();
	let discountInput = $state('');
	let selectedCategory = $state<string>('all');

	const categories = $derived(['all', ...new Set(cart.products.map((p) => p.category))]);

	const filteredProducts = $derived(
		selectedCategory === 'all'
			? cart.products
			: cart.products.filter((p) => p.category === selectedCategory)
	);

	function handleApplyDiscount() {
		cart.applyDiscount(discountInput);
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-orange-50 to-red-100 p-8">
	<div class="max-w-7xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">Shopping Cart with Reactive Class</h1>
		<p class="text-gray-600 mb-8">Complex state management with computed properties and methods</p>

		<div class="grid grid-cols-3 gap-6">
			<!-- Products Catalog -->
			<div class="col-span-2 space-y-6">
				<!-- Category Filter -->
				<div class="bg-white rounded-lg shadow p-4">
					<div class="flex items-center gap-2">
						<span class="text-sm font-medium text-gray-700">Category:</span>
						{#each categories as category}
							<button
								onclick={() => (selectedCategory = category)}
								class={`px-4 py-2 rounded-lg text-sm font-medium transition ${
									selectedCategory === category
										? 'bg-orange-500 text-white'
										: 'bg-gray-100 text-gray-700 hover:bg-gray-200'
								}`}
							>
								{category.charAt(0).toUpperCase() + category.slice(1)}
							</button>
						{/each}
					</div>
				</div>

				<!-- Products Grid -->
				<div class="grid grid-cols-2 gap-4">
					{#each filteredProducts as product (product.id)}
						<div class="bg-white rounded-lg shadow p-6">
							<div class="text-6xl mb-4 text-center">{product.image}</div>
							<h3 class="text-lg font-semibold text-gray-900 mb-2">{product.name}</h3>
							<div class="text-sm text-gray-600 mb-3">{product.category}</div>
							<div class="flex items-center justify-between">
								<div class="text-2xl font-bold text-orange-600">
									${product.price.toFixed(2)}
								</div>
								<button
									onclick={() => cart.addProduct(product)}
									class="px-4 py-2 bg-orange-500 text-white rounded-lg hover:bg-orange-600 transition font-medium"
								>
									Add to Cart
								</button>
							</div>
						</div>
					{/each}
				</div>
			</div>

			<!-- Cart Sidebar -->
			<div class="space-y-4">
				<!-- Cart Items -->
				<div class="bg-white rounded-lg shadow p-6">
					<div class="flex items-center justify-between mb-4">
						<h2 class="text-xl font-semibold text-gray-900">
							Cart ({cart.itemCount})
						</h2>
						{#if cart.items.length > 0}
							<button onclick={() => cart.clear()} class="text-sm text-red-500 hover:text-red-700">
								Clear
							</button>
						{/if}
					</div>

					{#if cart.items.length === 0}
						<div class="text-center py-8 text-gray-500">
							<div class="text-4xl mb-2">🛒</div>
							<div class="text-sm">Your cart is empty</div>
						</div>
					{:else}
						<div class="space-y-3 mb-4">
							{#each cart.items as item (item.product.id)}
								<div class="border border-gray-200 rounded-lg p-3">
									<div class="flex items-start gap-3">
										<div class="text-3xl">{item.product.image}</div>
										<div class="flex-1 min-w-0">
											<div class="text-sm font-medium text-gray-900 truncate">
												{item.product.name}
											</div>
											<div class="text-sm text-gray-600">
												${item.product.price.toFixed(2)}
											</div>
											<div class="flex items-center gap-2 mt-2">
												<button
													onclick={() => cart.updateQuantity(item.product.id, item.quantity - 1)}
													class="w-6 h-6 bg-gray-200 rounded text-gray-700 hover:bg-gray-300 text-sm"
												>
													-
												</button>
												<span class="text-sm font-medium w-8 text-center">
													{item.quantity}
												</span>
												<button
													onclick={() => cart.updateQuantity(item.product.id, item.quantity + 1)}
													class="w-6 h-6 bg-gray-200 rounded text-gray-700 hover:bg-gray-300 text-sm"
												>
													+
												</button>
												<button
													onclick={() => cart.removeProduct(item.product.id)}
													class="ml-auto text-red-500 hover:text-red-700 text-xs"
												>
													Remove
												</button>
											</div>
										</div>
									</div>
								</div>
							{/each}
						</div>
					{/if}
				</div>

				<!-- Discount Code -->
				{#if cart.items.length > 0}
					<div class="bg-white rounded-lg shadow p-6">
						<h3 class="text-sm font-semibold text-gray-900 mb-3">Discount Code</h3>
						<div class="flex gap-2">
							<input
								type="text"
								bind:value={discountInput}
								placeholder="Enter code"
								class="flex-1 px-3 py-2 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-orange-500"
							/>
							<button
								onclick={handleApplyDiscount}
								class="px-4 py-2 bg-orange-500 text-white rounded hover:bg-orange-600 transition text-sm font-medium"
							>
								Apply
							</button>
						</div>
						{#if cart.isDiscountValid}
							<div class="mt-2 text-sm text-green-600">
								✓ Discount code applied: {cart.discountCode}
							</div>
						{:else if cart.discountCode}
							<div class="mt-2 text-sm text-red-600">✗ Invalid discount code</div>
						{/if}
						<div class="mt-3 text-xs text-gray-500">Try: SAVE10, SAVE20, SAVE50</div>
					</div>

					<!-- Order Summary -->
					<div class="bg-white rounded-lg shadow p-6">
						<h3 class="text-lg font-semibold text-gray-900 mb-4">Order Summary</h3>
						<div class="space-y-2 text-sm">
							<div class="flex justify-between text-gray-700">
								<span>Subtotal:</span>
								<span>${cart.subtotal.toFixed(2)}</span>
							</div>
							{#if cart.discount > 0}
								<div class="flex justify-between text-green-600">
									<span>Discount:</span>
									<span>-${cart.discount.toFixed(2)}</span>
								</div>
							{/if}
							<div class="flex justify-between text-gray-700">
								<span>Tax ({(cart.taxRate * 100).toFixed(0)}%):</span>
								<span>${cart.tax.toFixed(2)}</span>
							</div>
							<div class="border-t border-gray-200 pt-2 mt-2">
								<div class="flex justify-between text-lg font-bold text-gray-900">
									<span>Total:</span>
									<span>${cart.total.toFixed(2)}</span>
								</div>
							</div>
						</div>
						<button
							class="w-full mt-4 px-4 py-3 bg-orange-500 text-white rounded-lg hover:bg-orange-600 transition font-semibold"
						>
							Checkout
						</button>
					</div>
				{/if}
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Complex computed properties**: Multiple derived values from state
- **Discount logic**: Conditional computations
- **Quantity management**: Update individual item quantities
- **Category grouping**: Organize data by properties
- **Form integration**: Apply discount codes reactively

---

## 7. Built-in Reactive Classes (Set, Map, Date)

### What are Built-in Reactive Classes?

Svelte 5 makes JavaScript's built-in classes like Set, Map, and Date automatically reactive when wrapped with `$state()`.

**Real-World Scenario:** You're building a collaborative app that tracks unique users, stores key-value pairs, and displays timestamps.

**What it does:** Demonstrates how Set, Map, and Date work reactively in Svelte 5.

### 📁 Files to Create

**Main Demo:**

- `src/routes/section-4/reactive-builtins/+page.svelte` - Built-in reactive classes demo

```svelte
<script lang="ts">
	// Reactive Set - tracks unique values
	let tags = $state(new Set<string>(['svelte', 'typescript', 'vite']));
	let newTag = $state('');

	// Reactive Map - key-value pairs
	let userSettings = $state(
		new Map<string, string | number | boolean>([
			['theme', 'dark'],
			['fontSize', 16],
			['notifications', true],
			['language', 'en']
		])
	);
	let newKey = $state('');
	let newValue = $state('');

	// Reactive Date - timestamps
	let events = $state<{ id: number; name: string; timestamp: Date }[]>([
		{ id: 1, name: 'App launched', timestamp: new Date() },
		{ id: 2, name: 'User logged in', timestamp: new Date(Date.now() - 3600000) },
		{ id: 3, name: 'Data synced', timestamp: new Date(Date.now() - 7200000) }
	]);
	let eventName = $state('');

	// Set operations
	function addTag() {
		// Check for duplicates (Set automatically handles this, but validate input)
		if (!newTag.trim() || tags.has(newTag.trim())) return;
		// Add the new tag to the Set
		tags.add(newTag.trim().toLowerCase());
		// IMPORTANT: Reassign to trigger reactivity for built-in classes
		tags = tags;
		newTag = '';
	}

	function removeTag(tag: string) {
		// Remove tag from Set
		tags.delete(tag);
		// IMPORTANT: Reassign to trigger reactivity
		tags = tags;
	}

	function clearTags() {
		// Clear all tags from Set
		tags.clear();
		// IMPORTANT: Reassign to trigger reactivity
		tags = tags;
	}

	// Map operations
	function addSetting() {
		if (!newKey.trim()) return;

		// Try to parse as number or boolean for proper typing
		let value: string | number | boolean = newValue;
		if (!isNaN(Number(newValue))) {
			value = Number(newValue);
		} else if (newValue === 'true') {
			value = true;
		} else if (newValue === 'false') {
			value = false;
		}

		// Add key-value pair to Map
		userSettings.set(newKey, value);
		// IMPORTANT: Reassign to trigger reactivity for built-in classes
		userSettings = userSettings;
		newKey = '';
		newValue = '';
	}

	function removeSetting(key: string) {
		// Remove key from Map
		userSettings.delete(key);
		// IMPORTANT: Reassign to trigger reactivity
		userSettings = userSettings;
	}

	function clearSettings() {
		// Clear all entries from Map
		userSettings.clear();
		// IMPORTANT: Reassign to trigger reactivity
		userSettings = userSettings;
	}

	// Date operations
	function addEvent() {
		if (!eventName.trim()) return;

		events.push({
			id: Date.now(),
			name: eventName,
			timestamp: new Date()
		});

		eventName = '';
	}

	function removeEvent(id: number) {
		events = events.filter((e) => e.id !== id);
	}

	function getTimeAgo(date: Date): string {
		const seconds = Math.floor((Date.now() - date.getTime()) / 1000);

		if (seconds < 60) return `${seconds}s ago`;
		if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`;
		if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`;
		return `${Math.floor(seconds / 86400)}d ago`;
	}

	// Computed values
	const tagCount = $derived(tags.size);
	const settingsCount = $derived(userSettings.size);
	const eventCount = $derived(events.length);
	const tagsArray = $derived([...tags]);
	const settingsArray = $derived([...userSettings.entries()]);
</script>

<div class="min-h-screen bg-gradient-to-br from-indigo-50 to-purple-100 p-8">
	<div class="max-w-7xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">Built-in Reactive Classes</h1>
		<p class="text-gray-600 mb-8">Set, Map, and Date work reactively in Svelte 5</p>

		<div class="grid grid-cols-3 gap-6">
			<!-- Set Demo -->
			<div class="bg-white rounded-lg shadow p-6">
				<div class="flex items-center justify-between mb-4">
					<h2 class="text-xl font-semibold text-gray-900">Set ({tagCount})</h2>
					{#if tags.size > 0}
						<button onclick={clearTags} class="text-sm text-red-500 hover:text-red-700">
							Clear All
						</button>
					{/if}
				</div>

				<p class="text-sm text-gray-600 mb-4">
					A Set stores unique values - duplicates are automatically ignored
				</p>

				<!-- Add Tag Form -->
				<form
					onsubmit={(e) => {
						e.preventDefault();
						addTag();
					}}
					class="mb-4"
				>
					<div class="flex gap-2">
						<input
							type="text"
							bind:value={newTag}
							placeholder="Add tag..."
							class="flex-1 px-3 py-2 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-indigo-500"
						/>
						<button
							type="submit"
							class="px-4 py-2 bg-indigo-500 text-white rounded hover:bg-indigo-600 transition text-sm font-medium"
						>
							Add
						</button>
					</div>
				</form>

				<!-- Tags List -->
				{#if tagsArray.length === 0}
					<div class="text-center py-8 text-gray-500 text-sm">No tags yet</div>
				{:else}
					<div class="flex flex-wrap gap-2">
						{#each tagsArray as tag (tag)}
							<div
								class="flex items-center gap-2 px-3 py-1 bg-indigo-100 text-indigo-700 rounded-full text-sm"
							>
								<span>{tag}</span>
								<button
									onclick={() => removeTag(tag)}
									class="text-indigo-500 hover:text-indigo-700"
								>
									×
								</button>
							</div>
						{/each}
					</div>
				{/if}

				<!-- Set Methods Reference -->
				<div class="mt-6 p-3 bg-indigo-50 rounded text-xs font-mono space-y-1">
					<div class="text-indigo-900 font-semibold mb-2">Set Methods:</div>
					<div class="text-indigo-700">.add(value)</div>
					<div class="text-indigo-700">.delete(value)</div>
					<div class="text-indigo-700">.has(value)</div>
					<div class="text-indigo-700">.clear()</div>
					<div class="text-indigo-700">.size</div>
				</div>
			</div>

			<!-- Map Demo -->
			<div class="bg-white rounded-lg shadow p-6">
				<div class="flex items-center justify-between mb-4">
					<h2 class="text-xl font-semibold text-gray-900">Map ({settingsCount})</h2>
					{#if userSettings.size > 0}
						<button onclick={clearSettings} class="text-sm text-red-500 hover:text-red-700">
							Clear All
						</button>
					{/if}
				</div>

				<p class="text-sm text-gray-600 mb-4">A Map stores key-value pairs with any type of key</p>

				<!-- Add Setting Form -->
				<form
					onsubmit={(e) => {
						e.preventDefault();
						addSetting();
					}}
					class="mb-4 space-y-2"
				>
					<input
						type="text"
						bind:value={newKey}
						placeholder="Key..."
						class="w-full px-3 py-2 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-purple-500"
					/>
					<div class="flex gap-2">
						<input
							type="text"
							bind:value={newValue}
							placeholder="Value..."
							class="flex-1 px-3 py-2 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-purple-500"
						/>
						<button
							type="submit"
							class="px-4 py-2 bg-purple-500 text-white rounded hover:bg-purple-600 transition text-sm font-medium"
						>
							Add
						</button>
					</div>
				</form>

				<!-- Settings List -->
				{#if settingsArray.length === 0}
					<div class="text-center py-8 text-gray-500 text-sm">No settings yet</div>
				{:else}
					<div class="space-y-2">
						{#each settingsArray as [key, value] (key)}
							<div class="flex items-center justify-between p-2 bg-gray-50 rounded">
								<div class="flex-1">
									<div class="text-sm font-medium text-gray-900">{key}</div>
									<div class="text-xs text-gray-600">
										{typeof value === 'boolean' ? (value ? '✓ true' : '✗ false') : value}
										<span class="text-gray-400 ml-2">({typeof value})</span>
									</div>
								</div>
								<button
									onclick={() => removeSetting(key)}
									class="text-red-500 hover:text-red-700 text-sm"
								>
									Delete
								</button>
							</div>
						{/each}
					</div>
				{/if}

				<!-- Map Methods Reference -->
				<div class="mt-6 p-3 bg-purple-50 rounded text-xs font-mono space-y-1">
					<div class="text-purple-900 font-semibold mb-2">Map Methods:</div>
					<div class="text-purple-700">.set(key, value)</div>
					<div class="text-purple-700">.get(key)</div>
					<div class="text-purple-700">.delete(key)</div>
					<div class="text-purple-700">.has(key)</div>
					<div class="text-purple-700">.clear()</div>
					<div class="text-purple-700">.size</div>
				</div>
			</div>

			<!-- Date Demo -->
			<div class="bg-white rounded-lg shadow p-6">
				<div class="flex items-center justify-between mb-4">
					<h2 class="text-xl font-semibold text-gray-900">Events ({eventCount})</h2>
				</div>

				<p class="text-sm text-gray-600 mb-4">
					Date objects track timestamps and update reactively
				</p>

				<!-- Add Event Form -->
				<form
					onsubmit={(e) => {
						e.preventDefault();
						addEvent();
					}}
					class="mb-4"
				>
					<div class="flex gap-2">
						<input
							type="text"
							bind:value={eventName}
							placeholder="Event name..."
							class="flex-1 px-3 py-2 border border-gray-300 rounded text-sm focus:outline-none focus:ring-2 focus:ring-pink-500"
						/>
						<button
							type="submit"
							class="px-4 py-2 bg-pink-500 text-white rounded hover:bg-pink-600 transition text-sm font-medium"
						>
							Add
						</button>
					</div>
				</form>

				<!-- Events List -->
				{#if events.length === 0}
					<div class="text-center py-8 text-gray-500 text-sm">No events yet</div>
				{:else}
					<div class="space-y-2 max-h-64 overflow-y-auto">
						{#each events as event (event.id)}
							<div class="border border-gray-200 rounded p-3">
								<div class="flex items-start justify-between">
									<div class="flex-1">
										<div class="text-sm font-medium text-gray-900">{event.name}</div>
										<div class="text-xs text-gray-600 mt-1">
											{event.timestamp.toLocaleString()}
										</div>
										<div class="text-xs text-pink-600 mt-1">
											{getTimeAgo(event.timestamp)}
										</div>
									</div>
									<button
										onclick={() => removeEvent(event.id)}
										class="text-red-500 hover:text-red-700 text-sm ml-2"
									>
										×
									</button>
								</div>
							</div>
						{/each}
					</div>
				{/if}

				<!-- Date Methods Reference -->
				<div class="mt-6 p-3 bg-pink-50 rounded text-xs font-mono space-y-1">
					<div class="text-pink-900 font-semibold mb-2">Date Methods:</div>
					<div class="text-pink-700">new Date()</div>
					<div class="text-pink-700">.getTime()</div>
					<div class="text-pink-700">.toLocaleString()</div>
					<div class="text-pink-700">.toISOString()</div>
					<div class="text-pink-700">Date.now()</div>
				</div>
			</div>
		</div>

		<!-- Important Note -->
		<div class="mt-6 bg-yellow-50 border border-yellow-200 rounded-lg p-6">
			<h3 class="text-lg font-semibold text-yellow-900 mb-2">⚠️ Important: Reassignment</h3>
			<p class="text-sm text-yellow-800 mb-3">
				Built-in classes like Set and Map require reassignment to trigger reactivity:
			</p>
			<div class="space-y-2 text-sm font-mono bg-white p-4 rounded">
				<div class="text-gray-700">
					<span class="text-green-600">// ✓ Correct - triggers reactivity</span>
				</div>
				<div class="text-gray-700">mySet.add('value');</div>
				<div class="text-gray-700">mySet = mySet;</div>
				<div class="mt-3 text-gray-700">
					<span class="text-red-600">// ✗ Won't update UI</span>
				</div>
				<div class="text-gray-700">mySet.add('value'); // Missing reassignment</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Reactive Set**: Use `.add()`, `.delete()`, `.has()`, `.clear()`
- **Reactive Map**: Use `.set()`, `.get()`, `.delete()`, `.has()`
- **Reactive Date**: Date objects update automatically
- **Reassignment**: Must reassign Set/Map after mutations: `set = set`
- **Unique values**: Set automatically prevents duplicates
- **Type flexibility**: Map keys can be any type

---

## 8. API Integration Patterns

### How to Structure API Calls?

Best practices for organizing API calls, error handling, loading states, and caching in Svelte 5 applications.

**Real-World Scenario:** You're building a dashboard that fetches data from multiple APIs with proper error handling and loading states.

**What it does:** Shows a reusable pattern for API integration with TypeScript.

### 📁 Files to Create

**API Service Layer:**

- `src/lib/services/api.ts` - Generic API service
- `src/routes/section-4/api-integration/+page.svelte` - API integration demo

**API Service:**

```typescript
// src/lib/services/api.ts

export interface ApiResponse<T> {
  data?: T;
  error?: string;
  loading: boolean;
}

export interface User {
  id: number;
  name: string;
  email: string;
  username: string;
}

export interface Post {
  id: number;
  title: string;
  body: string;
  userId: number;
}

// Best Practice: Separate service layer from components
export class ApiService {
  private baseUrl: string;
  private cache: Map<string, { data: unknown; timestamp: number }> = new Map();
  private cacheExpiry = 60000; // 1 minute

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  private getCacheKey(endpoint: string): string {
    return `${this.baseUrl}${endpoint}`;
  }

  private isCacheValid(timestamp: number): boolean {
    return Date.now() - timestamp < this.cacheExpiry;
  }

  async fetch<T>(endpoint: string, useCache = true): Promise<T> {
    const cacheKey = this.getCacheKey(endpoint);

    // Check cache first - avoid unnecessary network requests
    if (useCache) {
      const cached = this.cache.get(cacheKey);
      // If cached data exists and hasn't expired, return it immediately
      if (cached && this.isCacheValid(cached.timestamp)) {
        return cached.data as T;
      }
    }

    // Fetch from API if cache miss or expired
    const response = await fetch(`${this.baseUrl}${endpoint}`);

    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }

    const data = await response.json();

    // Update cache with fresh data and current timestamp
    if (useCache) {
      this.cache.set(cacheKey, {
        data,
        timestamp: Date.now(),
      });
    }

    return data as T;
  }

  async getUsers(): Promise<User[]> {
    return this.fetch<User[]>("/users");
  }

  async getUser(id: number): Promise<User> {
    return this.fetch<User>(`/users/${id}`);
  }

  async getPosts(userId?: number): Promise<Post[]> {
    const endpoint = userId ? `/posts?userId=${userId}` : "/posts";
    return this.fetch<Post[]>(endpoint);
  }

  clearCache() {
    this.cache.clear();
  }
}

// Singleton instance
export const api = new ApiService("https://jsonplaceholder.typicode.com");
```

**Demo Page:**

```svelte
<script lang="ts">
	import { api, type User, type Post, type ApiResponse } from '$lib/services/api';

	let selectedUserId = $state<number | null>(null);

	// Users state
	let usersResponse = $state<ApiResponse<User[]>>({
		loading: true
	});

	// Posts state
	let postsResponse = $state<ApiResponse<Post[]>>({
		loading: false
	});

	// Load users on mount
	$effect(() => {
		loadUsers();
	});

	async function loadUsers() {
		usersResponse = { loading: true };

		try {
			const data = await api.getUsers();
			usersResponse = { data, loading: false };
		} catch (error) {
			usersResponse = {
				error: error instanceof Error ? error.message : 'Failed to load users',
				loading: false
			};
		}
	}

	async function loadPosts(userId: number) {
		selectedUserId = userId;
		postsResponse = { loading: true };

		try {
			const data = await api.getPosts(userId);
			postsResponse = { data, loading: false };
		} catch (error) {
			postsResponse = {
				error: error instanceof Error ? error.message : 'Failed to load posts',
				loading: false
			};
		}
	}

	function clearCache() {
		api.clearCache();
		loadUsers();
		if (selectedUserId) {
			loadPosts(selectedUserId);
		}
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-blue-50 to-cyan-100 p-8">
	<div class="max-w-7xl mx-auto">
		<div class="flex items-center justify-between mb-8">
			<div>
				<h1 class="text-4xl font-bold text-gray-900 mb-2">API Integration Patterns</h1>
				<p class="text-gray-600">Fetch data with loading states, errors, and caching</p>
			</div>
			<button
				onclick={clearCache}
				class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600 transition font-medium"
			>
				Clear Cache & Reload
			</button>
		</div>

		<div class="grid grid-cols-2 gap-6">
			<!-- Users List -->
			<div class="bg-white rounded-lg shadow p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">Users</h2>

				{#if usersResponse.loading}
					<div class="flex items-center justify-center py-12">
						<div
							class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"
						></div>
					</div>
				{:else if usersResponse.error}
					<div class="bg-red-50 border border-red-200 rounded-lg p-4 text-red-700">
						<div class="font-semibold mb-1">Error loading users</div>
						<div class="text-sm">{usersResponse.error}</div>
						<button
							onclick={loadUsers}
							class="mt-3 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 transition text-sm"
						>
							Retry
						</button>
					</div>
				{:else if usersResponse.data}
					<div class="space-y-2 max-h-96 overflow-y-auto">
						{#each usersResponse.data as user (user.id)}
							<button
								onclick={() => loadPosts(user.id)}
								class={`w-full text-left p-4 rounded-lg border-2 transition ${
									selectedUserId === user.id
										? 'border-blue-500 bg-blue-50'
										: 'border-gray-200 hover:border-blue-300 bg-white'
								}`}
							>
								<div class="font-semibold text-gray-900">{user.name}</div>
								<div class="text-sm text-gray-600">@{user.username}</div>
								<div class="text-xs text-gray-500">{user.email}</div>
							</button>
						{/each}
					</div>
				{/if}
			</div>

			<!-- Posts List -->
			<div class="bg-white rounded-lg shadow p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">Posts</h2>

				{#if !selectedUserId}
					<div class="text-center py-12 text-gray-500">
						<div class="text-4xl mb-2">📝</div>
						<div>Select a user to view their posts</div>
					</div>
				{:else if postsResponse.loading}
					<div class="flex items-center justify-center py-12">
						<div
							class="animate-spin rounded-full h-12 w-12 border-4 border-blue-500 border-t-transparent"
						></div>
					</div>
				{:else if postsResponse.error}
					<div class="bg-red-50 border border-red-200 rounded-lg p-4 text-red-700">
						<div class="font-semibold mb-1">Error loading posts</div>
						<div class="text-sm">{postsResponse.error}</div>
						<button
							onclick={() => selectedUserId && loadPosts(selectedUserId)}
							class="mt-3 px-4 py-2 bg-red-500 text-white rounded hover:bg-red-600 transition text-sm"
						>
							Retry
						</button>
					</div>
				{:else if postsResponse.data}
					<div class="space-y-3 max-h-96 overflow-y-auto">
						{#each postsResponse.data as post (post.id)}
							<div class="border border-gray-200 rounded-lg p-4">
								<h3 class="font-semibold text-gray-900 mb-2">{post.title}</h3>
								<p class="text-sm text-gray-600">{post.body}</p>
							</div>
						{/each}
					</div>
				{/if}
			</div>
		</div>

		<!-- Pattern Reference -->
		<div class="mt-6 bg-white rounded-lg shadow p-6">
			<h3 class="text-lg font-semibold text-gray-900 mb-4">API Integration Pattern</h3>
			<div class="grid grid-cols-2 gap-4 text-sm">
				<div class="border-l-4 border-blue-500 pl-4 bg-blue-50 p-3 rounded">
					<div class="font-semibold text-blue-900 mb-2">1. Loading State</div>
					<div class="text-blue-700 font-mono text-xs">{'{ loading: true }'}</div>
					<div class="text-blue-600 mt-2">Show spinner while fetching</div>
				</div>
				<div class="border-l-4 border-green-500 pl-4 bg-green-50 p-3 rounded">
					<div class="font-semibold text-green-900 mb-2">2. Success State</div>
					<div class="text-green-700 font-mono text-xs">{'{ data, loading: false }'}</div>
					<div class="text-green-600 mt-2">Display the fetched data</div>
				</div>
				<div class="border-l-4 border-red-500 pl-4 bg-red-50 p-3 rounded">
					<div class="font-semibold text-red-900 mb-2">3. Error State</div>
					<div class="text-red-700 font-mono text-xs">{'{ error, loading: false }'}</div>
					<div class="text-red-600 mt-2">Show error message with retry</div>
				</div>
				<div class="border-l-4 border-purple-500 pl-4 bg-purple-50 p-3 rounded">
					<div class="font-semibold text-purple-900 mb-2">4. Caching</div>
					<div class="text-purple-700 font-mono text-xs">Map + timestamp</div>
					<div class="text-purple-600 mt-2">Avoid redundant API calls</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **ApiResponse type**: Unified interface for loading, data, and error states
- **Service class**: Encapsulate all API logic in one place
- **Caching**: Store responses with timestamps to reduce API calls
- **Error handling**: Try-catch blocks with user-friendly messages
- **TypeScript**: Full type safety for API responses
- **Reusable**: Service can be used across all components

---

## 10. 🚀 End-of-Section Project: E-Commerce Shopping Cart

### Project Overview

Build a **complete e-commerce shopping cart** that demonstrates all advanced state patterns. This real-world project combines reactive classes, derived state, API integration, and writable derived patterns.

**What You'll Build:**

- A **ProductService** class with caching for API calls
- A **CartManager** reactive class for cart state
- **Writable derived** for quantity controls
- **{#await}** blocks for async product loading
- A complete shopping experience

### 📁 Files to Create

**1. Product Service** - `src/lib/services/products.svelte.ts`

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  image: string;
  category: string;
  inStock: boolean;
}

interface ApiResponse<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

class ProductService {
  private cache = new Map<string, { data: Product[]; timestamp: number }>();
  private CACHE_DURATION = 5 * 60 * 1000; // 5 minutes

  products = $state<ApiResponse<Product[]>>({
    data: null,
    loading: false,
    error: null,
  });

  async fetchProducts(category?: string): Promise<void> {
    const cacheKey = category || "all";
    const cached = this.cache.get(cacheKey);

    if (cached && Date.now() - cached.timestamp < this.CACHE_DURATION) {
      this.products = { data: cached.data, loading: false, error: null };
      return;
    }

    this.products = { data: null, loading: true, error: null };

    try {
      // Simulated API call
      await new Promise((r) => setTimeout(r, 800));

      const mockProducts: Product[] = [
        {
          id: 1,
          name: "Wireless Headphones",
          price: 99.99,
          image: "🎧",
          category: "electronics",
          inStock: true,
        },
        {
          id: 2,
          name: "Running Shoes",
          price: 129.99,
          image: "👟",
          category: "sports",
          inStock: true,
        },
        {
          id: 3,
          name: "Coffee Maker",
          price: 79.99,
          image: "☕",
          category: "home",
          inStock: false,
        },
        {
          id: 4,
          name: "Backpack",
          price: 59.99,
          image: "🎒",
          category: "accessories",
          inStock: true,
        },
        {
          id: 5,
          name: "Smart Watch",
          price: 249.99,
          image: "⌚",
          category: "electronics",
          inStock: true,
        },
        {
          id: 6,
          name: "Yoga Mat",
          price: 29.99,
          image: "🧘",
          category: "sports",
          inStock: true,
        },
      ];

      const filtered = category
        ? mockProducts.filter((p) => p.category === category)
        : mockProducts;

      this.cache.set(cacheKey, { data: filtered, timestamp: Date.now() });
      this.products = { data: filtered, loading: false, error: null };
    } catch (e) {
      this.products = {
        data: null,
        loading: false,
        error: "Failed to load products",
      };
    }
  }

  clearCache() {
    this.cache.clear();
  }
}

export const productService = new ProductService();
```

**2. Cart Manager** - `src/lib/stores/cart.svelte.ts`

```typescript
interface CartItem {
  productId: number;
  name: string;
  price: number;
  image: string;
  quantity: number;
}

class CartManager {
  items = $state<CartItem[]>([]);

  // Derived values
  totalItems = $derived(
    this.items.reduce((sum, item) => sum + item.quantity, 0)
  );

  subtotal = $derived(
    this.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  );

  tax = $derived(this.subtotal * 0.08); // 8% tax

  total = $derived(this.subtotal + this.tax);

  // Writable derived for individual item quantities
  getQuantityControl(productId: number) {
    return {
      get value() {
        const item = cartManager.items.find((i) => i.productId === productId);
        return item?.quantity || 0;
      },
      set value(newQuantity: number) {
        if (newQuantity <= 0) {
          cartManager.removeItem(productId);
        } else {
          const item = cartManager.items.find((i) => i.productId === productId);
          if (item) {
            item.quantity = newQuantity;
          }
        }
      },
    };
  }

  addItem(product: { id: number; name: string; price: number; image: string }) {
    const existing = this.items.find((i) => i.productId === product.id);
    if (existing) {
      existing.quantity++;
    } else {
      this.items.push({
        productId: product.id,
        name: product.name,
        price: product.price,
        image: product.image,
        quantity: 1,
      });
    }
  }

  removeItem(productId: number) {
    this.items = this.items.filter((i) => i.productId !== productId);
  }

  updateQuantity(productId: number, quantity: number) {
    const item = this.items.find((i) => i.productId === productId);
    if (item) {
      if (quantity <= 0) {
        this.removeItem(productId);
      } else {
        item.quantity = quantity;
      }
    }
  }

  clearCart() {
    this.items = [];
  }

  isInCart(productId: number): boolean {
    return this.items.some((i) => i.productId === productId);
  }
}

export const cartManager = new CartManager();
```

**3. Shopping Page** - `src/routes/shop/+page.svelte`

```svelte
<script lang="ts">
	import { onMount } from 'svelte';
	import { productService } from '$lib/services/products.svelte';
	import { cartManager } from '$lib/stores/cart.svelte';

	let selectedCategory = $state<string | undefined>(undefined);
	let showCart = $state(false);

	const categories = ['all', 'electronics', 'sports', 'home', 'accessories'];

	onMount(() => {
		productService.fetchProducts();
	});

	function selectCategory(cat: string) {
		selectedCategory = cat === 'all' ? undefined : cat;
		productService.fetchProducts(selectedCategory);
	}
</script>

<div class="bg-gray-900 min-h-screen text-gray-200">
	<!-- Header -->
	<header class="bg-gray-800 border-b border-gray-700 p-4 sticky top-0 z-40">
		<div class="max-w-6xl mx-auto flex justify-between items-center">
			<h1 class="text-2xl font-bold text-blue-400">🛍️ Svelte Shop</h1>
			<button
				onclick={() => (showCart = !showCart)}
				class="relative bg-blue-500 hover:bg-blue-400 text-white px-4 py-2 rounded-lg font-semibold transition-colors"
			>
				Cart
				{#if cartManager.totalItems > 0}
					<span class="absolute -top-2 -right-2 bg-red-500 text-white text-xs w-6 h-6 rounded-full flex items-center justify-center">
						{cartManager.totalItems}
					</span>
				{/if}
			</button>
		</div>
	</header>

	<div class="max-w-6xl mx-auto p-6">
		<!-- Category Filter -->
		<div class="flex gap-2 mb-8 flex-wrap">
			{#each categories as cat}
				<button
					onclick={() => selectCategory(cat)}
					class="px-4 py-2 rounded-lg font-semibold transition-all capitalize"
					class:bg-blue-500={selectedCategory === (cat === 'all' ? undefined : cat) || (cat === 'all' && !selectedCategory)}
					class:text-white={selectedCategory === (cat === 'all' ? undefined : cat) || (cat === 'all' && !selectedCategory)}
					class:bg-gray-700={!(selectedCategory === (cat === 'all' ? undefined : cat) || (cat === 'all' && !selectedCategory))}
					class:text-gray-300={!(selectedCategory === (cat === 'all' ? undefined : cat) || (cat === 'all' && !selectedCategory))}
				>
					{cat}
				</button>
			{/each}
		</div>

		<!-- Products Grid -->
		{#if productService.products.loading}
			<div class="flex justify-center py-20">
				<div class="w-12 h-12 border-4 border-blue-500 border-t-transparent rounded-full animate-spin"></div>
			</div>
		{:else if productService.products.error}
			<div class="bg-red-500/10 border border-red-500 rounded-lg p-6 text-center">
				<p class="text-red-400">{productService.products.error}</p>
				<button
					onclick={() => productService.fetchProducts(selectedCategory)}
					class="mt-4 bg-red-500 hover:bg-red-400 text-white px-4 py-2 rounded-lg"
				>
					Retry
				</button>
			</div>
		{:else if productService.products.data}
			<div class="grid md:grid-cols-2 lg:grid-cols-3 gap-6">
				{#each productService.products.data as product (product.id)}
					<div class="bg-gray-800 rounded-xl p-6 border border-gray-700 hover:border-blue-500 transition-colors">
						<div class="text-6xl mb-4 text-center">{product.image}</div>
						<h3 class="text-lg font-bold text-white mb-2">{product.name}</h3>
						<p class="text-2xl font-bold text-blue-400 mb-4">${product.price.toFixed(2)}</p>

						{#if !product.inStock}
							<span class="text-red-400 font-semibold">Out of Stock</span>
						{:else if cartManager.isInCart(product.id)}
							<div class="flex items-center gap-3">
								<button
									onclick={() => cartManager.updateQuantity(product.id, cartManager.getQuantityControl(product.id).value - 1)}
									class="w-10 h-10 bg-gray-700 hover:bg-gray-600 rounded-lg text-xl"
								>
									-
								</button>
								<span class="text-lg font-bold w-8 text-center">
									{cartManager.getQuantityControl(product.id).value}
								</span>
								<button
									onclick={() => cartManager.updateQuantity(product.id, cartManager.getQuantityControl(product.id).value + 1)}
									class="w-10 h-10 bg-gray-700 hover:bg-gray-600 rounded-lg text-xl"
								>
									+
								</button>
							</div>
						{:else}
							<button
								onclick={() => cartManager.addItem(product)}
								class="w-full bg-blue-500 hover:bg-blue-400 text-white py-3 rounded-lg font-semibold transition-colors"
							>
								Add to Cart
							</button>
						{/if}
					</div>
				{/each}
			</div>
		{/if}
	</div>

	<!-- Cart Sidebar -->
	{#if showCart}
		<div class="fixed inset-0 bg-black/50 z-50" onclick={() => (showCart = false)}>
			<div
				class="absolute right-0 top-0 h-full w-full max-w-md bg-gray-800 p-6 overflow-y-auto"
				onclick={(e) => e.stopPropagation()}
			>
				<div class="flex justify-between items-center mb-6">
					<h2 class="text-2xl font-bold text-white">Your Cart</h2>
					<button onclick={() => (showCart = false)} class="text-gray-400 hover:text-white text-2xl">✕</button>
				</div>

				{#if cartManager.items.length === 0}
					<p class="text-gray-400 text-center py-10">Your cart is empty</p>
				{:else}
					<div class="flex flex-col gap-4 mb-6">
						{#each cartManager.items as item (item.productId)}
							<div class="bg-gray-700 rounded-lg p-4 flex gap-4">
								<div class="text-4xl">{item.image}</div>
								<div class="flex-1">
									<h4 class="font-semibold text-white">{item.name}</h4>
									<p class="text-blue-400">${item.price.toFixed(2)}</p>
									<div class="flex items-center gap-2 mt-2">
										<button
											onclick={() => cartManager.updateQuantity(item.productId, item.quantity - 1)}
											class="w-8 h-8 bg-gray-600 hover:bg-gray-500 rounded text-sm"
										>
											-
										</button>
										<span class="w-8 text-center">{item.quantity}</span>
										<button
											onclick={() => cartManager.updateQuantity(item.productId, item.quantity + 1)}
											class="w-8 h-8 bg-gray-600 hover:bg-gray-500 rounded text-sm"
										>
											+
										</button>
									</div>
								</div>
								<button
									onclick={() => cartManager.removeItem(item.productId)}
									class="text-red-400 hover:text-red-300"
								>
									🗑️
								</button>
							</div>
						{/each}
					</div>

					<div class="border-t border-gray-700 pt-4 space-y-2">
						<div class="flex justify-between text-gray-400">
							<span>Subtotal</span>
							<span>${cartManager.subtotal.toFixed(2)}</span>
						</div>
						<div class="flex justify-between text-gray-400">
							<span>Tax (8%)</span>
							<span>${cartManager.tax.toFixed(2)}</span>
						</div>
						<div class="flex justify-between text-xl font-bold text-white pt-2 border-t border-gray-700">
							<span>Total</span>
							<span>${cartManager.total.toFixed(2)}</span>
						</div>
					</div>

					<button class="w-full bg-green-500 hover:bg-green-400 text-white py-4 rounded-lg font-bold mt-6 transition-colors">
						Checkout (${cartManager.total.toFixed(2)})
					</button>
				{/if}
			</div>
		</div>
	{/if}
</div>
```

### What This Project Demonstrates

| Concept                 | Implementation                           |
| ----------------------- | ---------------------------------------- |
| **Reactive Classes**    | CartManager, ProductService              |
| **$derived**            | totalItems, subtotal, tax, total         |
| **Writable Derived**    | getQuantityControl() for quantity inputs |
| **API Caching**         | ProductService with timestamp cache      |
| **Loading States**      | products.loading, products.error         |
| **Class Methods**       | addItem, removeItem, updateQuantity      |
| **Built-in Reactivity** | Map for caching, array mutations         |

---

## 📝 Key Takeaways

✅ API integration with caching
✅ Writable derived state with getters/setters
✅ {#await} blocks for async data
✅ Reactive classes encapsulate logic
✅ Class methods trigger UI updates
✅ Built-in reactive classes (Set, Map, Date)

> 💡 **Best Practice**: Separate API logic into service layer (`src/lib/services/`). Keep components focused on UI, services focused on data.

**⚠️ Common Mistakes:**

- Don't put fetch calls directly in components - use service layer
- Don't forget error handling - always show error states to users
- Don't skip loading states - they improve perceived performance
- Don't forget to handle race conditions with AbortController

**⚡ Performance Tips:**

- Cache API responses to avoid redundant requests
- Use `{#await}` for automatic loading/error states
- Debounce search inputs to reduce API calls
- Consider SWR (stale-while-revalidate) pattern for better UX

**🏗️ Architecture:**

```
src/lib/
├── services/
│   ├── api.ts (base service)
│   ├── users.ts (user-specific endpoints)
│   └── posts.ts (post-specific endpoints)
└── stores/
    └── data.svelte.ts (reactive data stores)
```

---

## 🚀 Next Steps

After mastering advanced state patterns, you'll be ready for:

- **Section 5**: Universal reactivity and shared state
- Building production applications
- Managing global state effectively
