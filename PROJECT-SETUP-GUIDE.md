# 🚀 Project Setup Guide

Complete setup instructions for following along with the Svelte 5 Context API course.

---

## Initial Setup

This project is already scaffolded with SvelteKit! But if you're starting fresh:

```bash
npm create svelte@latest my-svelte-app
cd my-svelte-app
npm install
npm run dev
```

---

## 📁 Folder Structure Best Practices

```
src/
├── lib/
│   ├── components/       # Reusable UI components
│   │   ├── Modal.svelte
│   │   ├── ProfileCard.svelte
│   │   └── DataTable.svelte
│   │
│   ├── stores/           # Global reactive state (.svelte.ts)
│   │   ├── theme.svelte.ts
│   │   └── auth.svelte.ts
│   │
│   ├── actions/          # Reusable DOM behaviors
│   │   ├── tooltip.ts
│   │   ├── longpress.ts
│   │   └── clickOutside.ts
│   │
│   ├── services/         # API clients and business logic
│   │   ├── currencyAPI.ts
│   │   └── mockAPI.ts
│   │
│   ├── utils/            # Helper functions and utilities
│   │   ├── validation.svelte.ts
│   │   ├── formatters.ts
│   │   └── constants.ts
│   │
│   ├── classes/          # Reactive class definitions (.svelte.ts)
│   │   ├── Cart.svelte.ts
│   │   └── FormValidator.svelte.ts
│   │
│   ├── transitions/      # Custom animation functions
│   │   └── custom.ts
│   │
│   └── types/            # Shared TypeScript interfaces
│       ├── kanban.ts
│       └── user.ts
│
└── routes/               # Pages and routes
    ├── +layout.svelte
    ├── +page.svelte
    ├── todo-comparison/
    │   └── +page.svelte
    └── shopping-cart/
        └── +page.svelte
```

---

## 🎯 File Naming Conventions

### **Components**: `PascalCase.svelte`
✅ `Modal.svelte`, `ProfileCard.svelte`, `TodoItem.svelte`

### **Routes**: `kebab-case/+page.svelte`
✅ `shopping-cart/+page.svelte`, `user-profile/+page.svelte`

### **Utilities**: `camelCase.ts` or `.svelte.ts`
✅ `validation.svelte.ts`, `formatters.ts`, `constants.ts`

### **Actions**: `camelCase.ts`
✅ `tooltip.ts`, `clickOutside.ts`, `longpress.ts`

### **Types**: `PascalCase.ts` or `camelCase.ts`
✅ `User.ts` or `types.ts`

### **Services**: `camelCase.ts`
✅ `currencyAPI.ts`, `authService.ts`

---

## 🔧 Important Extensions

### **When to use `.svelte.ts`**

Use `.svelte.ts` for TypeScript files that use Svelte 5 runes:

```typescript
// ✅ theme.svelte.ts (uses $state)
let currentTheme = $state('dark');

// ✅ Cart.svelte.ts (uses $state and $derived)
export class Cart {
  items = $state([]);
  total = $derived(this.items.reduce(...));
}
```

### **When to use `.ts`**

Use regular `.ts` for files without runes:

```typescript
// ✅ formatters.ts (pure functions, no runes)
export function formatCurrency(amount: number) {
  return `$${amount.toFixed(2)}`;
}

// ✅ constants.ts (just data)
export const API_URL = 'https://api.example.com';
```

---

## 📦 Essential Utilities to Create

### 1. **Formatters** (`src/lib/utils/formatters.ts`)

Common formatting functions used across the app:

```typescript
// src/lib/utils/formatters.ts

export function formatCurrency(amount: number, currency = 'USD'): string {
	return new Intl.NumberFormat('en-US', {
		style: 'currency',
		currency
	}).format(amount);
}

export function formatDate(date: Date): string {
	return new Intl.DateTimeFormat('en-US', {
		year: 'numeric',
		month: 'long',
		day: 'numeric'
	}).format(date);
}

export function formatNumber(num: number): string {
	return new Intl.NumberFormat('en-US').format(num);
}

export function truncate(str: string, length: number): string {
	return str.length > length ? str.slice(0, length) + '...' : str;
}

export function capitalize(str: string): string {
	return str.charAt(0).toUpperCase() + str.slice(1);
}
```

### 2. **Constants** (`src/lib/utils/constants.ts`)

App-wide constants:

