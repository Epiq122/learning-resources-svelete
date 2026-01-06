# Svelte 5 Patterns & Best Practices

Quick reference guide for common Svelte 5 patterns used in real projects.

---

## Table of Contents

1. [Reactive State](#reactive-state)
2. [Computed Values](#computed-values)
3. [Side Effects](#side-effects)
4. [Fetching Data](#fetching-data)
5. [Template Logic](#template-logic)
6. [Binding Patterns](#binding-patterns)
7. [Reactive Classes](#reactive-classes)

---

## Reactive State

### `$state()` - Basic Reactive Variables

**Use when:** You need a variable that updates the UI when it changes

```ts
let count = $state(0); // Simple value
let items = $state([]); // Array
let user = $state({ name: 'John' }); // Object

// Changing them updates the UI automatically
count = 5;
items.push('new item');
user.name = 'Jane';
```

### In Classes

```ts
class Counter {
	count = $state(0); // Each instance has its own reactive count

	increment() {
		this.count++; // UI updates automatically
	}
}

const counter1 = new Counter();
const counter2 = new Counter(); // Independent from counter1
```

---

## Computed Values

### `$derived()` - Auto-Calculated Values

**Use when:** A value depends on other reactive state and should update automatically

```ts
let price = $state(100);
let quantity = $state(2);

// Recalculates automatically when price or quantity changes
let total = $derived(price * quantity);
```

### Real Example: Currency Converter

```ts
class CurrencyConverter {
	baseValue = $state(1);
	rate = $state(1.2);

	// Automatically updates when baseValue or rate changes
	convertedValue = $derived(this.baseValue * this.rate);
}
```

### Multiple Dependencies

```ts
class Cart {
	items = $state([
		{ name: 'Apple', price: 1.5, qty: 3 },
		{ name: 'Bread', price: 2.0, qty: 1 }
	]);
	taxRate = $state(0.08);

	// Recalculates when items or taxRate changes
	subtotal = $derived(this.items.reduce((sum, item) => sum + item.price * item.qty, 0));

	tax = $derived(this.subtotal * this.taxRate);
	total = $derived(this.subtotal + this.tax);
}
```

**Why it's better than manual calculation:**

- No need to remember to update `total` every time `items` or `taxRate` changes
- Can't forget to recalculate
- Cleaner code

---

## Side Effects

### `$effect()` - Run Code When State Changes

**Use when:** You need to do something in response to state changes (fetch data, update localStorage, etc.)

```ts
let theme = $state('dark');

$effect(() => {
	// Runs when theme changes
	document.body.className = theme;
	localStorage.setItem('theme', theme);
});
```

### With Cleanup

```ts
let isActive = $state(false);

$effect(() => {
	if (isActive) {
		const interval = setInterval(() => console.log('tick'), 1000);

		// Cleanup function - runs when effect re-runs or component unmounts
		return () => clearInterval(interval);
	}
});
```

### In Classes - Watch for Changes

```ts
class DataFetcher {
	userId = $state(1);
	userData = $state(null);

	constructor() {
		// Re-fetches whenever userId changes
		$effect(() => {
			this.fetchUser();
		});
	}

	async fetchUser() {
		const res = await fetch(`/api/users/${this.userId}`);
		this.userData = await res.json();
	}
}
```

---

## Fetching Data

### Basic Fetch Pattern

```ts
let data = $state(null);
let loading = $state(false);
let error = $state(null);

async function fetchData() {
	loading = true;
	error = null;

	try {
		const response = await fetch('https://api.example.com/data');

		if (!response.ok) {
			throw new Error('Failed to fetch');
		}

		data = await response.json();
	} catch (e) {
		error = e.message;
	} finally {
		loading = false;
	}
}

// Call it when component loads
$effect(() => {
	fetchData();
});
```

### Fetch in Class with Auto-Refetch

```ts
class CurrencyData {
	currency = $state('usd');
	rates = $state({});
	loading = $state(false);
	error = $state(undefined);

	constructor(initialCurrency) {
		this.currency = initialCurrency;

		// Automatically re-fetches when currency changes
		$effect(() => {
			this.#fetchRates();
		});
	}

	async #fetchRates() {
		this.loading = true;
		this.error = undefined;

		try {
			const res = await fetch(`https://api.example.com/rates/${this.currency}.json`);
			const data = await res.json();
			this.rates = data[this.currency];
		} catch (e) {
			this.error = 'Failed to load rates';
		}

		this.loading = false;
	}
}

// Usage
const converter = new CurrencyData('usd');
// Change currency - automatically fetches new rates
converter.currency = 'eur';
```

### Chaining Promises (Cleaner Fetch)

```ts
// Instead of await twice:
const res = await fetch(url);
const data = await res.json();

// You can chain:
const data = await fetch(url).then((r) => r.json());
```

---

## Template Logic

### `{#if}` - Conditional Rendering

```svelte
{#if loading}
	<p>Loading...</p>
{:else if error}
	<p>Error: {error}</p>
{:else}
	<p>Data loaded!</p>
{/if}
```

### `{#each}` - Loop Over Arrays

```svelte
<script lang="ts">
	let items = $state(['Apple', 'Banana', 'Orange']);
</script>

<ul>
	{#each items as item}
		<li>{item}</li>
	{/each}
</ul>
```

### `{#each}` with Index and Key

```svelte
<script lang="ts">
	let users = $state([
		{ id: 1, name: 'Alice' },
		{ id: 2, name: 'Bob' }
	]);
</script>

{#each users as user, index (user.id)}
	<div>
		{index + 1}. {user.name}
	</div>
{/each}
```

**Why use a key?** Helps Svelte track which item is which when the array changes.

### `{#each}` with Objects

```svelte
<script lang="ts">
	let currencies = $state({
		usd: 'US Dollar',
		eur: 'Euro',
		gbp: 'British Pound'
	});
</script>

<select>
	{#each Object.entries(currencies) as [code, name]}
		<option value={code}>{name}</option>
	{/each}
</select>
```

---

## Binding Patterns

### Simple Two-Way Binding

```svelte
<script lang="ts">
	let name = $state('');
</script>

<input bind:value={name} /><p>Hello, {name}!</p>
```

### Bind with Validation (Custom Getter/Setter)

```svelte
<script lang="ts">
	let age = $state(0);
</script>

<input
	type="number"
	bind:value={
		() => age, // Getter: what to show
		(value) => {
			// Setter: handle input
			if (value < 0) {
				age = 0; // Don't allow negative
			} else if (value > 120) {
				age = 120; // Cap at 120
			} else {
				age = value;
			}
		}
	}
/>
```

### Binding to Derived Values (Write to Source)

```svelte
<script lang="ts">
	class Converter {
		celsius = $state(0);

		// Derived from celsius
		fahrenheit = $derived((this.celsius * 9) / 5 + 32);
	}

	const temp = new Converter();
</script>

<!-- Read from derived -->
<input bind:value={temp.celsius} />

<!-- Write back to source when editing derived value -->
<input
	bind:value={
		() => temp.fahrenheit,
		(f) => {
			temp.celsius = ((f - 32) * 5) / 9; // Convert back
		}
	}
/>

<p>{temp.celsius}°C = {temp.fahrenheit.toFixed(1)}°F</p>
```

### Select Binding

```svelte
<script lang="ts">
	let selected = $state('option1');
</script>

<select bind:value={selected}>
	<option value="option1">First</option>
	<option value="option2">Second</option>
</select>
```

---

## Reactive Classes

### Full Example: Reusable Currency Converter

```ts
// currency-converter.svelte.ts
class CurrencyConverter {
	baseValue: number | undefined = $state(1);
	baseCurrency = $state('usd');
	baseRates: Record<string, number> = $state({});
	targetCurrency = $state('eur');
	currencies: Record<string, string> = $state({});
	loading = $state(true);
	error: string | undefined = $state();

	// Auto-calculated - updates when dependencies change
	targetValue = $derived(
		this.baseValue && this.baseRates[this.targetCurrency]
			? this.baseValue * this.baseRates[this.targetCurrency]
			: null
	);

	constructor(baseValue: number, baseCurrency: string, targetCurrency: string) {
		this.baseValue = baseValue;
		this.baseCurrency = baseCurrency;
		this.targetCurrency = targetCurrency;

		this.#loadCurrencies();

		// Watch baseCurrency - refetch rates when it changes
		$effect(() => {
			this.#fetchRates();
		});
	}

	async #fetchRates() {
		try {
			const res = await fetch(`https://api.example.com/currencies/${this.baseCurrency}.json`);
			const data = await res.json();
			this.baseRates = data[this.baseCurrency];
		} catch {
			this.error = 'Failed to fetch rates';
		}
	}

	async #loadCurrencies() {
		this.loading = true;
		this.error = undefined;

		try {
			this.currencies = await fetch('https://api.example.com/currencies.json').then((r) =>
				r.json()
			);
		} catch {
			this.error = 'Failed to load currencies';
		}

		this.loading = false;
	}
}

export default CurrencyConverter;
```

### Using the Class

```svelte
<script lang="ts">
	import CurrencyConverter from './currency-converter.svelte';

	const converter = new CurrencyConverter(1, 'usd', 'eur');
</script>

{#if converter.error}
	<p>Error: {converter.error}</p>
{:else if converter.loading}
	<p>Loading...</p>
{:else}
	<div>
		<input
			type="number"
			bind:value={
				() => converter.baseValue,
				(value) => {
					converter.baseValue = value;
				}
			}
		/>

		<select bind:value={converter.baseCurrency}>
			{#each Object.entries(converter.currencies) as [code, name]}
				<option value={code}>{name}</option>
			{/each}
		</select>

		<p>=</p>

		<input
			type="number"
			bind:value={
				() => converter.targetValue,
				(value) => {
					// Writing to derived value - update the source
					converter.baseValue = value / converter.baseRates[converter.targetCurrency];
				}
			}
		/>

		<select bind:value={converter.targetCurrency}>
			{#each Object.entries(converter.currencies) as [code, name]}
				<option value={code}>{name}</option>
			{/each}
		</select>
	</div>
{/if}
```

---

## Common Patterns Summary

### Fetch Data Once on Load

```ts
$effect(() => {
	fetch('/api/data')
		.then((r) => r.json())
		.then((data) => (myData = data));
});
```

### Fetch Data When Something Changes

```ts
let userId = $state(1);

$effect(() => {
	fetch(`/api/users/${userId}`)
		.then((r) => r.json())
		.then((data) => (userData = data));
});
```

### Update localStorage When State Changes

```ts
let settings = $state({ theme: 'dark' });

$effect(() => {
	localStorage.setItem('settings', JSON.stringify(settings));
});
```

### Validate Input

```svelte
<input
	bind:value={
		() => value,
		(v) => {
			value = v < 0 ? 0 : v;
		}
	}
/>
```

### Format Display Without Changing State

```svelte
<p>{price.toFixed(2)}</p><p>{date.toLocaleDateString()}</p>
```

---

## When to Use What

| Pattern          | Use Case                                |
| ---------------- | --------------------------------------- |
| `$state()`       | Store any data that changes             |
| `$derived()`     | Calculate values from other state       |
| `$effect()`      | Fetch data, subscriptions, side effects |
| `{#if}`          | Show/hide based on condition            |
| `{#each}`        | Display lists                           |
| `bind:value`     | Two-way form input binding              |
| Custom bind      | Validation or write to derived values   |
| Reactive classes | Reusable logic across components        |

---

## Don't Use Reactive Classes When

- ❌ Simple one-off component (just use `$state` directly)
- ❌ No shared logic between components
- ❌ Pure presentation (props are simpler)
- ❌ One-time calculations (use regular functions)

## DO Use Reactive Classes When

- ✅ Logic used in multiple components
- ✅ Complex state with multiple related pieces
- ✅ Multiple computed values
- ✅ Need independent instances with their own state

## When NOT to Use Reactive Classes

- **Simple components** - Just use regular `let` with `$state`
- **One-time calculations** - Use regular functions
- **Pure presentation** - Props and events are simpler
- **No shared logic** - Classes add unnecessary complexity

Use reactive classes when you need:

- Reusable stateful logic across multiple components
- Encapsulated business logic with multiple related pieces of state
- Complex computed values with multiple dependencies

---

## Universal Reactivity - Sharing State Across Components

Universal reactivity lets you create reactive state **outside of components** that can be shared across your entire app. This is perfect for global state like user auth, themes, shopping carts, or any data multiple components need.

---

### Basic Shared State with Functions

**Use when:** You need simple shared state accessible from multiple components

```typescript
// cart.svelte.ts
let items = $state([]);

export function addItem(item) {
	items.push(item);
}

export function removeItem(id) {
	items = items.filter((item) => item.id !== id);
}

export function getItems() {
	return items;
}

export function getTotal() {
	return items.reduce((sum, item) => sum + item.price, 0);
}
```

**Usage in Components:**

```svelte
<script lang="ts">
	import { addItem, getItems, getTotal } from '$lib/cart.svelte';

	let items = $derived(getItems());
	let total = $derived(getTotal());
</script>

<ul>
	{#each items as item}
		<li>{item.name} - ${item.price}</li>
	{/each}
</ul>

<p>Total: ${total}</p>
<button onclick={() => addItem({ id: 1, name: 'Apple', price: 1.5 })}>Add Apple</button>
```

---

### Shared State with Object Getters/Setters

**Use when:** You want property-like access (cleaner syntax than functions)

```typescript
// counter.svelte.ts
const state = $state({ value: 0 });

export default {
	// ✅ Getter - access like a property
	get value() {
		return state.value;
	},

	// ✅ Setter - assign like a property
	set value(newValue) {
		state.value = newValue;
	},

	// Methods for actions
	increment() {
		state.value++;
	},

	reset() {
		state.value = 0;
	}
};
```

**Usage:**

```svelte
<script lang="ts">
	import counter from '$lib/counter.svelte';

	// Reactive - automatically updates when counter changes
	let doubled = $derived(counter.value * 2);
</script>

<p>Count: {counter.value}</p>
<p>Doubled: {doubled}</p>

<!-- All these work -->
<button onclick={() => counter.increment()}>+1</button>
<button onclick={() => counter.value++}>+1 (direct)</button>
<button onclick={() => (counter.value = 10)}>Set to 10</button>
<button onclick={() => counter.reset()}>Reset</button>
```

**Why use getters/setters?**

- Cleaner syntax: `counter.value` vs `counter.getValue()`
- Works with `$derived` automatically
- Can assign values: `counter.value = 5`
- Feels like working with regular properties

---

### Shared State with Classes

**Use when:** You need multiple instances or complex logic

```typescript
// store.svelte.ts
class Store {
	items = $state([]);
	loading = $state(false);

	get total() {
		return this.items.reduce((sum, item) => sum + item.price * item.qty, 0);
	}

	get itemCount() {
		return this.items.reduce((sum, item) => sum + item.qty, 0);
	}

	addItem(item) {
		const existing = this.items.find((i) => i.id === item.id);
		if (existing) {
			existing.qty++;
		} else {
			this.items.push({ ...item, qty: 1 });
		}
	}

	removeItem(id) {
		this.items = this.items.filter((item) => item.id !== id);
	}

	clear() {
		this.items = [];
	}
}

// Export a single instance for the whole app
export const cart = new Store();
```

**Usage:**

```svelte
<script lang="ts">
	import { cart } from '$lib/store.svelte';
</script>

<p>Items: {cart.itemCount}</p>
<p>Total: ${cart.total.toFixed(2)}</p>

{#each cart.items as item}
	<div>
		{item.name} x{item.qty} = ${(item.price * item.qty).toFixed(2)}
		<button onclick={() => cart.removeItem(item.id)}>Remove</button>
	</div>
{/each}
```

---

### Shared State with Proxy

**Use when:** You want to intercept and customize property access/assignment

```typescript
// theme.svelte.ts
const state = $state({ theme: 'light' });

const themeProxy = new Proxy(state, {
	get(target, prop) {
		console.log(`Getting ${prop}: ${target[prop]}`);
		return target[prop];
	},

	set(target, prop, value) {
		console.log(`Setting ${prop} to ${value}`);

		// Custom validation
		if (prop === 'theme' && !['light', 'dark'].includes(value)) {
			console.error('Invalid theme. Use "light" or "dark"');
			return false;
		}

		target[prop] = value;
		return true;
	}
});

export default themeProxy;
```

**Usage:**

```svelte
<script lang="ts">
	import theme from '$lib/theme.svelte';
</script>

<p>Current theme: {theme.theme}</p>

<button onclick={() => (theme.theme = 'dark')}>Dark Mode</button>
<button onclick={() => (theme.theme = 'light')}>Light Mode</button>
<!-- This will log an error but not crash -->
<button onclick={() => (theme.theme = 'invalid')}>Invalid</button>
```

---

### Effects in Shared State with $effect.root()

**Use when:** You need effects (like localStorage sync) in module-level code

```typescript
// counter.svelte.ts
import { browser } from '$app/environment';

// Load from localStorage on startup
const storedCount = browser && localStorage.getItem('count');
const count = $state(storedCount ? JSON.parse(storedCount) : { value: 0 });

// ✅ $effect.root() lets you use $effect outside components
$effect.root(() => {
	$effect(() => {
		// Save to localStorage whenever count changes
		if (browser) {
			localStorage.setItem('count', JSON.stringify(count));
		}
	});
});

export function increment() {
	count.value++;
}

export function reset() {
	count.value = 0;
}

export default {
	get value() {
		return count.value;
	},
	increment,
	reset
};
```

**Why use $effect.root()?**

- Allows `$effect` in module scope (outside components)
- Effect runs immediately and persists
- Perfect for syncing state with localStorage, APIs, etc.

**Usage:**

```svelte
<script lang="ts">
	import counter from '$lib/counter.svelte';
	// Counter value automatically persists to localStorage!
</script>

<p>{counter.value}</p>
<button onclick={counter.increment}>+1</button>
<!-- Refresh the page - value is still there! -->
```

---

### Persisting State to LocalStorage

**Pattern 1: Simple Persistence with Functions**

```typescript
// settings.svelte.ts
import { browser } from '$app/environment';

const stored = browser && localStorage.getItem('settings');
const settings = $state(stored ? JSON.parse(stored) : { theme: 'light', fontSize: 16 });

// Save whenever settings change
$effect.root(() => {
	$effect(() => {
		if (browser) {
			localStorage.setItem('settings', JSON.stringify(settings));
		}
	});
});

export function updateTheme(theme) {
	settings.theme = theme;
}

export function updateFontSize(size) {
	settings.fontSize = size;
}

export function getSettings() {
	return settings;
}
```

**Pattern 2: Persistence with Classes**

```typescript
// settings-store.svelte.ts
import { browser } from '$app/environment';

class SettingsStore {
	#state = $state({ theme: 'light', fontSize: 16 });

	constructor() {
		// Load from localStorage
		if (browser) {
			const stored = localStorage.getItem('settings');
			if (stored) {
				this.#state = JSON.parse(stored);
			}
		}

		// Auto-save to localStorage
		$effect.root(() => {
			$effect(() => {
				if (browser) {
					localStorage.setItem('settings', JSON.stringify(this.#state));
				}
			});
		});
	}

	get theme() {
		return this.#state.theme;
	}

	set theme(value) {
		this.#state.theme = value;
	}

	get fontSize() {
		return this.#state.fontSize;
	}

	set fontSize(value) {
		this.#state.fontSize = value;
	}
}

export const settings = new SettingsStore();
```

**Usage:**

```svelte
<script lang="ts">
	import { settings } from '$lib/settings-store.svelte';
</script>

<div style="font-size: {settings.fontSize}px" class={settings.theme}>
	<label>
		Theme:
		<select bind:value={settings.theme}>
			<option value="light">Light</option>
			<option value="dark">Dark</option>
		</select>
	</label>

	<label>
		Font Size: {settings.fontSize}px
		<input type="range" min="12" max="24" bind:value={settings.fontSize} />
	</label>
</div>
<!-- Changes automatically save to localStorage! -->
```

---

### $effect.tracking()

**Use when:** You need to check if code is running inside an effect

```typescript
function logMessage(msg) {
	if ($effect.tracking()) {
		console.log('Called from inside an effect:', msg);
	} else {
		console.log('Called directly:', msg);
	}
}

$effect(() => {
	logMessage('Hello'); // "Called from inside an effect: Hello"
});

logMessage('World'); // "Called directly: World"
```

**Real Use Case - Conditional Subscriptions:**

```typescript
// scroll-tracker.svelte.ts
let scrollY = $state(0);

function subscribeToScroll() {
	// Only set up listener if someone is tracking this value
	if ($effect.tracking()) {
		const handler = () => {
			scrollY = window.scrollY;
		};

		window.addEventListener('scroll', handler);

		return () => {
			window.removeEventListener('scroll', handler);
		};
	}
}

export function getScrollY() {
	subscribeToScroll();
	return scrollY;
}
```

---

### createSubscriber Pattern

**Use when:** You want to lazily subscribe to external events only when needed

```typescript
// scroll-position.svelte.ts
import { createSubscriber } from 'svelte/reactivity';

const [getScrollY, subscribe] = createSubscriber(0);

// This function is called ONLY when something reads scrollY in an effect
subscribe((set) => {
	function handleScroll() {
		set(window.scrollY);
	}

	window.addEventListener('scroll', handleScroll);

	// Cleanup when no longer needed
	return () => window.removeEventListener('scroll', handleScroll);
});

export { getScrollY };
```

**Usage:**

```svelte
<script lang="ts">
	import { getScrollY } from '$lib/scroll-position.svelte';

	// Only subscribes to scroll events when this component is mounted
	let scrollY = $derived(getScrollY());
	let scrollPercent = $derived((scrollY / document.body.scrollHeight) * 100);
</script>

<div class="scroll-indicator" style="width: {scrollPercent}%"></div><p>Scrolled: {scrollY}px</p>
```

**Why use createSubscriber?**

- Lazy - only subscribes when someone reads the value
- Automatic cleanup - unsubscribes when no one is using it
- Efficient - one subscription shared by all consumers
- Perfect for expensive operations (scroll, resize, mouse position)

---

### createSubscriber in a Reusable Class

**Use when:** You want encapsulated, reusable subscriptions

```typescript
// mouse-tracker.svelte.ts
import { createSubscriber } from 'svelte/reactivity';

class MouseTracker {
	#getX;
	#getY;

	constructor() {
		// Track X position
		const [getX, subscribeX] = createSubscriber(0);
		this.#getX = getX;

		subscribeX((set) => {
			const handler = (e) => set(e.clientX);
			window.addEventListener('mousemove', handler);
			return () => window.removeEventListener('mousemove', handler);
		});

		// Track Y position
		const [getY, subscribeY] = createSubscriber(0);
		this.#getY = getY;

		subscribeY((set) => {
			const handler = (e) => set(e.clientY);
			window.addEventListener('mousemove', handler);
			return () => window.removeEventListener('mousemove', handler);
		});
	}

	get x() {
		return this.#getX();
	}

	get y() {
		return this.#getY();
	}

	get position() {
		return { x: this.x, y: this.y };
	}
}

export const mouse = new MouseTracker();
```

**Usage:**

```svelte
<script lang="ts">
	import { mouse } from '$lib/mouse-tracker.svelte';

	let distance = $derived(Math.sqrt(mouse.x ** 2 + mouse.y ** 2));
</script>

<div>
	<p>Mouse: ({mouse.x}, {mouse.y})</p>
	<p>Distance from origin: {distance.toFixed(0)}px</p>
</div>

<div class="follower" style="left: {mouse.x}px; top: {mouse.y}px"></div>
```

---

## Universal Reactivity Best Practices

### ✅ DO

- Use getters for reactive values, methods for actions
- Use `$effect.root()` for module-level effects
- Check `browser` before using localStorage or DOM APIs
- Use `createSubscriber` for expensive external subscriptions
- Export a single instance for truly global state

### ❌ DON'T

- Don't forget `$effect.root()` when using `$effect` in modules
- Don't access localStorage without checking `browser` (breaks SSR)
- Don't subscribe to events without cleanup
- Don't use methods when getters are more appropriate

### When to Use What

| Pattern                  | Use Case                                         |
| ------------------------ | ------------------------------------------------ |
| Functions + getters      | Simple shared state (counters, toggles)          |
| Object + getters/setters | Medium complexity, property-like access          |
| Classes                  | Complex state, multiple instances, encapsulation |
| Proxy                    | Need to intercept/validate property access       |
| `$effect.root()`         | Effects in module scope (localStorage sync)      |
| `createSubscriber`       | Lazy, expensive external subscriptions           |
