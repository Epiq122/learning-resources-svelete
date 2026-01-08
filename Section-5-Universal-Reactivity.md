# Section 5: Universal Reactivity & Shared State

## 📚 Learning Objectives

By the end of this section, you will:

- Understand universal reactivity (outside components)
- Extract reactive logic into reusable functions and classes
- Share state across multiple components
- Implement global effects
- Persist state to localStorage
- Master $effect.tracking() for conditional reactivity
- Create custom reactive stores with createSubscriber

---

## Table of Contents

- [Section 5: Universal Reactivity \& Shared State](#section-5-universal-reactivity--shared-state)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction to Universal Reactivity](#1-introduction-to-universal-reactivity)
    - [What is Universal Reactivity?](#what-is-universal-reactivity)
  - [2. Extracting Logic - Functions](#2-extracting-logic---functions)
    - [Why Extract Logic?](#why-extract-logic)
  - [3. Extracting Logic - Classes](#3-extracting-logic---classes)
    - [Why Use Reactive Classes?](#why-use-reactive-classes)
  - [4. Sharing State Across Components](#4-sharing-state-across-components)
    - [How to Share State?](#how-to-share-state)
  - [5. Global Effects](#5-global-effects)
    - [What are Global Effects?](#what-are-global-effects)
  - [6. LocalStorage Persistence - Method 1](#6-localstorage-persistence---method-1)
    - [How to Persist State?](#how-to-persist-state)
  - [7. LocalStorage Persistence - Method 2](#7-localstorage-persistence---method-2)
    - [Alternative Persistence Pattern](#alternative-persistence-pattern)
  - [8. $effect.tracking() - Conditional Reactivity](#8-effecttracking---conditional-reactivity)
    - [What is $effect.tracking()?](#what-is-effecttracking)
  - [9. createSubscriber - Custom Reactive Stores](#9-createsubscriber---custom-reactive-stores)
    - [What is createSubscriber?](#what-is-createsubscriber)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Introduction to Universal Reactivity

### What is Universal Reactivity?

Svelte 5's reactivity works ANYWHERE - not just in components! Create reactive state in modules, classes, or functions.

**Real-World Scenario:** You're building a theme manager that needs to be accessible from any component without prop drilling.

**What it does:** Demonstrates reactive state outside of components.

### 📁 Files to Create

**Reactive Module** (use `.svelte.ts` extension!):

- `src/lib/stores/theme.svelte.ts` - Theme manager with universal reactivity

**Demo Pages:**

- `src/routes/theme-demo/+page.svelte` - Page using the theme
- `src/lib/components/ThemeToggle.svelte` - Reusable theme toggle component

**Best Practices:**

- Use `.svelte.ts` extension for files with runes ($state, $derived, etc.)
- Export singleton instances for shared state
- Place in `src/lib/stores/` for state management modules
- Any component can import and use the same instance

```typescript
// lib/stores/theme.svelte.ts
type Theme = "light" | "dark" | "auto";

class ThemeManager {
  // Reactive state - works OUTSIDE components!
  // This state updates UI anywhere it's used in the component tree
  current = $state<Theme>("dark");
  systemPreference = $state<"light" | "dark">("dark");

  // Computed property: auto-recalculates when dependencies change
  // Returns the actual theme (handles 'auto' mode)
  get effective(): "light" | "dark" {
    if (this.current === "auto") {
      return this.systemPreference;
    }
    return this.current;
  }

  // Another computed property for convenience
  get isDark() {
    return this.effective === "dark";
  }

  constructor() {
    // Check system preference using media query
    if (typeof window !== "undefined") {
      const media = window.matchMedia("(prefers-color-scheme: dark)");
      this.systemPreference = media.matches ? "dark" : "light";

      media.addEventListener("change", (e) => {
        this.systemPreference = e.matches ? "dark" : "light";
      });
    }
  }

  setTheme(theme: Theme) {
    this.current = theme;
  }

  toggle() {
    this.current = this.effective === "dark" ? "light" : "dark";
  }
}

// Export singleton instance
export const themeManager = new ThemeManager();
```

```svelte
<!-- Component 1 -->
<script lang="ts">
	import { themeManager } from '$lib/theme.svelte';
</script>

<div
	class="p-5"
	class:bg-white={!themeManager.isDark}
	class:text-black={!themeManager.isDark}
	class:bg-gray-900={themeManager.isDark}
	class:text-white={themeManager.isDark}
>
	<h1>My App</h1>
	<button
		onclick={() => themeManager.toggle()}
		class="text-2xl bg-transparent border-none cursor-pointer"
	>
		{themeManager.isDark ? '🌙' : '☀️'}
	</button>
</div>
```

```svelte
<!-- Component 2 (different file, same theme) -->
<script lang="ts">
	import { themeManager } from '$lib/theme.svelte';
</script>

<div
	class="p-5 rounded-lg"
	class:bg-gray-100={!themeManager.isDark}
	class:bg-gray-800={themeManager.isDark}
	class:text-white={themeManager.isDark}
>
	<h2>Theme Settings</h2>
	<select bind:value={themeManager.current} class="p-2 rounded">
		<option value="light">Light</option>
		<option value="dark">Dark</option>
		<option value="auto">Auto</option>
	</select>
	<p>Current: {themeManager.effective}</p>
</div>
```

**Key Concepts:**

- **Universal reactivity**: $state works outside components
- **Singleton pattern**: One instance shared everywhere
- **No prop drilling**: Import and use directly
- **File extension**: Use `.svelte.ts` for reactive modules

---

## 2. Extracting Logic - Functions

### Why Extract Logic?

Reuse reactive logic across components without duplication.

**Real-World Scenario:** You're building multiple forms that all need validation logic.

**What it does:** Shows how to extract reactive logic into reusable functions.

### 📁 Files to Create

**Reusable Utility:**

- `src/lib/utils/validation.svelte.ts` - Validation helper functions

**Demo Forms:**

- `src/routes/form-validation/+page.svelte` - Page using validation

**Best Practices:**

- Extract common patterns into `src/lib/utils/`
- Use `.svelte.ts` for files using runes
- Return objects with getters/setters for reactive state
- Make functions composable and testable

```typescript
// lib/utils/validation.svelte.ts
export interface ValidationRule<T> {
  validate: (value: T) => boolean;
  message: string;
}

export function createValidator<T>(
  initialValue: T,
  rules: ValidationRule<T>[]
) {
  // Reactive state for the field value
  let value = $state(initialValue);
  // Track if user has interacted with field (for showing errors)
  let touched = $state(false);

  // Computed errors: only shows errors after field is touched
  // Runs all validation rules and collects error messages
  const errors = $derived.by(() => {
    if (!touched) return []; // Don't show errors until user touches field
    return rules
      .filter((rule) => !rule.validate(value))
      .map((rule) => rule.message);
  });

  const isValid = $derived(errors.length === 0);

  function validate() {
    touched = true;
  }

  function reset() {
    value = initialValue;
    touched = false;
  }

  return {
    get value() {
      return value;
    },
    set value(v: T) {
      value = v;
    },
    get errors() {
      return errors;
    },
    get isValid() {
      return isValid;
    },
    get touched() {
      return touched;
    },
    validate,
    reset,
  };
}

// Pre-built rules
export const rules = {
  required: (message = "This field is required"): ValidationRule<string> => ({
    validate: (value) => value.trim().length > 0,
    message,
  }),

  minLength: (min: number, message?: string): ValidationRule<string> => ({
    validate: (value) => value.length >= min,
    message: message || `Must be at least ${min} characters`,
  }),

  maxLength: (max: number, message?: string): ValidationRule<string> => ({
    validate: (value) => value.length <= max,
    message: message || `Must be at most ${max} characters`,
  }),

  email: (message = "Must be a valid email"): ValidationRule<string> => ({
    validate: (value) => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    message,
  }),

  pattern: (regex: RegExp, message: string): ValidationRule<string> => ({
    validate: (value) => regex.test(value),
    message,
  }),

  min: (min: number, message?: string): ValidationRule<number> => ({
    validate: (value) => value >= min,
    message: message || `Must be at least ${min}`,
  }),

  max: (max: number, message?: string): ValidationRule<number> => ({
    validate: (value) => value <= max,
    message: message || `Must be at most ${max}`,
  }),
};
```

```svelte
<script lang="ts">
	import { createValidator, rules } from '$lib/validation.svelte';

	const email = createValidator('', [rules.required('Email is required'), rules.email()]);

	const password = createValidator('', [
		rules.required('Password is required'),
		rules.minLength(8)
	]);

	const age = createValidator(0, [
		rules.min(18, 'Must be 18 or older'),
		rules.max(120, 'Invalid age')
	]);

	const formValid = $derived(email.isValid && password.isValid && age.isValid);

	function handleSubmit() {
		email.validate();
		password.validate();
		age.validate();

		if (formValid) {
			alert('Form submitted!');
		}
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-8">📝 Reusable Validation</h1>

	<form
		onsubmit|preventDefault={handleSubmit}
		class="max-w-lg mx-auto bg-gray-800 border-2 border-gray-700 rounded-xl p-8"
	>
		<div class="mb-6">
			<label class="block text-gray-300 font-semibold mb-2">Email</label>
			<input
				type="email"
				bind:value={email.value}
				onblur={() => email.validate()}
				class="w-full bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
			/>
			{#if email.touched && email.errors.length > 0}
				<div class="mt-2">
					{#each email.errors as error}
						<div class="text-red-400 text-sm mt-1">{error}</div>
					{/each}
				</div>
			{/if}
		</div>

		<div class="mb-6">
			<label class="block text-gray-300 font-semibold mb-2">Password</label>
			<input
				type="password"
				bind:value={password.value}
				onblur={() => password.validate()}
				class="w-full bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
			/>
			{#if password.touched && password.errors.length > 0}
				<div class="mt-2">
					{#each password.errors as error}
						<div class="text-red-400 text-sm mt-1">{error}</div>
					{/each}
				</div>
			{/if}
		</div>

		<div class="mb-6">
			<label class="block text-gray-300 font-semibold mb-2">Age</label>
			<input
				type="number"
				bind:value={age.value}
				onblur={() => age.validate()}
				class="w-full bg-gray-900 border-2 border-gray-700 text-white p-3 rounded-lg text-base focus:outline-none focus:border-blue-400"
			/>
			{#if age.touched && age.errors.length > 0}
				<div class="mt-2">
					{#each age.errors as error}
						<div class="text-red-400 text-sm mt-1">{error}</div>
					{/each}
				</div>
			{/if}
		</div>

		<button
			type="submit"
			disabled={!formValid && (email.touched || password.touched || age.touched)}
			class="w-full bg-blue-400 text-black border-none py-3.5 px-3.5 rounded-lg font-bold text-base cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
		>
			Submit
		</button>
	</form>
</div>
```

**Key Concepts:**

- **Factory functions**: Return reactive objects
- **Reusable logic**: Use across multiple components
- **Composition**: Combine validators flexibly
- **Separation of concerns**: Logic separate from UI

---

## 3. Extracting Logic - Classes

### Why Use Reactive Classes?

Encapsulate complex state and behavior in a reusable class.

**Real-World Scenario:** You're building a data grid that needs sorting, filtering, and pagination.

**What it does:** Shows a reactive data table class with full functionality.

```typescript
// lib/dataTable.svelte.ts
export interface Column<T> {
  key: keyof T;
  label: string;
  sortable?: boolean;
  filterable?: boolean;
}

export type SortDirection = "asc" | "desc" | null;

export class DataTable<T extends Record<string, any>> {
  data = $state<T[]>([]);
  columns = $state<Column<T>[]>([]);

  // Sorting
  sortColumn = $state<keyof T | null>(null);
  sortDirection = $state<SortDirection>(null);

  // Filtering
  filters = $state<Partial<Record<keyof T, string>>>({});

  // Pagination
  page = $state(1);
  pageSize = $state(10);

  constructor(data: T[], columns: Column<T>[]) {
    this.data = data;
    this.columns = columns;
  }

  // Computed filtered data - recalculates automatically when data or filters change
  get filtered(): T[] {
    let result = [...this.data];

    // Apply all active filters to the data
    for (const [key, value] of Object.entries(this.filters)) {
      if (value) {
        // Case-insensitive search for each filter
        result = result.filter((row) =>
          String(row[key]).toLowerCase().includes(value.toLowerCase())
        );
      }
    }

    return result;
  }

  // Computed sorted data - applies sorting to filtered data
  get sorted(): T[] {
    if (!this.sortColumn) return this.filtered;

    return [...this.filtered].sort((a, b) => {
      const aVal = a[this.sortColumn!];
      const bVal = b[this.sortColumn!];

      if (aVal < bVal) return this.sortDirection === "asc" ? -1 : 1;
      if (aVal > bVal) return this.sortDirection === "asc" ? 1 : -1;
      return 0;
    });
  }

  // Computed paginated data
  get paginated(): T[] {
    const start = (this.page - 1) * this.pageSize;
    const end = start + this.pageSize;
    return this.sorted.slice(start, end);
  }

  get totalPages(): number {
    return Math.ceil(this.sorted.length / this.pageSize);
  }

  get stats() {
    return {
      total: this.data.length,
      filtered: this.filtered.length,
      showing: this.paginated.length,
    };
  }

  // Methods
  sort(column: keyof T) {
    if (this.sortColumn === column) {
      // Cycle through: asc → desc → null
      if (this.sortDirection === "asc") {
        this.sortDirection = "desc";
      } else if (this.sortDirection === "desc") {
        this.sortColumn = null;
        this.sortDirection = null;
      } else {
        this.sortDirection = "asc";
      }
    } else {
      this.sortColumn = column;
      this.sortDirection = "asc";
    }
    this.page = 1; // Reset to first page
  }

  setFilter(column: keyof T, value: string) {
    this.filters[column] = value;
    this.page = 1; // Reset to first page
  }

  clearFilters() {
    this.filters = {};
    this.page = 1;
  }

  nextPage() {
    if (this.page < this.totalPages) {
      this.page++;
    }
  }

  prevPage() {
    if (this.page > 1) {
      this.page--;
    }
  }

  goToPage(page: number) {
    if (page >= 1 && page <= this.totalPages) {
      this.page = page;
    }
  }
}
```

```svelte
<script lang="ts">
	import { DataTable, type Column } from '$lib/dataTable.svelte';

	interface User {
		id: number;
		name: string;
		email: string;
		role: string;
		status: string;
	}

	const users: User[] = [
		{ id: 1, name: 'Alice Johnson', email: 'alice@example.com', role: 'Admin', status: 'Active' },
		{ id: 2, name: 'Bob Smith', email: 'bob@example.com', role: 'User', status: 'Active' },
		{ id: 3, name: 'Carol White', email: 'carol@example.com', role: 'User', status: 'Inactive' },
		{ id: 4, name: 'David Brown', email: 'david@example.com', role: 'Moderator', status: 'Active' },
		{ id: 5, name: 'Eve Davis', email: 'eve@example.com', role: 'User', status: 'Active' },
		{ id: 6, name: 'Frank Miller', email: 'frank@example.com', role: 'Admin', status: 'Active' },
		{ id: 7, name: 'Grace Lee', email: 'grace@example.com', role: 'User', status: 'Inactive' },
		{ id: 8, name: 'Henry Wilson', email: 'henry@example.com', role: 'User', status: 'Active' }
		// Add more for pagination demo...
	];

	const columns: Column<User>[] = [
		{ key: 'name', label: 'Name', sortable: true, filterable: true },
		{ key: 'email', label: 'Email', sortable: true, filterable: true },
		{ key: 'role', label: 'Role', sortable: true, filterable: true },
		{ key: 'status', label: 'Status', sortable: true, filterable: true }
	];

	const table = new DataTable(users, columns);
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-8">📊 Reactive Data Table Class</h1>

	<div class="max-w-7xl mx-auto bg-gray-800 border-2 border-gray-700 rounded-xl p-6">
		<div class="flex gap-6 mb-5 p-4 bg-gray-900 rounded-lg">
			<span class="text-blue-400 font-bold">Total: {table.stats.total}</span>
			<span class="text-blue-400 font-bold">Filtered: {table.stats.filtered}</span>
			<span class="text-blue-400 font-bold">Showing: {table.stats.showing}</span>
		</div>

		<div class="flex gap-3 mb-5">
			{#each table.columns.filter((c) => c.filterable) as column}
				<input
					type="text"
					placeholder="Filter {column.label}..."
					value={table.filters[column.key] || ''}
					oninput={(e) => table.setFilter(column.key, e.currentTarget.value)}
					class="flex-1 bg-gray-900 border-2 border-gray-700 text-white p-2.5 rounded-md"
				/>
			{/each}
			<button
				onclick={() => table.clearFilters()}
				class="bg-gray-700 text-white border-none py-2.5 px-5 rounded-md cursor-pointer"
				>Clear</button
			>
		</div>

		<div class="overflow-x-auto mb-5">
			<table class="w-full border-collapse">
				<thead>
					<tr>
						{#each table.columns as column}
							<th class="bg-gray-900 p-3 text-left font-bold text-blue-400">
								{#if column.sortable}
									<button
										onclick={() => table.sort(column.key)}
										class="bg-transparent border-none text-blue-400 font-bold cursor-pointer flex items-center gap-2"
									>
										{column.label}
										{#if table.sortColumn === column.key}
											{table.sortDirection === 'asc' ? '↑' : '↓'}
										{/if}
									</button>
								{:else}
									{column.label}
								{/if}
							</th>
						{/each}
					</tr>
				</thead>
				<tbody>
					{#each table.paginated as row}
						<tr class="hover:bg-gray-700">
							{#each table.columns as column}
								<td class="p-3 border-b border-gray-700">{row[column.key]}</td>
							{/each}
						</tr>
					{/each}
				</tbody>
			</table>
		</div>

		<div class="flex justify-center items-center gap-5">
			<button
				onclick={() => table.prevPage()}
				disabled={table.page === 1}
				class="bg-blue-400 text-black border-none py-2.5 px-5 rounded-md font-bold cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
			>
				← Prev
			</button>
			<span>Page {table.page} of {table.totalPages}</span>
			<button
				onclick={() => table.nextPage()}
				disabled={table.page === table.totalPages}
				class="bg-blue-400 text-black border-none py-2.5 px-5 rounded-md font-bold cursor-pointer disabled:opacity-50 disabled:cursor-not-allowed"
			>
				Next →
			</button>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **Encapsulation**: All table logic in one class
- **Computed properties**: Filtered, sorted, paginated
- **Methods**: Modify state, UI updates automatically
- **Reusability**: Use same class for different tables

---

## 4. Sharing State Across Components

### How to Share State?

Three methods: modules, Context API, or props.

**Real-World Scenario:** You're building a notification system that any component can trigger.

**What it does:** Shows a global notification manager.

```typescript
// lib/notifications.svelte.ts
export interface Notification {
  id: number;
  message: string;
  type: "info" | "success" | "warning" | "error";
  duration?: number;
}

class NotificationManager {
  // Reactive array - UI updates when notifications are added/removed
  notifications = $state<Notification[]>([]);
  nextId = 1;

  // Add a new notification with auto-dismiss
  add(message: string, type: Notification["type"] = "info", duration = 5000) {
    const notification: Notification = {
      id: this.nextId++,
      message,
      type,
      duration,
    };

    // Array mutation triggers reactivity in Svelte 5
    this.notifications.push(notification);

    // Auto-remove after duration (if duration > 0)
    if (duration > 0) {
      setTimeout(() => this.remove(notification.id), duration);
    }
  }

  remove(id: number) {
    this.notifications = this.notifications.filter((n) => n.id !== id);
  }

  clear() {
    this.notifications = [];
  }

  // Convenience methods
  info(message: string, duration?: number) {
    this.add(message, "info", duration);
  }

  success(message: string, duration?: number) {
    this.add(message, "success", duration);
  }

  warning(message: string, duration?: number) {
    this.add(message, "warning", duration);
  }

  error(message: string, duration?: number) {
    this.add(message, "error", duration);
  }
}

export const notifications = new NotificationManager();
```

```svelte
<!-- lib/NotificationContainer.svelte -->
<script lang="ts">
	import { notifications, type Notification } from './notifications.svelte';

	function getIcon(type: Notification['type']) {
		switch (type) {
			case 'success':
				return '✅';
			case 'warning':
				return '⚠️';
			case 'error':
				return '❌';
			default:
				return 'ℹ️';
		}
	}
</script>

<div class="fixed top-5 right-5 flex flex-col gap-3 z-50 max-w-sm">
	{#each notifications.notifications as notification (notification.id)}
		<div
			class="bg-gray-800 border-2 rounded-lg p-4 flex items-center gap-3 shadow-lg"
			class:border-blue-400={notification.type === 'info'}
			class:border-green-400={notification.type === 'success'}
			class:border-orange-500={notification.type === 'warning'}
			class:border-red-400={notification.type === 'error'}
		>
			<span class="text-2xl">{getIcon(notification.type)}</span>
			<span class="flex-1 text-white">{notification.message}</span>
			<button
				onclick={() => notifications.remove(notification.id)}
				class="bg-transparent border-none text-gray-500 cursor-pointer text-xl p-0 hover:text-white"
				>✕</button
			>
		</div>
	{/each}
</div>

<style>
	@keyframes slideIn {
		from {
			transform: translateX(100%);
			opacity: 0;
		}
		to {
			transform: translateX(0);
			opacity: 1;
		}
	}
</style>
```

```svelte
<!-- Usage in any component -->
<script lang="ts">
	import { notifications } from '$lib/notifications.svelte';
	import NotificationContainer from '$lib/NotificationContainer.svelte';

	function handleSuccess() {
		notifications.success('Operation completed successfully!');
	}

	function handleError() {
		notifications.error('Something went wrong!');
	}

	function handleInfo() {
		notifications.info('Here is some information');
	}
</script>

<NotificationContainer />

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200 text-center">
	<h1 class="text-blue-400 mb-8">🔔 Global Notifications</h1>

	<div class="flex gap-3 justify-center">
		<button
			onclick={handleSuccess}
			class="bg-blue-400 text-black border-none py-3 px-6 rounded-lg font-bold cursor-pointer"
			>Show Success</button
		>
		<button
			onclick={handleError}
			class="bg-blue-400 text-black border-none py-3 px-6 rounded-lg font-bold cursor-pointer"
			>Show Error</button
		>
		<button
			onclick={handleInfo}
			class="bg-blue-400 text-black border-none py-3 px-6 rounded-lg font-bold cursor-pointer"
			>Show Info</button
		>
	</div>
</div>
```

**Key Concepts:**

- **Global singleton**: One instance shared everywhere
- **Import anywhere**: No prop drilling needed
- **Automatic updates**: All components see changes
- **Module pattern**: Simple and effective

---

## 5. Global Effects

### What are Global Effects?

Run effects at module level, outside components.

**Real-World Scenario:** You're building analytics that tracks user activity globally.

**What it does:** Demonstrates global effects for cross-cutting concerns.

```typescript
// lib/analytics.svelte.ts
interface AnalyticsEvent {
  name: string;
  timestamp: Date;
  data?: Record<string, any>;
}

class Analytics {
  events = $state<AnalyticsEvent[]>([]);
  enabled = $state(true);
  sessionStart = new Date();

  get sessionDuration() {
    return Date.now() - this.sessionStart.getTime();
  }

  get eventCount() {
    return this.events.length;
  }

  constructor() {
    // Global effect: Track page visibility
    $effect(() => {
      if (typeof document === "undefined") return;

      const handleVisibilityChange = () => {
        this.track(document.hidden ? "page_hidden" : "page_visible");
      };

      document.addEventListener("visibilitychange", handleVisibilityChange);

      return () => {
        document.removeEventListener(
          "visibilitychange",
          handleVisibilityChange
        );
      };
    });

    // Global effect: Track online/offline
    $effect(() => {
      if (typeof window === "undefined") return;

      const handleOnline = () => this.track("online");
      const handleOffline = () => this.track("offline");

      window.addEventListener("online", handleOnline);
      window.addEventListener("offline", handleOffline);

      return () => {
        window.removeEventListener("online", handleOnline);
        window.removeEventListener("offline", handleOffline);
      };
    });

    // Log session stats every 30 seconds
    $effect(() => {
      const interval = setInterval(() => {
        console.log(
          `Session: ${this.eventCount} events in ${Math.round(this.sessionDuration / 1000)}s`
        );
      }, 30000);

      return () => clearInterval(interval);
    });
  }

  track(name: string, data?: Record<string, any>) {
    if (!this.enabled) return;

    const event: AnalyticsEvent = {
      name,
      timestamp: new Date(),
      data,
    };

    this.events.push(event);
    console.log("📊 Analytics:", event);
  }

  clear() {
    this.events = [];
  }
}

export const analytics = new Analytics();
```

**Key Concepts:**

- **Module-level effects**: Run outside components
- **Global listeners**: DOM events, network status
- **Lifecycle**: Cleanup with return function
- **Always active**: Not tied to component lifecycle

---

## 6. LocalStorage Persistence - Method 1

### How to Persist State?

Sync reactive state with localStorage automatically.

**Real-World Scenario:** You're building user preferences that persist across sessions.

**What it does:** Shows automatic localStorage sync.

```typescript
// lib/storage.svelte.ts
export function createPersistentState<T>(key: string, initialValue: T) {
  // Load from localStorage on initialization (hydration)
  let storedValue: T = initialValue;

  // Check if we're in browser (not SSR)
  if (typeof localStorage !== "undefined") {
    const stored = localStorage.getItem(key);
    if (stored) {
      try {
        storedValue = JSON.parse(stored);
      } catch (e) {
        console.error("Failed to parse stored value:", e);
      }
    }
  }

  // Create reactive state with stored or initial value
  let value = $state<T>(storedValue);

  // Effect: Automatically sync to localStorage when value changes
  // This runs every time 'value' is updated
  $effect(() => {
    if (typeof localStorage !== "undefined") {
      localStorage.setItem(key, JSON.stringify(value));
    }
  });

  return {
    get value() {
      return value;
    },
    set value(v: T) {
      value = v;
    },
    reset() {
      value = initialValue;
    },
  };
}
```

```svelte
<script lang="ts">
	import { createPersistentState } from '$lib/storage.svelte';

	interface UserPreferences {
		theme: 'light' | 'dark';
		fontSize: number;
		sidebarVisible: boolean;
	}

	const preferences = createPersistentState<UserPreferences>('user-prefs', {
		theme: 'dark',
		fontSize: 16,
		sidebarVisible: true
	});
</script>

<div
	class="transition-all duration-300 min-h-screen px-5 py-10"
	class:bg-white={preferences.value.theme !== 'dark'}
	class:text-black={preferences.value.theme !== 'dark'}
	class:bg-gray-900={preferences.value.theme === 'dark'}
	class:text-white={preferences.value.theme === 'dark'}
>
	<h1 class="text-center text-blue-400 mb-8">💾 Persistent Preferences</h1>

	<div
		class="max-w-lg mx-auto p-6 rounded-xl flex flex-col gap-5"
		class:bg-gray-100={preferences.value.theme !== 'dark'}
		class:bg-gray-800={preferences.value.theme === 'dark'}
	>
		<label class="flex flex-col gap-2 font-semibold">
			Theme:
			<select
				bind:value={preferences.value.theme}
				class="p-2 rounded-md border-2"
				class:border-gray-300={preferences.value.theme !== 'dark'}
				class:bg-gray-900={preferences.value.theme === 'dark'}
				class:border-gray-700={preferences.value.theme === 'dark'}
				class:text-white={preferences.value.theme === 'dark'}
			>
				<option value="light">Light</option>
				<option value="dark">Dark</option>
			</select>
		</label>

		<label class="flex flex-col gap-2 font-semibold">
			Font Size: {preferences.value.fontSize}px
			<input
				type="range"
				bind:value={preferences.value.fontSize}
				min="12"
				max="24"
				class="p-2 rounded-md border-2"
				class:border-gray-300={preferences.value.theme !== 'dark'}
				class:bg-gray-900={preferences.value.theme === 'dark'}
				class:border-gray-700={preferences.value.theme === 'dark'}
				class:text-white={preferences.value.theme === 'dark'}
			/>
		</label>

		<label class="flex flex-col gap-2 font-semibold">
			<input type="checkbox" bind:checked={preferences.value.sidebarVisible} />
			Show Sidebar
		</label>

		<button
			onclick={() => preferences.reset()}
			class="bg-red-400 text-white border-none p-3 rounded-lg font-bold cursor-pointer"
		>
			Reset to Defaults
		</button>
	</div>

	<p class="text-center mt-8 italic text-blue-400">✨ Reload the page - your settings persist!</p>
</div>
```

**Key Concepts:**

- **Automatic sync**: Changes save to localStorage
- **Initialization**: Load on module load
- **JSON serialization**: Handle complex objects
- **Type safety**: Full TypeScript support

---

## 7. LocalStorage Persistence - Method 2

### Alternative Persistence Pattern

An alternative approach using a generic helper function to create persistent reactive state.

**Real-World Scenario:** You need to persist multiple different pieces of state and want a reusable pattern.

**What it does:** Creates a generic `persisted` function that wraps any state with localStorage persistence.

### 📁 Files to Create

**Utility Function:**

- `src/lib/utils/persisted.ts` - Generic persistence helper

```typescript
// src/lib/utils/persisted.ts
import { untrack } from "svelte";

export interface PersistedOptions<T> {
  key: string;
  initial: T;
  storage?: Storage;
  serializer?: {
    parse: (text: string) => T;
    stringify: (object: T) => string;
  };
}

export function persisted<T>(options: PersistedOptions<T>) {
  const {
    key,
    initial,
    storage = typeof window !== "undefined" ? window.localStorage : undefined,
    serializer = JSON,
  } = options;

  // Load from storage
  let value = $state<T>(initial);

  if (storage) {
    const stored = storage.getItem(key);
    if (stored) {
      try {
        value = serializer.parse(stored);
      } catch (e) {
        console.error(`Failed to parse stored value for key "${key}":`, e);
      }
    }
  }

  // Watch for changes and save
  $effect(() => {
    // Access value to track it
    const current = value;

    // Skip saving on initial load
    untrack(() => {
      if (storage) {
        try {
          storage.setItem(key, serializer.stringify(current));
        } catch (e) {
          console.error(`Failed to save value for key "${key}":`, e);
        }
      }
    });
  });

  return {
    get value() {
      return value;
    },
    set value(newValue: T) {
      value = newValue;
    },
    clear() {
      value = initial;
      storage?.removeItem(key);
    },
  };
}
```

**Demo Page:**

- `src/routes/section-5/persisted-demo/+page.svelte` - Demo using persisted helper

```svelte
<script lang="ts">
	import { persisted } from '$lib/utils/persisted';

	// Create multiple persisted states
	const username = persisted({
		key: 'demo-username',
		initial: ''
	});

	const theme = persisted({
		key: 'demo-theme',
		initial: 'light' as 'light' | 'dark'
	});

	const settings = persisted({
		key: 'demo-settings',
		initial: {
			fontSize: 16,
			notifications: true,
			language: 'en'
		}
	});

	const todos = persisted<string[]>({
		key: 'demo-todos',
		initial: []
	});

	let newTodo = $state('');

	function addTodo() {
		if (!newTodo.trim()) return;
		todos.value = [...todos.value, newTodo.trim()];
		newTodo = '';
	}

	function removeTodo(index: number) {
		todos.value = todos.value.filter((_, i) => i !== index);
	}

	function clearAll() {
		username.clear();
		theme.clear();
		settings.clear();
		todos.clear();
	}
</script>

<div
	class={`min-h-screen p-8 transition-colors ${
		theme.value === 'dark'
			? 'bg-gradient-to-br from-gray-900 to-gray-800 text-white'
			: 'bg-gradient-to-br from-cyan-50 to-blue-100 text-gray-900'
	}`}
>
	<div class="max-w-5xl mx-auto">
		<h1 class="text-4xl font-bold mb-2">Persisted State - Method 2</h1>
		<p class={`mb-8 ${theme.value === 'dark' ? 'text-gray-400' : 'text-gray-600'}`}>
			Generic helper function for localStorage persistence
		</p>

		<div class="grid grid-cols-2 gap-6">
			<!-- User Profile -->
			<div
				class={`rounded-lg shadow-lg p-6 ${theme.value === 'dark' ? 'bg-gray-800' : 'bg-white'}`}
			>
				<h2 class="text-xl font-semibold mb-4">User Profile</h2>
				<div class="space-y-4">
					<div>
						<label class="block text-sm font-medium mb-2">Username</label>
						<input
							type="text"
							bind:value={username.value}
							placeholder="Enter username"
							class={`w-full px-4 py-2 rounded-lg border focus:outline-none focus:ring-2 focus:ring-cyan-500 ${
								theme.value === 'dark'
									? 'bg-gray-700 border-gray-600 text-white'
									: 'bg-white border-gray-300'
							}`}
						/>
					</div>
					<div>
						<label class="block text-sm font-medium mb-2">Theme</label>
						<div class="flex gap-2">
							<button
								onclick={() => (theme.value = 'light')}
								class={`flex-1 px-4 py-2 rounded-lg border-2 transition ${
									theme.value === 'light'
										? 'border-cyan-500 bg-cyan-50 text-cyan-700'
										: theme.value === 'dark'
											? 'border-gray-600 bg-gray-700 text-gray-300'
											: 'border-gray-300 hover:border-gray-400'
								}`}
							>
								☀️ Light
							</button>
							<button
								onclick={() => (theme.value = 'dark')}
								class={`flex-1 px-4 py-2 rounded-lg border-2 transition ${
									theme.value === 'dark'
										? 'border-cyan-500 bg-cyan-900 text-cyan-300'
										: 'border-gray-300 hover:border-gray-400'
								}`}
							>
								🌙 Dark
							</button>
						</div>
					</div>
					{#if username.value}
						<div class={`p-3 rounded-lg ${theme.value === 'dark' ? 'bg-gray-700' : 'bg-gray-100'}`}>
							<div class="text-sm font-medium">Greeting:</div>
							<div class="text-lg">Hello, {username.value}! 👋</div>
						</div>
					{/if}
				</div>
			</div>

			<!-- Settings -->
			<div
				class={`rounded-lg shadow-lg p-6 ${theme.value === 'dark' ? 'bg-gray-800' : 'bg-white'}`}
			>
				<h2 class="text-xl font-semibold mb-4">Settings</h2>
				<div class="space-y-4">
					<div>
						<label class="flex items-center justify-between">
							<span class="text-sm font-medium">Font Size: {settings.value.fontSize}px</span>
						</label>
						<input
							type="range"
							bind:value={settings.value.fontSize}
							min="12"
							max="24"
							step="1"
							class="w-full"
						/>
					</div>
					<div>
						<label class="flex items-center justify-between">
							<span class="text-sm font-medium">Notifications</span>
							<input type="checkbox" bind:checked={settings.value.notifications} class="w-5 h-5" />
						</label>
					</div>
					<div>
						<label class="block text-sm font-medium mb-2">Language</label>
						<select
							bind:value={settings.value.language}
							class={`w-full px-4 py-2 rounded-lg border focus:outline-none focus:ring-2 focus:ring-cyan-500 ${
								theme.value === 'dark'
									? 'bg-gray-700 border-gray-600 text-white'
									: 'bg-white border-gray-300'
							}`}
						>
							<option value="en">English</option>
							<option value="es">Español</option>
							<option value="fr">Français</option>
							<option value="de">Deutsch</option>
						</select>
					</div>
					<div
						class={`p-3 rounded-lg text-sm ${theme.value === 'dark' ? 'bg-gray-700' : 'bg-gray-100'}`}
					>
						<div class="font-mono space-y-1">
							<div>fontSize: {settings.value.fontSize}</div>
							<div>notifications: {settings.value.notifications}</div>
							<div>language: {settings.value.language}</div>
						</div>
					</div>
				</div>
			</div>

			<!-- Todo List -->
			<div
				class={`col-span-2 rounded-lg shadow-lg p-6 ${
					theme.value === 'dark' ? 'bg-gray-800' : 'bg-white'
				}`}
			>
				<h2 class="text-xl font-semibold mb-4">Persisted Todos</h2>
				<form
					onsubmit={(e) => {
						e.preventDefault();
						addTodo();
					}}
					class="mb-4"
				>
					<div class="flex gap-2">
						<input
							type="text"
							bind:value={newTodo}
							placeholder="Add a todo..."
							class={`flex-1 px-4 py-2 rounded-lg border focus:outline-none focus:ring-2 focus:ring-cyan-500 ${
								theme.value === 'dark'
									? 'bg-gray-700 border-gray-600 text-white'
									: 'bg-white border-gray-300'
							}`}
						/>
						<button
							type="submit"
							class="px-6 py-2 bg-cyan-500 text-white rounded-lg hover:bg-cyan-600 transition font-medium"
						>
							Add
						</button>
					</div>
				</form>

				{#if todos.value.length === 0}
					<div
						class={`text-center py-8 ${theme.value === 'dark' ? 'text-gray-500' : 'text-gray-400'}`}
					>
						No todos yet. Add one to get started!
					</div>
				{:else}
					<div class="space-y-2">
						{#each todos.value as todo, i (i)}
							<div
								class={`flex items-center justify-between p-3 rounded-lg ${
									theme.value === 'dark' ? 'bg-gray-700' : 'bg-gray-100'
								}`}
							>
								<span>{todo}</span>
								<button onclick={() => removeTodo(i)} class="text-red-500 hover:text-red-700">
									Delete
								</button>
							</div>
						{/each}
					</div>
				{/if}
			</div>
		</div>

		<!-- Actions -->
		<div class="mt-6 flex gap-4 justify-center">
			<button
				onclick={clearAll}
				class={`px-6 py-3 rounded-lg border-2 transition font-medium ${
					theme.value === 'dark'
						? 'border-red-600 text-red-400 hover:bg-red-900/20'
						: 'border-red-500 text-red-600 hover:bg-red-50'
				}`}
			>
				Clear All Data
			</button>
			<button
				onclick={() => window.location.reload()}
				class="px-6 py-3 bg-cyan-500 text-white rounded-lg hover:bg-cyan-600 transition font-medium"
			>
				Reload Page
			</button>
		</div>

		<p
			class={`text-center mt-6 italic ${theme.value === 'dark' ? 'text-cyan-400' : 'text-cyan-600'}`}
		>
			✨ All changes are automatically persisted!
		</p>
	</div>
</div>
```

**Key Concepts:**

- **Generic helper**: Reusable pattern for any type
- **Options object**: Customize key, initial value, storage
- **Custom serializer**: Support non-JSON data if needed
- **Clear method**: Reset to initial value and remove from storage
- **Type safety**: Full TypeScript support with generics

---

## 8. $effect.tracking() - Conditional Reactivity

### What is $effect.tracking()?

`$effect.tracking()` tells you if code is running inside a reactive context. Use it to conditionally track dependencies.

**Real-World Scenario:** You're building a library that works both inside and outside of Svelte components.

**What it does:** Demonstrates how to use `$effect.tracking()` to create flexible reactive code.

### 📁 Files to Create

**Main Demo:**

- `src/routes/section-5/tracking-demo/+page.svelte` - $effect.tracking() examples

```svelte
<script lang="ts">
	import { untrack } from 'svelte';

	// Example 1: Conditional logging
	let count = $state(0);
	let name = $state('');
	let enableLogging = $state(true);

	// This effect tracks count conditionally
	$effect(() => {
		if (enableLogging && $effect.tracking()) {
			console.log('Count changed to:', count);
		}
	});

	// Example 2: Flexible reactive function
	function createReactiveValue<T>(initial: T) {
		let value = $state(initial);
		let updateCount = $state(0);

		// Only track if we're in a reactive context
		$effect(() => {
			if ($effect.tracking()) {
				const v = value;
				untrack(() => {
					updateCount++;
					console.log(`Value updated ${updateCount} times:`, v);
				});
			}
		});

		return {
			get value() {
				return value;
			},
			set value(newValue: T) {
				value = newValue;
			},
			get updateCount() {
				return updateCount;
			}
		};
	}

	const reactiveNumber = createReactiveValue(0);
	const reactiveText = createReactiveValue('Hello');

	// Example 3: Conditional tracking in derived state
	let temperature = $state(20);
	let unit = $state<'C' | 'F'>('C');
	let showWarning = $state(true);

	const displayTemp = $derived(() => {
		if (unit === 'F') {
			return (temperature * 9) / 5 + 32;
		}
		return temperature;
	});

	$effect(() => {
		if (showWarning && $effect.tracking()) {
			if (temperature > 30) {
				console.warn('High temperature warning!', temperature);
			}
		}
	});

	// Example 4: Debug mode with tracking
	let debugMode = $state(false);
	let apiCalls = $state(0);
	let stateChanges = $state<string[]>([]);

	function trackChange(label: string) {
		if (debugMode && $effect.tracking()) {
			stateChanges = [...stateChanges, `${new Date().toLocaleTimeString()}: ${label}`];
		}
	}

	$effect(() => {
		trackChange(`Count changed to ${count}`);
	});

	$effect(() => {
		trackChange(`Name changed to "${name}"`);
	});

	// Example 5: Performance monitoring
	let performanceLog = $state<{ operation: string; isTracking: boolean; timestamp: Date }[]>([]);

	function performOperation(operation: string) {
		const isTracking = $effect.tracking();
		performanceLog.push({
			operation,
			isTracking,
			timestamp: new Date()
		});
		performanceLog = performanceLog; // Trigger reactivity
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-purple-50 to-pink-100 p-8">
	<div class="max-w-6xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">$effect.tracking() Demo</h1>
		<p class="text-gray-600 mb-8">Conditional reactivity and tracking detection</p>

		<div class="grid grid-cols-2 gap-6">
			<!-- Example 1: Conditional Logging -->
			<div class="bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">1. Conditional Logging</h2>
				<div class="space-y-4">
					<div>
						<label class="flex items-center gap-2 mb-2">
							<input type="checkbox" bind:checked={enableLogging} class="w-5 h-5" />
							<span class="text-sm font-medium">Enable Console Logging</span>
						</label>
					</div>
					<div>
						<label class="block text-sm font-medium mb-2">Count: {count}</label>
						<div class="flex gap-2">
							<button
								onclick={() => count--}
								class="flex-1 px-4 py-2 bg-red-500 text-white rounded-lg hover:bg-red-600"
							>
								-
							</button>
							<button
								onclick={() => count++}
								class="flex-1 px-4 py-2 bg-green-500 text-white rounded-lg hover:bg-green-600"
							>
								+
							</button>
						</div>
					</div>
					<div class="p-3 bg-purple-50 rounded text-sm">
						Open DevTools Console to see conditional logs
					</div>
				</div>
			</div>

			<!-- Example 2: Reactive Value with Tracking -->
			<div class="bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">2. Reactive Value</h2>
				<div class="space-y-4">
					<div>
						<label class="block text-sm font-medium mb-2">
							Number: {reactiveNumber.value}
						</label>
						<input
							type="range"
							bind:value={reactiveNumber.value}
							min="0"
							max="100"
							class="w-full"
						/>
						<div class="text-xs text-gray-600 mt-1">
							Updated {reactiveNumber.updateCount} times
						</div>
					</div>
					<div>
						<label class="block text-sm font-medium mb-2">Text: {reactiveText.value}</label>
						<input
							type="text"
							bind:value={reactiveText.value}
							class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-500"
						/>
						<div class="text-xs text-gray-600 mt-1">
							Updated {reactiveText.updateCount} times
						</div>
					</div>
				</div>
			</div>

			<!-- Example 3: Temperature Warning -->
			<div class="bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">3. Temperature Monitor</h2>
				<div class="space-y-4">
					<div>
						<label class="flex items-center gap-2 mb-2">
							<input type="checkbox" bind:checked={showWarning} class="w-5 h-5" />
							<span class="text-sm font-medium">Show Warnings</span>
						</label>
					</div>
					<div>
						<label class="block text-sm font-medium mb-2">
							Temperature: {temperature}°C = {displayTemp().toFixed(1)}°F
						</label>
						<input type="range" bind:value={temperature} min="-10" max="50" class="w-full" />
					</div>
					<div class="flex gap-2">
						<button
							onclick={() => (unit = 'C')}
							class={`flex-1 px-4 py-2 rounded-lg transition ${
								unit === 'C' ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-700'
							}`}
						>
							°C
						</button>
						<button
							onclick={() => (unit = 'F')}
							class={`flex-1 px-4 py-2 rounded-lg transition ${
								unit === 'F' ? 'bg-blue-500 text-white' : 'bg-gray-200 text-gray-700'
							}`}
						>
							°F
						</button>
					</div>
					{#if temperature > 30}
						<div class="p-3 bg-red-50 border border-red-200 rounded text-red-700 text-sm">
							⚠️ High temperature warning! Check console.
						</div>
					{/if}
				</div>
			</div>

			<!-- Example 4: Debug Mode -->
			<div class="bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">4. Debug Mode</h2>
				<div class="space-y-4">
					<div>
						<label class="flex items-center gap-2 mb-2">
							<input type="checkbox" bind:checked={debugMode} class="w-5 h-5" />
							<span class="text-sm font-medium">Enable Debug Mode</span>
						</label>
					</div>
					<div>
						<input
							type="text"
							bind:value={name}
							placeholder="Type your name..."
							class="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-500"
						/>
					</div>
					{#if debugMode && stateChanges.length > 0}
						<div class="p-3 bg-gray-50 rounded max-h-40 overflow-y-auto">
							<div class="text-xs font-semibold text-gray-700 mb-2">
								State Changes ({stateChanges.length}):
							</div>
							<div class="space-y-1 text-xs font-mono text-gray-600">
								{#each stateChanges.slice(-5) as change}
									<div>{change}</div>
								{/each}
							</div>
						</div>
					{/if}
					<button
						onclick={() => (stateChanges = [])}
						class="w-full px-4 py-2 bg-gray-500 text-white rounded-lg hover:bg-gray-600 transition"
					>
						Clear Log
					</button>
				</div>
			</div>

			<!-- Example 5: Performance Monitoring -->
			<div class="col-span-2 bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">5. Tracking Detection</h2>
				<div class="space-y-4">
					<div class="flex gap-2">
						<button
							onclick={() => performOperation('Button Click')}
							class="px-4 py-2 bg-purple-500 text-white rounded-lg hover:bg-purple-600"
						>
							Perform Operation
						</button>
						<button
							onclick={() => {
								$effect(() => {
									performOperation('Inside Effect');
								});
							}}
							class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-600"
						>
							Perform in Effect
						</button>
						<button
							onclick={() => (performanceLog = [])}
							class="px-4 py-2 bg-gray-500 text-white rounded-lg hover:bg-gray-600"
						>
							Clear Log
						</button>
					</div>

					{#if performanceLog.length > 0}
						<div class="overflow-x-auto">
							<table class="w-full text-sm">
								<thead class="bg-gray-50">
									<tr>
										<th class="px-4 py-2 text-left">Operation</th>
										<th class="px-4 py-2 text-left">Tracking?</th>
										<th class="px-4 py-2 text-left">Timestamp</th>
									</tr>
								</thead>
								<tbody class="divide-y divide-gray-200">
									{#each performanceLog as log}
										<tr>
											<td class="px-4 py-2">{log.operation}</td>
											<td class="px-4 py-2">
												<span
													class={`px-2 py-1 rounded text-xs font-medium ${
														log.isTracking
															? 'bg-green-100 text-green-700'
															: 'bg-gray-100 text-gray-700'
													}`}
												>
													{log.isTracking ? '✓ Yes' : '✗ No'}
												</span>
											</td>
											<td class="px-4 py-2 text-xs text-gray-600">
												{log.timestamp.toLocaleTimeString()}
											</td>
										</tr>
									{/each}
								</tbody>
							</table>
						</div>
					{/if}
				</div>
			</div>
		</div>

		<!-- Key Points -->
		<div class="mt-6 bg-white rounded-lg shadow-lg p-6">
			<h3 class="text-lg font-semibold text-gray-900 mb-4">Key Concepts</h3>
			<div class="grid grid-cols-2 gap-4 text-sm">
				<div class="border-l-4 border-purple-500 pl-4">
					<div class="font-semibold text-purple-900 mb-1">$effect.tracking()</div>
					<div class="text-gray-700">Returns true if inside reactive context</div>
				</div>
				<div class="border-l-4 border-blue-500 pl-4">
					<div class="font-semibold text-blue-900 mb-1">Conditional tracking</div>
					<div class="text-gray-700">Only track dependencies when needed</div>
				</div>
				<div class="border-l-4 border-green-500 pl-4">
					<div class="font-semibold text-green-900 mb-1">Library code</div>
					<div class="text-gray-700">Works both inside and outside components</div>
				</div>
				<div class="border-l-4 border-orange-500 pl-4">
					<div class="font-semibold text-orange-900 mb-1">Debug mode</div>
					<div class="text-gray-700">Toggle logging based on tracking state</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **$effect.tracking()**: Returns `true` if code is running inside a reactive context
- **Conditional reactivity**: Only track dependencies when needed
- **Library compatibility**: Write code that works inside and outside components
- **Debug modes**: Enable/disable tracking based on conditions
- **Performance**: Avoid unnecessary reactive subscriptions

---

## 9. createSubscriber - Custom Reactive Stores

### What is createSubscriber?

`createSubscriber` from `svelte/reactivity` creates custom reactive stores with full control over subscriptions and updates.

**Real-World Scenario:** You're building a WebSocket connection manager that needs to notify multiple components when data arrives.

**What it does:** Shows how to create custom reactive stores using the low-level `createSubscriber` API.

### 📁 Files to Create

**Custom Store:**

- `src/lib/stores/customStore.ts` - Custom reactive stores

```typescript
// src/lib/stores/customStore.ts
import { createSubscriber } from "svelte/reactivity";

export interface WebSocketMessage {
  type: string;
  data: unknown;
  timestamp: Date;
}

// Example 1: Simple reactive value with createSubscriber
export function createReactiveValue<T>(initial: T) {
  let value = $state(initial);

  const subscriber = createSubscriber((update) => {
    $effect(() => {
      // Track value changes
      const current = value;
      // Notify subscribers
      update();
    });

    return () => {
      // Cleanup if needed
    };
  });

  return {
    get value() {
      subscriber.subscribe();
      return value;
    },
    set value(newValue: T) {
      value = newValue;
    },
  };
}

// Example 2: WebSocket manager
export function createWebSocketStore(url: string) {
  let socket: WebSocket | null = null;
  let messages = $state<WebSocketMessage[]>([]);
  let isConnected = $state(false);
  let error = $state<string | null>(null);

  const subscriber = createSubscriber((update) => {
    $effect(() => {
      // Track messages, isConnected, error
      messages;
      isConnected;
      error;
      // Notify subscribers
      update();
    });

    return () => {
      // Cleanup
      if (socket) {
        socket.close();
      }
    };
  });

  function connect() {
    try {
      socket = new WebSocket(url);

      socket.onopen = () => {
        isConnected = true;
        error = null;
      };

      socket.onmessage = (event) => {
        messages.push({
          type: "message",
          data: event.data,
          timestamp: new Date(),
        });
        messages = messages; // Trigger reactivity
      };

      socket.onerror = () => {
        error = "Connection error";
        isConnected = false;
      };

      socket.onclose = () => {
        isConnected = false;
      };
    } catch (e) {
      error = e instanceof Error ? e.message : "Failed to connect";
    }
  }

  function send(data: string) {
    if (socket && isConnected) {
      socket.send(data);
    }
  }

  function disconnect() {
    if (socket) {
      socket.close();
      socket = null;
    }
  }

  return {
    get messages() {
      subscriber.subscribe();
      return messages;
    },
    get isConnected() {
      subscriber.subscribe();
      return isConnected;
    },
    get error() {
      subscriber.subscribe();
      return error;
    },
    connect,
    send,
    disconnect,
    clear: () => {
      messages = [];
    },
  };
}

// Example 3: Interval counter with createSubscriber
export function createIntervalCounter(intervalMs: number = 1000) {
  let count = $state(0);
  let isRunning = $state(false);
  let intervalId: number | null = null;

  const subscriber = createSubscriber((update) => {
    $effect(() => {
      // Track count and isRunning
      count;
      isRunning;
      // Notify subscribers
      update();
    });

    return () => {
      if (intervalId !== null) {
        clearInterval(intervalId);
      }
    };
  });

  function start() {
    if (isRunning) return;

    isRunning = true;
    intervalId = window.setInterval(() => {
      count++;
    }, intervalMs);
  }

  function stop() {
    if (intervalId !== null) {
      clearInterval(intervalId);
      intervalId = null;
    }
    isRunning = false;
  }

  function reset() {
    stop();
    count = 0;
  }

  return {
    get count() {
      subscriber.subscribe();
      return count;
    },
    get isRunning() {
      subscriber.subscribe();
      return isRunning;
    },
    start,
    stop,
    reset,
  };
}
```

**Demo Page:**

- `src/routes/section-5/subscriber-demo/+page.svelte` - createSubscriber demo

```svelte
<script lang="ts">
	import { createReactiveValue, createIntervalCounter } from '$lib/stores/customStore';

	// Example 1: Simple reactive value
	const reactiveValue = createReactiveValue('Hello, Svelte 5!');

	// Example 2: Interval counters
	const counter1 = createIntervalCounter(1000);
	const counter2 = createIntervalCounter(500);
	const counter3 = createIntervalCounter(2000);

	// Example 3: Multiple instances share the same pattern
	const temperature = createReactiveValue(20);
	const humidity = createReactiveValue(50);
	const pressure = createReactiveValue(1013);
</script>

<div class="min-h-screen bg-gradient-to-br from-teal-50 to-green-100 p-8">
	<div class="max-w-6xl mx-auto">
		<h1 class="text-4xl font-bold text-gray-900 mb-2">createSubscriber Pattern</h1>
		<p class="text-gray-600 mb-8">Build custom reactive stores with full control</p>

		<div class="grid grid-cols-2 gap-6">
			<!-- Example 1: Simple Reactive Value -->
			<div class="bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">1. Reactive Value Store</h2>
				<div class="space-y-4">
					<div>
						<label class="block text-sm font-medium mb-2">Edit Value:</label>
						<input
							type="text"
							bind:value={reactiveValue.value}
							class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-teal-500"
						/>
					</div>
					<div class="p-4 bg-teal-50 rounded-lg">
						<div class="text-sm text-gray-600 mb-1">Current Value:</div>
						<div class="text-lg font-medium text-teal-700">{reactiveValue.value}</div>
					</div>
					<div class="text-sm text-gray-600">
						This value is reactive and notifies all subscribers when changed.
					</div>
				</div>
			</div>

			<!-- Example 2: Interval Counters -->
			<div class="bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">2. Interval Counters</h2>
				<div class="space-y-4">
					<!-- Counter 1: Every 1 second -->
					<div class="border border-gray-200 rounded-lg p-4">
						<div class="flex items-center justify-between mb-2">
							<div>
								<div class="text-sm text-gray-600">1 Second Counter</div>
								<div class="text-3xl font-bold text-teal-600">{counter1.count}</div>
							</div>
							<div class="flex gap-2">
								<button
									onclick={() => counter1.start()}
									disabled={counter1.isRunning}
									class="px-3 py-1 bg-green-500 text-white rounded text-sm hover:bg-green-600 disabled:opacity-50 disabled:cursor-not-allowed"
								>
									Start
								</button>
								<button
									onclick={() => counter1.stop()}
									disabled={!counter1.isRunning}
									class="px-3 py-1 bg-red-500 text-white rounded text-sm hover:bg-red-600 disabled:opacity-50 disabled:cursor-not-allowed"
								>
									Stop
								</button>
								<button
									onclick={() => counter1.reset()}
									class="px-3 py-1 bg-gray-500 text-white rounded text-sm hover:bg-gray-600"
								>
									Reset
								</button>
							</div>
						</div>
					</div>

					<!-- Counter 2: Every 0.5 seconds -->
					<div class="border border-gray-200 rounded-lg p-4">
						<div class="flex items-center justify-between mb-2">
							<div>
								<div class="text-sm text-gray-600">0.5 Second Counter</div>
								<div class="text-3xl font-bold text-blue-600">{counter2.count}</div>
							</div>
							<div class="flex gap-2">
								<button
									onclick={() => counter2.start()}
									disabled={counter2.isRunning}
									class="px-3 py-1 bg-green-500 text-white rounded text-sm hover:bg-green-600 disabled:opacity-50"
								>
									Start
								</button>
								<button
									onclick={() => counter2.stop()}
									disabled={!counter2.isRunning}
									class="px-3 py-1 bg-red-500 text-white rounded text-sm hover:bg-red-600 disabled:opacity-50"
								>
									Stop
								</button>
								<button
									onclick={() => counter2.reset()}
									class="px-3 py-1 bg-gray-500 text-white rounded text-sm hover:bg-gray-600"
								>
									Reset
								</button>
							</div>
						</div>
					</div>

					<!-- Counter 3: Every 2 seconds -->
					<div class="border border-gray-200 rounded-lg p-4">
						<div class="flex items-center justify-between mb-2">
							<div>
								<div class="text-sm text-gray-600">2 Second Counter</div>
								<div class="text-3xl font-bold text-purple-600">{counter3.count}</div>
							</div>
							<div class="flex gap-2">
								<button
									onclick={() => counter3.start()}
									disabled={counter3.isRunning}
									class="px-3 py-1 bg-green-500 text-white rounded text-sm hover:bg-green-600 disabled:opacity-50"
								>
									Start
								</button>
								<button
									onclick={() => counter3.stop()}
									disabled={!counter3.isRunning}
									class="px-3 py-1 bg-red-500 text-white rounded text-sm hover:bg-red-600 disabled:opacity-50"
								>
									Stop
								</button>
								<button
									onclick={() => counter3.reset()}
									class="px-3 py-1 bg-gray-500 text-white rounded text-sm hover:bg-gray-600"
								>
									Reset
								</button>
							</div>
						</div>
					</div>
				</div>
			</div>

			<!-- Example 3: Sensor Dashboard -->
			<div class="col-span-2 bg-white rounded-lg shadow-lg p-6">
				<h2 class="text-xl font-semibold text-gray-900 mb-4">3. Sensor Dashboard</h2>
				<div class="grid grid-cols-3 gap-4">
					<!-- Temperature -->
					<div class="border border-gray-200 rounded-lg p-4">
						<div class="text-sm text-gray-600 mb-2">Temperature (°C)</div>
						<div class="text-4xl font-bold text-red-600 mb-3">{temperature.value}</div>
						<input type="range" bind:value={temperature.value} min="-10" max="50" class="w-full" />
					</div>

					<!-- Humidity -->
					<div class="border border-gray-200 rounded-lg p-4">
						<div class="text-sm text-gray-600 mb-2">Humidity (%)</div>
						<div class="text-4xl font-bold text-blue-600 mb-3">{humidity.value}</div>
						<input type="range" bind:value={humidity.value} min="0" max="100" class="w-full" />
					</div>

					<!-- Pressure -->
					<div class="border border-gray-200 rounded-lg p-4">
						<div class="text-sm text-gray-600 mb-2">Pressure (hPa)</div>
						<div class="text-4xl font-bold text-green-600 mb-3">{pressure.value}</div>
						<input type="range" bind:value={pressure.value} min="950" max="1050" class="w-full" />
					</div>
				</div>
			</div>
		</div>

		<!-- Pattern Explanation -->
		<div class="mt-6 bg-white rounded-lg shadow-lg p-6">
			<h3 class="text-lg font-semibold text-gray-900 mb-4">createSubscriber Pattern</h3>
			<div class="grid grid-cols-2 gap-4 text-sm">
				<div class="border-l-4 border-teal-500 pl-4 bg-teal-50 p-3 rounded">
					<div class="font-semibold text-teal-900 mb-1">1. Create Subscriber</div>
					<div class="text-teal-700 font-mono text-xs">createSubscriber((update) => ...)</div>
					<div class="text-teal-600 mt-2">Initialize the subscription system</div>
				</div>
				<div class="border-l-4 border-blue-500 pl-4 bg-blue-50 p-3 rounded">
					<div class="font-semibold text-blue-900 mb-1">2. Track State</div>
					<div class="text-blue-700 font-mono text-xs">$effect(() => ... update())</div>
					<div class="text-blue-600 mt-2">Watch for changes and notify</div>
				</div>
				<div class="border-l-4 border-green-500 pl-4 bg-green-50 p-3 rounded">
					<div class="font-semibold text-green-900 mb-1">3. Subscribe on Access</div>
					<div class="text-green-700 font-mono text-xs">subscriber.subscribe()</div>
					<div class="text-green-600 mt-2">Register component as subscriber</div>
				</div>
				<div class="border-l-4 border-purple-500 pl-4 bg-purple-50 p-3 rounded">
					<div class="font-semibold text-purple-900 mb-1">4. Cleanup</div>
					<div class="text-purple-700 font-mono text-xs">return () => cleanup()</div>
					<div class="text-purple-600 mt-2">Clean up resources when done</div>
				</div>
			</div>
		</div>
	</div>
</div>
```

**Key Concepts:**

- **createSubscriber**: Low-level API for custom reactive stores
- **Full control**: Manage when and how subscribers are notified
- **Reusable pattern**: Create stores for any use case
- **Cleanup**: Return cleanup function from subscriber
- **Performance**: Only notify when needed
- **Type safety**: Full TypeScript support

---

## 📝 Key Takeaways

✅ Universal reactivity works outside components
✅ Extract logic into functions and classes
✅ Share state without prop drilling
✅ Global effects for cross-cutting concerns
✅ Persist state to localStorage automatically
✅ $effect.tracking() for conditional reactivity
✅ createSubscriber for custom reactive stores

### 💡 Best Practices for State Management

**Reactive Classes vs Functions:**

```typescript
// ✅ Use classes for complex state with multiple methods
class UserManager {
  users = $state<User[]>([]);
  loading = $state(false);

  async loadUsers() {
    /* ... */
  }
  findById(id: number) {
    /* ... */
  }
  updateUser(id: number, data: Partial<User>) {
    /* ... */
  }
}

// ✅ Use functions for simple, stateless logic
function calculateDiscount(price: number, percent: number) {
  return price * (1 - percent / 100);
}
```

**localStorage Best Practices:**

```typescript
// ✅ Use Method 2 (persisted utility) for multiple values
const theme = persisted({ key: "theme", initial: "light" });
const user = persisted({ key: "user", initial: null });

// ✅ Add error handling for corrupted data
try {
  const data = JSON.parse(localStorage.getItem(key) || "");
} catch {
  console.warn("Failed to parse storage, using defaults");
  return initialValue;
}

// ✅ Use custom serializers for complex types (Dates, Maps, Sets)
const lastVisit = persisted({
  key: "lastVisit",
  initial: new Date(),
  serializer: {
    parse: (text) => new Date(text),
    stringify: (date) => date.toISOString(),
  },
});
```

**File Organization for Stores:**

```
src/lib/
├── stores/
│   ├── theme.svelte.ts         # Theme store
│   ├── auth.svelte.ts          # Auth state
│   └── cart.svelte.ts          # Shopping cart
├── utils/
│   ├── persisted.ts            # localStorage helper
│   └── createSubscriber.ts     # Custom subscriber utils
└── classes/
    ├── UserManager.svelte.ts   # User management class
    └── DataCache.svelte.ts     # Caching class
```

### ⚠️ Common Mistakes

1. **Forgetting untrack() in effects** - causes infinite loops:

```typescript
// ❌ Bad - saves on every read, infinite loop
$effect(() => {
  const current = value;
  storage.setItem(key, JSON.stringify(current));
});

// ✅ Good - untrack prevents re-triggering
$effect(() => {
  const current = value;
  untrack(() => storage.setItem(key, JSON.stringify(current)));
});
```

2. **Not handling SSR** - localStorage doesn't exist on server:

```typescript
// ❌ Bad - crashes during SSR
const stored = localStorage.getItem(key);

// ✅ Good - checks for browser environment
const stored = typeof window !== "undefined" ? localStorage.getItem(key) : null;
```

3. **Overusing reactive classes** - simple state doesn't need classes:

```typescript
// ❌ Bad - overkill for simple counter
class Counter {
  count = $state(0);
  increment() {
    this.count++;
  }
}

// ✅ Good - simple state stays simple
let count = $state(0);
```

### ⚡ Performance Tips

- **$state.raw for large objects** - Skip deep reactivity when not needed
- **Batch localStorage writes** - Debounce saves to avoid excessive writes
- **Use $effect.tracking()** - Conditionally track dependencies for complex logic
- **Memoize with $derived** - Cache expensive computations
- **Lazy class instantiation** - Create instances only when needed

### 🧪 Testing Reactive State

```typescript
// Mock localStorage for tests
const mockStorage = new Map();
global.localStorage = {
  getItem: (key) => mockStorage.get(key),
  setItem: (key, value) => mockStorage.set(key, value),
  clear: () => mockStorage.clear(),
};

// Test reactive classes
test("UserManager loads users", async () => {
  const manager = new UserManager();
  await manager.loadUsers();
  expect(manager.users.length).toBeGreaterThan(0);
});
```

---

## 10. 🚀 End-of-Section Project: Universal Settings Dashboard

### Project Overview

Build a **complete settings dashboard** that demonstrates all universal reactivity patterns. This real-world project combines $state.raw, localStorage persistence, reactive classes, global state, and $effect.tracking().

**What You'll Build:**

- A **SettingsManager** reactive class with persistence
- **Theme system** with global state
- **User preferences** with localStorage
- **Analytics tracker** using $effect.tracking()
- A complete settings experience

### 📁 Files to Create

**1. Settings Manager** - `src/lib/stores/settings.svelte.ts`

```typescript
interface AppSettings {
  theme: "light" | "dark" | "system";
  language: string;
  notifications: {
    email: boolean;
    push: boolean;
    sms: boolean;
  };
  privacy: {
    analytics: boolean;
    personalization: boolean;
  };
  display: {
    fontSize: "small" | "medium" | "large";
    density: "compact" | "comfortable" | "spacious";
    animations: boolean;
  };
}

const defaultSettings: AppSettings = {
  theme: "dark",
  language: "en",
  notifications: { email: true, push: true, sms: false },
  privacy: { analytics: true, personalization: true },
  display: { fontSize: "medium", density: "comfortable", animations: true },
};

class SettingsManager {
  private storageKey = "app-settings";

  // Deep reactive state for settings
  settings = $state<AppSettings>(this.loadSettings());

  // Track unsaved changes
  hasChanges = $state(false);
  lastSaved = $state<Date | null>(null);

  // Derived computed values
  isDark = $derived(
    this.settings.theme === "dark" ||
      (this.settings.theme === "system" && this.prefersDark())
  );

  fontSizeClass = $derived(
    {
      small: "text-sm",
      medium: "text-base",
      large: "text-lg",
    }[this.settings.display.fontSize]
  );

  private prefersDark(): boolean {
    if (typeof window === "undefined") return true;
    return window.matchMedia("(prefers-color-scheme: dark)").matches;
  }

  private loadSettings(): AppSettings {
    if (typeof localStorage === "undefined") return defaultSettings;

    try {
      const stored = localStorage.getItem(this.storageKey);
      if (stored) {
        return { ...defaultSettings, ...JSON.parse(stored) };
      }
    } catch (e) {
      console.error("Failed to load settings:", e);
    }
    return defaultSettings;
  }

  saveSettings() {
    if (typeof localStorage === "undefined") return;

    try {
      localStorage.setItem(this.storageKey, JSON.stringify(this.settings));
      this.hasChanges = false;
      this.lastSaved = new Date();
    } catch (e) {
      console.error("Failed to save settings:", e);
    }
  }

  updateSetting<K extends keyof AppSettings>(key: K, value: AppSettings[K]) {
    this.settings[key] = value;
    this.hasChanges = true;
  }

  resetToDefaults() {
    this.settings = { ...defaultSettings };
    this.hasChanges = true;
  }

  exportSettings(): string {
    return JSON.stringify(this.settings, null, 2);
  }

  importSettings(json: string): boolean {
    try {
      const imported = JSON.parse(json);
      this.settings = { ...defaultSettings, ...imported };
      this.hasChanges = true;
      return true;
    } catch {
      return false;
    }
  }
}

export const settingsManager = new SettingsManager();
```

**2. Analytics Tracker** - `src/lib/services/analytics.svelte.ts`

```typescript
// Uses $effect.tracking() for conditional tracking

interface AnalyticsEvent {
  name: string;
  data: Record<string, unknown>;
  timestamp: Date;
}

class AnalyticsTracker {
  // Use $state.raw for large event log (no deep reactivity needed)
  events = $state.raw<AnalyticsEvent[]>([]);
  isEnabled = $state(true);

  eventCount = $derived(this.events.length);

  track(name: string, data: Record<string, unknown> = {}) {
    if (!this.isEnabled) return;

    // Only track if in reactive context or explicitly called
    const shouldTrack = $effect.tracking() ? true : true;

    if (shouldTrack) {
      // Must create new array for $state.raw
      this.events = [
        ...this.events,
        {
          name,
          data,
          timestamp: new Date(),
        },
      ];
    }
  }

  trackPageView(path: string) {
    this.track("page_view", { path });
  }

  trackSettingChange(setting: string, value: unknown) {
    this.track("setting_change", { setting, value });
  }

  clearEvents() {
    this.events = [];
  }

  getRecentEvents(count: number = 10): AnalyticsEvent[] {
    return this.events.slice(-count);
  }
}

export const analyticsTracker = new AnalyticsTracker();
```

**3. Settings Dashboard Page** - `src/routes/settings/+page.svelte`

```svelte
<script lang="ts">
	import { settingsManager } from '$lib/stores/settings.svelte';
	import { analyticsTracker } from '$lib/services/analytics.svelte';
	import { onMount } from 'svelte';

	let activeTab = $state<'general' | 'notifications' | 'privacy' | 'display'>('general');
	let showExportModal = $state(false);
	let importText = $state('');
	let importError = $state('');

	const tabs = [
		{ id: 'general', label: 'General', icon: '⚙️' },
		{ id: 'notifications', label: 'Notifications', icon: '🔔' },
		{ id: 'privacy', label: 'Privacy', icon: '🔒' },
		{ id: 'display', label: 'Display', icon: '🎨' }
	] as const;

	onMount(() => {
		analyticsTracker.trackPageView('/settings');
	});

	function handleSave() {
		settingsManager.saveSettings();
		analyticsTracker.track('settings_saved');
	}

	function handleImport() {
		if (settingsManager.importSettings(importText)) {
			importText = '';
			importError = '';
			analyticsTracker.track('settings_imported');
		} else {
			importError = 'Invalid JSON format';
		}
	}
</script>

<div class={`min-h-screen transition-colors ${settingsManager.isDark ? 'bg-gray-900 text-gray-100' : 'bg-gray-50 text-gray-900'}`}>
	<div class="max-w-4xl mx-auto p-6">
		<!-- Header -->
		<div class="flex justify-between items-center mb-8">
			<div>
				<h1 class="text-3xl font-bold">Settings</h1>
				{#if settingsManager.lastSaved}
					<p class={`text-sm mt-1 ${settingsManager.isDark ? 'text-gray-400' : 'text-gray-600'}`}>
						Last saved: {settingsManager.lastSaved.toLocaleTimeString()}
					</p>
				{/if}
			</div>
			<div class="flex gap-3">
				{#if settingsManager.hasChanges}
					<span class="px-3 py-1 bg-yellow-500/20 text-yellow-400 rounded-full text-sm font-medium">
						Unsaved changes
					</span>
				{/if}
				<button
					onclick={handleSave}
					disabled={!settingsManager.hasChanges}
					class={`px-6 py-2 rounded-lg font-semibold transition-all ${
						settingsManager.hasChanges
							? 'bg-blue-500 hover:bg-blue-400 text-white'
							: 'bg-gray-700 text-gray-500 cursor-not-allowed'
					}`}
				>
					Save Changes
				</button>
			</div>
		</div>

		<!-- Tabs -->
		<div class={`flex gap-2 mb-8 border-b ${settingsManager.isDark ? 'border-gray-700' : 'border-gray-200'}`}>
			{#each tabs as tab}
				<button
					onclick={() => {
						activeTab = tab.id;
						analyticsTracker.track('tab_change', { tab: tab.id });
					}}
					class={`px-4 py-3 font-medium transition-colors border-b-2 -mb-px ${
						activeTab === tab.id
							? 'border-blue-500 text-blue-500'
							: settingsManager.isDark
								? 'border-transparent text-gray-400 hover:text-gray-200'
								: 'border-transparent text-gray-600 hover:text-gray-900'
					}`}
				>
					{tab.icon} {tab.label}
				</button>
			{/each}
		</div>

		<!-- Tab Content -->
		<div class={`p-6 rounded-xl ${settingsManager.isDark ? 'bg-gray-800' : 'bg-white shadow-md'}`}>
			{#if activeTab === 'general'}
				<!-- Theme -->
				<div class="mb-8">
					<h3 class="text-lg font-semibold mb-4">Theme</h3>
					<div class="grid grid-cols-3 gap-4">
						{#each ['light', 'dark', 'system'] as theme}
							<button
								onclick={() => {
									settingsManager.updateSetting('theme', theme as 'light' | 'dark' | 'system');
									analyticsTracker.trackSettingChange('theme', theme);
								}}
								class={`p-4 rounded-lg border-2 transition-all capitalize ${
									settingsManager.settings.theme === theme
										? 'border-blue-500 bg-blue-500/10'
										: settingsManager.isDark
											? 'border-gray-700 hover:border-gray-600'
											: 'border-gray-200 hover:border-gray-300'
								}`}
							>
								{theme === 'light' ? '☀️' : theme === 'dark' ? '🌙' : '💻'} {theme}
							</button>
						{/each}
					</div>
				</div>

				<!-- Language -->
				<div class="mb-8">
					<h3 class="text-lg font-semibold mb-4">Language</h3>
					<select
						value={settingsManager.settings.language}
						onchange={(e) => {
							settingsManager.updateSetting('language', e.currentTarget.value);
							analyticsTracker.trackSettingChange('language', e.currentTarget.value);
						}}
						class={`w-full p-3 rounded-lg border ${
							settingsManager.isDark
								? 'bg-gray-700 border-gray-600 text-white'
								: 'bg-white border-gray-300 text-gray-900'
						}`}
					>
						<option value="en">English</option>
						<option value="es">Español</option>
						<option value="fr">Français</option>
						<option value="de">Deutsch</option>
						<option value="ja">日本語</option>
					</select>
				</div>

			{:else if activeTab === 'notifications'}
				<h3 class="text-lg font-semibold mb-6">Notification Preferences</h3>
				{#each Object.entries(settingsManager.settings.notifications) as [key, value]}
					<label class={`flex items-center justify-between p-4 rounded-lg mb-3 ${settingsManager.isDark ? 'bg-gray-700' : 'bg-gray-100'}`}>
						<span class="font-medium capitalize">{key} Notifications</span>
						<button
							onclick={() => {
								settingsManager.settings.notifications[key as keyof typeof settingsManager.settings.notifications] = !value;
								settingsManager.hasChanges = true;
								analyticsTracker.trackSettingChange(`notifications.${key}`, !value);
							}}
							class={`w-12 h-6 rounded-full transition-colors relative ${value ? 'bg-blue-500' : 'bg-gray-500'}`}
						>
							<span class={`absolute top-1 w-4 h-4 rounded-full bg-white transition-transform ${value ? 'left-7' : 'left-1'}`}></span>
						</button>
					</label>
				{/each}

			{:else if activeTab === 'privacy'}
				<h3 class="text-lg font-semibold mb-6">Privacy Settings</h3>
				{#each Object.entries(settingsManager.settings.privacy) as [key, value]}
					<label class={`flex items-center justify-between p-4 rounded-lg mb-3 ${settingsManager.isDark ? 'bg-gray-700' : 'bg-gray-100'}`}>
						<div>
							<span class="font-medium capitalize">{key}</span>
							<p class={`text-sm ${settingsManager.isDark ? 'text-gray-400' : 'text-gray-600'}`}>
								{key === 'analytics' ? 'Help improve the app with usage data' : 'Get personalized recommendations'}
							</p>
						</div>
						<button
							onclick={() => {
								settingsManager.settings.privacy[key as keyof typeof settingsManager.settings.privacy] = !value;
								settingsManager.hasChanges = true;
								analyticsTracker.trackSettingChange(`privacy.${key}`, !value);
							}}
							class={`w-12 h-6 rounded-full transition-colors relative ${value ? 'bg-blue-500' : 'bg-gray-500'}`}
						>
							<span class={`absolute top-1 w-4 h-4 rounded-full bg-white transition-transform ${value ? 'left-7' : 'left-1'}`}></span>
						</button>
					</label>
				{/each}

			{:else if activeTab === 'display'}
				<h3 class="text-lg font-semibold mb-6">Display Options</h3>

				<!-- Font Size -->
				<div class="mb-6">
					<label class="block font-medium mb-3">Font Size</label>
					<div class="flex gap-3">
						{#each ['small', 'medium', 'large'] as size}
							<button
								onclick={() => {
									settingsManager.settings.display.fontSize = size as 'small' | 'medium' | 'large';
									settingsManager.hasChanges = true;
									analyticsTracker.trackSettingChange('fontSize', size);
								}}
								class={`flex-1 p-3 rounded-lg border-2 capitalize transition-all ${
									settingsManager.settings.display.fontSize === size
										? 'border-blue-500 bg-blue-500/10'
										: settingsManager.isDark
											? 'border-gray-700 hover:border-gray-600'
											: 'border-gray-200 hover:border-gray-300'
								}`}
							>
								<span class={size === 'small' ? 'text-sm' : size === 'large' ? 'text-lg' : 'text-base'}>{size}</span>
							</button>
						{/each}
					</div>
				</div>

				<!-- Animations -->
				<label class={`flex items-center justify-between p-4 rounded-lg ${settingsManager.isDark ? 'bg-gray-700' : 'bg-gray-100'}`}>
					<div>
						<span class="font-medium">Animations</span>
						<p class={`text-sm ${settingsManager.isDark ? 'text-gray-400' : 'text-gray-600'}`}>
							Enable smooth transitions and animations
						</p>
					</div>
					<button
						onclick={() => {
							settingsManager.settings.display.animations = !settingsManager.settings.display.animations;
							settingsManager.hasChanges = true;
							analyticsTracker.trackSettingChange('animations', settingsManager.settings.display.animations);
						}}
						class={`w-12 h-6 rounded-full transition-colors relative ${settingsManager.settings.display.animations ? 'bg-blue-500' : 'bg-gray-500'}`}
					>
						<span class={`absolute top-1 w-4 h-4 rounded-full bg-white transition-transform ${settingsManager.settings.display.animations ? 'left-7' : 'left-1'}`}></span>
					</button>
				</label>
			{/if}
		</div>

		<!-- Actions -->
		<div class={`mt-8 p-6 rounded-xl ${settingsManager.isDark ? 'bg-gray-800' : 'bg-white shadow-md'}`}>
			<h3 class="text-lg font-semibold mb-4">Data Management</h3>
			<div class="flex flex-wrap gap-4">
				<button
					onclick={() => (showExportModal = true)}
					class={`px-4 py-2 rounded-lg border transition-colors ${
						settingsManager.isDark
							? 'border-gray-600 hover:bg-gray-700'
							: 'border-gray-300 hover:bg-gray-100'
					}`}
				>
					📤 Export Settings
				</button>
				<button
					onclick={() => settingsManager.resetToDefaults()}
					class="px-4 py-2 rounded-lg border border-red-500 text-red-500 hover:bg-red-500/10 transition-colors"
				>
					🔄 Reset to Defaults
				</button>
			</div>
		</div>

		<!-- Analytics Preview -->
		{#if settingsManager.settings.privacy.analytics}
			<div class={`mt-8 p-6 rounded-xl ${settingsManager.isDark ? 'bg-gray-800' : 'bg-white shadow-md'}`}>
				<h3 class="text-lg font-semibold mb-4">Recent Activity ({analyticsTracker.eventCount} events)</h3>
				<div class="space-y-2 max-h-48 overflow-y-auto">
					{#each analyticsTracker.getRecentEvents() as event}
						<div class={`p-3 rounded-lg text-sm ${settingsManager.isDark ? 'bg-gray-700' : 'bg-gray-100'}`}>
							<span class="font-medium">{event.name}</span>
							<span class={settingsManager.isDark ? 'text-gray-400' : 'text-gray-600'}> - {event.timestamp.toLocaleTimeString()}</span>
						</div>
					{/each}
				</div>
			</div>
		{/if}
	</div>

	<!-- Export Modal -->
	{#if showExportModal}
		<div class="fixed inset-0 bg-black/50 flex items-center justify-center z-50 p-4">
			<div class={`w-full max-w-lg rounded-xl p-6 ${settingsManager.isDark ? 'bg-gray-800' : 'bg-white'}`}>
				<h3 class="text-xl font-bold mb-4">Export / Import Settings</h3>

				<div class="mb-4">
					<label class="block font-medium mb-2">Current Settings (JSON)</label>
					<textarea
						readonly
						value={settingsManager.exportSettings()}
						class={`w-full h-32 p-3 rounded-lg font-mono text-sm ${
							settingsManager.isDark
								? 'bg-gray-700 border-gray-600'
								: 'bg-gray-100 border-gray-300'
						}`}
					></textarea>
				</div>

				<div class="mb-4">
					<label class="block font-medium mb-2">Import Settings</label>
					<textarea
						bind:value={importText}
						placeholder="Paste JSON here..."
						class={`w-full h-32 p-3 rounded-lg font-mono text-sm ${
							settingsManager.isDark
								? 'bg-gray-700 border-gray-600'
								: 'bg-gray-100 border-gray-300'
						}`}
					></textarea>
					{#if importError}
						<p class="text-red-500 text-sm mt-1">{importError}</p>
					{/if}
				</div>

				<div class="flex gap-3 justify-end">
					<button
						onclick={() => (showExportModal = false)}
						class={`px-4 py-2 rounded-lg ${settingsManager.isDark ? 'hover:bg-gray-700' : 'hover:bg-gray-100'}`}
					>
						Close
					</button>
					<button
						onclick={handleImport}
						disabled={!importText}
						class="px-4 py-2 bg-blue-500 text-white rounded-lg hover:bg-blue-400 disabled:opacity-50"
					>
						Import
					</button>
				</div>
			</div>
		</div>
	{/if}
</div>
```

### What This Project Demonstrates

| Concept                      | Implementation                              |
| ---------------------------- | ------------------------------------------- |
| **Reactive Classes**         | SettingsManager, AnalyticsTracker           |
| **$state.raw**               | Analytics events array (no deep reactivity) |
| **$derived**                 | isDark, fontSizeClass                       |
| **localStorage Persistence** | loadSettings(), saveSettings()              |
| **$effect.tracking()**       | Conditional analytics tracking              |
| **Global State**             | Singleton settingsManager instance          |
| **Generic Helpers**          | Reusable persistence patterns               |

---

## 🚀 Next Steps

You've completed the fundamentals! You're now ready for:

- **Section 6**: Context API and advanced patterns
- **Section 7**: Actions and special elements
- **Section 8**: Animations and transitions
- **SvelteKit**: Full-stack applications

Congratulations on mastering Svelte 5! 🎉