```typescript
// src/lib/utils/constants.ts

export const APP_NAME = 'My Svelte App';

export const COLORS = {
	primary: '#4a9eff',
	success: '#4ade80',
	warning: '#ffa500',
	error: '#ff6b6b',
	info: '#4a9eff'
} as const;

export const BREAKPOINTS = {
	mobile: 640,
	tablet: 768,
	desktop: 1024,
	wide: 1280
} as const;

export const API_ENDPOINTS = {
	users: '/api/users',
	posts: '/api/posts',
	comments: '/api/comments'
} as const;
```

### 3. **Type Definitions** (`src/lib/types/`)

Shared interfaces across your app:

```typescript
// src/lib/types/user.ts

export interface User {
	id: string;
	name: string;
	email: string;
	role: 'admin' | 'user' | 'moderator';
	avatar?: string;
	createdAt: Date;
}

export interface AuthUser extends User {
	token: string;
	permissions: string[];
}
```

```typescript
// src/lib/types/common.ts

export type Status = 'idle' | 'loading' | 'success' | 'error';

export interface ApiResponse<T> {
	data: T;
	message?: string;
	error?: string;
}

export type DeepPartial<T> = {
	[P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```

### 4. **Validation Utilities** (`src/lib/utils/validators.ts`)

Reusable validation functions:

```typescript
// src/lib/utils/validators.ts

export function isEmail(email: string): boolean {
	const regex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
	return regex.test(email);
}

export function isStrongPassword(password: string): boolean {
	// At least 8 chars, 1 uppercase, 1 lowercase, 1 number
	return (
		password.length >= 8 &&
		/[A-Z]/.test(password) &&
		/[a-z]/.test(password) &&
		/[0-9]/.test(password)
	);
}

export function isValidURL(url: string): boolean {
	try {
		new URL(url);
		return true;
	} catch {
		return false;
	}
}

export function isInRange(value: number, min: number, max: number): boolean {
	return value >= min && value <= max;
}

export function isNotEmpty(value: string): boolean {
	return value.trim().length > 0;
}
```

### 5. **API Client Base** (`src/lib/services/apiClient.ts`)

Reusable fetch wrapper:

```typescript
// src/lib/services/apiClient.ts

interface RequestOptions extends RequestInit {
	params?: Record<string, string>;
}

class APIClient {
	private baseURL: string;

	constructor(baseURL: string) {
		this.baseURL = baseURL;
	}

	private async request<T>(endpoint: string, options: RequestOptions = {}): Promise<T> {
		const { params, ...fetchOptions } = options;

		let url = `${this.baseURL}${endpoint}`;

		if (params) {
			const searchParams = new URLSearchParams(params);
			url += `?${searchParams}`;
		}

		const response = await fetch(url, {
			...fetchOptions,
			headers: {
				'Content-Type': 'application/json',
				...fetchOptions.headers
			}
		});

		if (!response.ok) {
			throw new Error(`API Error: ${response.statusText}`);
		}

		return response.json();
	}

	async get<T>(endpoint: string, params?: Record<string, string>): Promise<T> {
		return this.request<T>(endpoint, { method: 'GET', params });
	}

	async post<T>(endpoint: string, data: unknown): Promise<T> {
		return this.request<T>(endpoint, {
			method: 'POST',
			body: JSON.stringify(data)
		});
	}

	async put<T>(endpoint: string, data: unknown): Promise<T> {
		return this.request<T>(endpoint, {
			method: 'PUT',
			body: JSON.stringify(data)
		});
	}

	async delete<T>(endpoint: string): Promise<T> {
		return this.request<T>(endpoint, { method: 'DELETE' });
	}
}

export const apiClient = new APIClient('https://api.example.com');
```

### 6. **Local Storage Helper** (`src/lib/utils/storage.ts`)

Type-safe localStorage wrapper:

```typescript
// src/lib/utils/storage.ts

export const storage = {
	get<T>(key: string, defaultValue: T): T {
		if (typeof window === 'undefined') return defaultValue;

		try {
			const item = window.localStorage.getItem(key);
			return item ? JSON.parse(item) : defaultValue;
		} catch {
			return defaultValue;
		}
	},

	set<T>(key: string, value: T): void {
		if (typeof window === 'undefined') return;

		try {
			window.localStorage.setItem(key, JSON.stringify(value));
		} catch (error) {
			console.error('Failed to save to localStorage:', error);
		}
	},

	remove(key: string): void {
		if (typeof window === 'undefined') return;
		window.localStorage.removeItem(key);
	},

	clear(): void {
		if (typeof window === 'undefined') return;
		window.localStorage.clear();
	}
};
```

---

## 🎨 Global Styles Setup

### **Create** `src/app.css` (if not exists):

```css
/* src/app.css */

@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom base styles */
* {
	margin: 0;
	padding: 0;
	box-sizing: border-box;
}

body {
	font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell,
		sans-serif;
	-webkit-font-smoothing: antialiased;
	-moz-osx-font-smoothing: grayscale;
}

/* Custom scrollbar */
::-webkit-scrollbar {
	width: 8px;
	height: 8px;
}

::-webkit-scrollbar-track {
	background: #1a1a1a;
}

::-webkit-scrollbar-thumb {
	background: #3a3a3a;
	border-radius: 4px;
}

::-webkit-scrollbar-thumb:hover {
	background: #4a4a4a;
}
```

### **Import in** `src/routes/+layout.svelte`:

```svelte
<script lang="ts">
	import '../app.css';
</script>

<slot />
```

---

## 📝 Import Conventions

### **Alias Imports** (Already configured in SvelteKit)

```typescript
// ✅ Use $lib alias
import { Modal } from '$lib/components/Modal.svelte';
import { formatCurrency } from '$lib/utils/formatters';
import { themeStore } from '$lib/stores/theme.svelte';

// ❌ Don't use relative paths for lib
import { Modal } from '../../lib/components/Modal.svelte';
```

### **Import Order** (Recommended)

```typescript
// 1. Svelte imports
import { onMount } from 'svelte';

// 2. External libraries
import tippy from 'tippy.js';

// 3. Internal components
import Modal from '$lib/components/Modal.svelte';

// 4. Internal utilities
import { formatCurrency } from '$lib/utils/formatters';
import { isEmail } from '$lib/utils/validators';

// 5. Types
import type { User } from '$lib/types/user';

// 6. Styles (if any)
import './styles.css';
```

---

## 🚦 TypeScript Configuration

Your `tsconfig.json` should have these paths configured:

```json
{
	"compilerOptions": {
		"paths": {
			"$lib": ["./src/lib"],
			"$lib/*": ["./src/lib/*"]
		}
	}
}
```

---

## 📦 Recommended NPM Packages

### **For Sections 6-8**

```bash
# Tooltips (Section 7)
npm install tippy.js

# Date utilities (optional)
npm install date-fns

# Icons (optional)
npm install lucide-svelte
```

---

## ✅ Quick Start Checklist

- [ ] Project initialized with `npm create svelte@latest`
- [ ] Created folder structure (`lib/components`, `lib/utils`, etc.)
- [ ] Created `formatters.ts` utility file
- [ ] Created `constants.ts` for app constants
- [ ] Created `validators.ts` for validation logic
- [ ] Created `storage.ts` for localStorage helpers
- [ ] Set up `app.css` with global styles
- [ ] Imported styles in `+layout.svelte`
- [ ] Verified `$lib` alias works in imports

---

## 🎓 Learning Path

Follow sections in order:

1. **Section 1**: Fundamentals & Reactivity
2. **Section 2**: Components & Styling
3. **Section 3**: Deep State & Data
4. **Section 4**: Advanced State Patterns
5. **Section 5**: Universal Reactivity
6. **Section 6**: Context API
7. **Section 7**: Actions & Special Elements
8. **Section 8**: Animations & Transitions

Each section builds on previous concepts!

---

## 🤝 Best Practices Summary

### **Component Design**
- Keep components small and focused
- Use props for input, events for output
- Extract reusable logic to utilities

### **State Management**
- Use `$state` in components for local state
- Use `.svelte.ts` files for shared state
- Keep state as close to where it's used as possible

### **File Organization**
- One component per file
- Group related files in folders
- Use consistent naming conventions

### **Performance**
- Prefer `$derived` over `$effect` for computed values
- Keep effects minimal and focused
- Use `{#key}` blocks to force re-renders when needed

---

## 🐛 Debugging Tips

1. **Use `$inspect()`** to log state changes
2. **Browser DevTools** - check the console
3. **Svelte DevTools** - browser extension for inspecting components
4. **`console.log()` is your friend** - don't be afraid to use it!

---

## 📚 Additional Resources

- [Svelte 5 Docs](https://svelte.dev/docs)
- [SvelteKit Docs](https://kit.svelte.dev/docs)
- [Svelte Discord](https://svelte.dev/chat)
- [Svelte GitHub](https://github.com/sveltejs/svelte)

---

Happy coding! 🚀
