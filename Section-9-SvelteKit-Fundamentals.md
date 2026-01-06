# Section 9: SvelteKit Fundamentals

## 📚 Learning Objectives

By the end of this section, you will:

- Understand what SvelteKit is and how it differs from vanilla Svelte
- Set up a production-ready SvelteKit project with Tailwind 4 & DaisyUI
- Master the SvelteKit folder structure and module resolution
- Configure VSCode for optimal SvelteKit development
- Understand file-based routing and the `$lib` alias

---

## Table of Contents

- [Section 9: SvelteKit Fundamentals](#section-9-sveltekit-fundamentals)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Introduction to SvelteKit](#1-introduction-to-sveltekit)
    - [What is SvelteKit?](#what-is-sveltekit)
  - [2. Creating a New SvelteKit Application with Tailwind \& DaisyUI](#2-creating-a-new-sveltekit-application-with-tailwind--daisyui)
    - [Project Setup from Scratch](#project-setup-from-scratch)
    - [📁 Step-by-Step Setup](#-step-by-step-setup)
  - [3. Folder Structure Overview, $lib Alias and Custom Aliases](#3-folder-structure-overview-lib-alias-and-custom-aliases)
    - [Understanding SvelteKit's Folder Structure](#understanding-sveltekits-folder-structure)
    - [📁 Complete Folder Structure](#-complete-folder-structure)
    - [The `$lib` Alias](#the-lib-alias)
    - [Custom Aliases](#custom-aliases)
    - [Practical Example: Barrel Exports](#practical-example-barrel-exports)
    - [Demo Page: Testing Folder Structure](#demo-page-testing-folder-structure)
  - [4. VSCode Extensions \& Settings](#4-vscode-extensions--settings)
    - [Essential VSCode Configuration](#essential-vscode-configuration)
    - [📦 Required Extensions](#-required-extensions)
    - [Install All at Once](#install-all-at-once)
    - [📝 VSCode Settings](#-vscode-settings)
    - [🎨 Prettier Configuration](#-prettier-configuration)
    - [🔍 ESLint Configuration](#-eslint-configuration)
    - [⌨️ Useful Keyboard Shortcuts](#️-useful-keyboard-shortcuts)
    - [🧪 Testing Extensions (Optional)](#-testing-extensions-optional)
    - [Demo: Verify Setup](#demo-verify-setup)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Introduction to SvelteKit

### What is SvelteKit?

SvelteKit is the official **full-stack framework** for building Svelte applications. Think of it as Next.js for React, but with less boilerplate and better performance!

**Real-World Scenario:** You're building a modern SaaS application with user authentication, API routes, server-side rendering, and multiple pages. SvelteKit provides all of this out of the box.

**What it does:** Provides routing, SSR, SSG, API endpoints, and deployment tooling.

**Key Features:**

- 🎯 **File-based routing**: No router config needed
- ⚡ **Server-Side Rendering (SSR)**: Better SEO and initial load
- 🔄 **Automatic code splitting**: Only load what you need
- 🌐 **API routes**: Build full-stack apps in one codebase
- 📦 **Adapters**: Deploy anywhere (Vercel, Netlify, Node, etc.)
- 🎨 **Layouts**: Share UI between pages effortlessly

**SvelteKit vs Vanilla Svelte:**

| Feature        | Svelte                    | SvelteKit                |
| -------------- | ------------------------- | ------------------------ |
| Routing        | Manual (client-side only) | File-based, SSR-ready    |
| SSR            | Not included              | Built-in                 |
| API Routes     | Need separate backend     | Built-in                 |
| Code Splitting | Manual                    | Automatic                |
| SEO            | Requires extra work       | Excellent out of the box |
| Deployment     | SPA only                  | Full-stack deployment    |

> 💡 **Best Practice**: Always use SvelteKit for production apps. Even if you're building a simple SPA, SvelteKit provides better DX and performance with minimal overhead.

**⚠️ Common Mistakes:**

- Don't use vanilla Svelte for multi-page apps - you'll rebuild what SvelteKit provides
- Don't skip SSR unless you have a specific reason (admin panels, etc.)
- Don't confuse Svelte (the compiler) with SvelteKit (the framework)

**When to use SvelteKit:**

- ✅ Multi-page websites
- ✅ Web applications with routing
- ✅ Projects needing SEO
- ✅ Full-stack applications
- ✅ Projects requiring API endpoints

**When NOT to use SvelteKit:**

- ❌ Embedded widgets
- ❌ Browser extensions
- ❌ Learning Svelte basics (start with vanilla Svelte)

---

## 2. Creating a New SvelteKit Application with Tailwind & DaisyUI

### Project Setup from Scratch

Let's create a production-ready SvelteKit app with modern tooling!

**Real-World Scenario:** You're starting a new project and want Tailwind 4 for utility classes and DaisyUI for pre-built components.

**What it does:** Sets up a complete SvelteKit project with TypeScript, Tailwind 4, and DaisyUI.

### 📁 Step-by-Step Setup

**1. Create SvelteKit Project**

```bash
# Create a new SvelteKit project
npm create svelte@latest my-sveltekit-app

# During the wizard, select:
# ✓ Skeleton project (or SvelteKit demo app)
# ✓ Yes, using TypeScript syntax
# ✓ Add ESLint for code linting
# ✓ Add Prettier for code formatting
# ✓ Add Playwright for browser testing (optional)
# ✓ Add Vitest for unit testing (optional)

cd my-sveltekit-app
npm install
```

**2. Install Tailwind CSS 4**

```bash
# Install Tailwind CSS
npm install -D tailwindcss@next @tailwindcss/vite@next
npx tailwindcss init
```

**3. Configure Tailwind (vite.config.ts)**

```typescript
import { sveltekit } from "@sveltejs/kit/vite";
import { defineConfig } from "vite";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [sveltekit(), tailwindcss()],
});
```

**4. Install DaisyUI**

```bash
npm install -D daisyui@latest
```

**5. Configure Tailwind with DaisyUI (tailwind.config.js)**

```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: ["./src/**/*.{html,js,svelte,ts}"],
  theme: {
    extend: {},
  },
  plugins: [require("daisyui")],
  daisyui: {
    themes: ["light", "dark", "cupcake", "cyberpunk"], // Add your favorite themes
    darkTheme: "dark", // name of one of the included themes
    base: true, // applies background color and foreground color
    styled: true, // include daisyUI colors and design decisions
    utils: true, // adds responsive and modifier utility classes
    prefix: "", // prefix for daisyUI classnames (components, modifiers and responsive class names)
    logs: true, // Shows info about daisyUI version and used config
    themeRoot: ":root", // The element that receives theme color CSS variables
  },
};
```

**6. Create Global CSS (src/app.css)**

```css
@import "tailwindcss";
```

**7. Import CSS in Layout (src/routes/+layout.svelte)**

```svelte
<script lang="ts">
	import '../app.css';
	import type { Snippet } from 'svelte';

	let { children }: { children: Snippet } = $props();
</script>

{@render children()}
```

**8. Test the Setup (src/routes/+page.svelte)**

```svelte
<script lang="ts">
	let count = $state(0);
</script>

<div class="min-h-screen bg-base-200 flex items-center justify-center p-8">
	<div class="card w-96 bg-base-100 shadow-xl">
		<div class="card-body items-center text-center">
			<h2 class="card-title text-4xl">🎉 SvelteKit + DaisyUI</h2>
			<p class="text-base-content/70 mb-4">Click the button to increment the counter!</p>

			<div class="stat bg-primary text-primary-content rounded-lg">
				<div class="stat-title text-primary-content/70">Count</div>
				<div class="stat-value">{count}</div>
				<div class="stat-desc text-primary-content/70">Times clicked</div>
			</div>

			<div class="card-actions mt-4">
				<button class="btn btn-primary" onclick={() => count++}> Increment </button>
				<button class="btn btn-secondary" onclick={() => (count = 0)}> Reset </button>
			</div>
		</div>
	</div>
</div>
```

**9. Run the Development Server**

```bash
npm run dev -- --open
```

**Key Concepts:**

- **Tailwind 4**: Latest version with improved performance
- **DaisyUI**: Component library built on Tailwind
- **TypeScript**: Type safety for better DX
- **Vite**: Lightning-fast dev server

> 💡 **Best Practice**: Use DaisyUI for common UI patterns (buttons, cards, modals) and Tailwind utilities for custom styling. This gives you speed AND flexibility.

**⚠️ Common Mistakes:**

- Don't forget to import `app.css` in your root layout
- Don't skip the `@import 'tailwindcss'` directive in app.css
- Don't use both `class` and `className` (Svelte uses `class`)

**⚡ Performance Tips:**

- DaisyUI only includes CSS, no JS overhead
- Tailwind 4's new engine is significantly faster
- Use `npm run build` to see production bundle sizes

**DaisyUI Themes Preview:**

Create a theme switcher component:

```svelte
<!-- src/lib/components/ThemeSwitcher.svelte -->
<script lang="ts">
	const themes = ['light', 'dark', 'cupcake', 'cyberpunk'];
	let currentTheme = $state('dark');

	function changeTheme(theme: string) {
		currentTheme = theme;
		document.documentElement.setAttribute('data-theme', theme);
	}
</script>

<div class="dropdown dropdown-end">
	<div tabindex="0" role="button" class="btn btn-ghost gap-1 normal-case">
		<svg
			width="20"
			height="20"
			xmlns="http://www.w3.org/2000/svg"
			fill="none"
			viewBox="0 0 24 24"
			class="inline-block h-5 w-5 stroke-current"
		>
			<path
				stroke-linecap="round"
				stroke-linejoin="round"
				stroke-width="2"
				d="M7 21a4 4 0 01-4-4V5a2 2 0 012-2h4a2 2 0 012 2v12a4 4 0 01-4 4zm0 0h12a2 2 0 002-2v-4a2 2 0 00-2-2h-2.343M11 7.343l1.657-1.657a2 2 0 012.828 0l2.829 2.829a2 2 0 010 2.828l-8.486 8.485M7 17h.01"
			></path>
		</svg>
		<span class="hidden md:inline">Theme</span>
	</div>
	<ul class="dropdown-content menu bg-base-200 rounded-box z-50 w-52 p-2 shadow-2xl">
		{#each themes as theme}
			<li>
				<button
					class:active={currentTheme === theme}
					onclick={() => changeTheme(theme)}
					class="capitalize"
				>
					{theme}
				</button>
			</li>
		{/each}
	</ul>
</div>
```

---

## 3. Folder Structure Overview, $lib Alias and Custom Aliases

### Understanding SvelteKit's Folder Structure

SvelteKit uses a **convention-over-configuration** approach where folder structure determines routing!

**Real-World Scenario:** You're organizing a large application with shared components, utilities, and multiple routes.

**What it does:** Explains SvelteKit's folder structure and module resolution.

### 📁 Complete Folder Structure

```
my-sveltekit-app/
├── src/
│   ├── lib/                    # Reusable code (aliased as $lib)
│   │   ├── components/         # Shared components
│   │   │   ├── ui/            # UI components
│   │   │   │   ├── Button.svelte
│   │   │   │   ├── Card.svelte
│   │   │   │   └── Modal.svelte
│   │   │   ├── layout/        # Layout components
│   │   │   │   ├── Header.svelte
│   │   │   │   ├── Footer.svelte
│   │   │   │   └── Sidebar.svelte
│   │   │   └── features/      # Feature-specific components
│   │   │       ├── auth/
│   │   │       └── blog/
│   │   ├── server/            # Server-only code
│   │   │   ├── database.ts
│   │   │   └── auth.ts
│   │   ├── stores/            # Svelte stores
│   │   │   ├── user.ts
│   │   │   └── theme.ts
│   │   ├── utils/             # Utility functions
│   │   │   ├── formatters.ts
│   │   │   └── validators.ts
│   │   ├── types/             # TypeScript types
│   │   │   ├── user.ts
│   │   │   └── api.ts
│   │   └── index.ts           # Barrel exports
│   ├── routes/                # File-based routing
│   │   ├── +layout.svelte     # Root layout (wraps all pages)
│   │   ├── +layout.ts         # Root layout data
│   │   ├── +page.svelte       # Home page (/)
│   │   ├── +page.ts           # Home page data
│   │   ├── about/
│   │   │   └── +page.svelte   # About page (/about)
│   │   ├── blog/
│   │   │   ├── +page.svelte   # Blog list (/blog)
│   │   │   └── [slug]/
│   │   │       └── +page.svelte # Blog post (/blog/my-post)
│   │   └── api/               # API routes
│   │       └── posts/
│   │           └── +server.ts  # API endpoint
│   ├── app.html               # HTML template
│   ├── app.css                # Global styles
│   └── app.d.ts               # Type definitions
├── static/                     # Static assets (served from /)
│   ├── favicon.png
│   ├── robots.txt
│   └── images/
├── tests/                      # Test files
├── .env                        # Environment variables
├── svelte.config.js            # SvelteKit configuration
├── vite.config.ts              # Vite configuration
├── tailwind.config.js          # Tailwind configuration
└── package.json
```

### The `$lib` Alias

The `$lib` alias is a **magic import path** that always points to `src/lib/`.

**Example Usage:**

```typescript
// ❌ Bad: Relative imports get messy
import Button from "../../../lib/components/ui/Button.svelte";

// ✅ Good: Clean $lib imports
import Button from "$lib/components/ui/Button.svelte";
import { formatDate } from "$lib/utils/formatters";
import type { User } from "$lib/types/user";
```

### Custom Aliases

You can add custom aliases in `svelte.config.js`:

```javascript
import adapter from "@sveltejs/adapter-auto";
import { vitePreprocess } from "@sveltejs/vite-plugin-svelte";

/** @type {import('@sveltejs/kit').Config} */
const config = {
  preprocess: vitePreprocess(),
  kit: {
    adapter: adapter(),
    alias: {
      $lib: "src/lib",
      $components: "src/lib/components",
      $utils: "src/lib/utils",
      $types: "src/lib/types",
      $stores: "src/lib/stores",
    },
  },
};

export default config;
```

**Now you can use:**

```typescript
import Button from "$components/ui/Button.svelte";
import { formatDate } from "$utils/formatters";
import type { User } from "$types/user";
```

### Practical Example: Barrel Exports

Create `src/lib/index.ts` for convenient imports:

```typescript
// src/lib/index.ts
export { default as Button } from "./components/ui/Button.svelte";
export { default as Card } from "./components/ui/Card.svelte";
export { default as Modal } from "./components/ui/Modal.svelte";
export { default as Header } from "./components/layout/Header.svelte";

export * from "./utils/formatters";
export * from "./utils/validators";
export * from "./stores/user";
export * from "./stores/theme";
```

**Usage:**

```svelte
<script lang="ts">
	// Import multiple items from one source
	import { Button, Card, Modal, formatDate } from '$lib';
</script>
```

### Demo Page: Testing Folder Structure

```svelte
<!-- src/routes/+page.svelte -->
<script lang="ts">
	// Clean imports using $lib
	import Button from '$lib/components/ui/Button.svelte';
	import Card from '$lib/components/ui/Card.svelte';
	import { formatDate } from '$lib/utils/formatters';

	const now = formatDate(new Date());
</script>

<div class="container mx-auto p-8">
	<h1 class="text-4xl font-bold mb-8">SvelteKit Folder Structure Demo</h1>

	<div class="grid grid-cols-1 md:grid-cols-2 gap-6">
		<Card title="$lib Alias">
			<p>Clean imports from src/lib/ using $lib alias</p>
			<p class="mt-2 text-sm opacity-70">Current date: {now}</p>
		</Card>

		<Card title="Components">
			<p>Reusable components organized by category</p>
			<Button>Click Me</Button>
		</Card>
	</div>
</div>
```

**Key Concepts:**

- **src/lib**: For shared, reusable code
- **src/routes**: For pages and API routes (file-based routing)
- **static**: For static assets (favicon, images, etc.)
- **$lib alias**: Always points to src/lib/
- **Custom aliases**: Create shortcuts for common paths

> 💡 **Best Practice**: Organize `src/lib/` by type (components, utils, stores) not by feature. Save feature organization for large components.

**⚠️ Common Mistakes:**

- Don't put routes in src/lib - they belong in src/routes
- Don't use relative imports when $lib is available
- Don't forget to export from barrel files (index.ts)

**⚡ Organization Tips:**

```
src/lib/
├── components/           # Group by UI type
│   ├── ui/              # Generic UI components
│   ├── layout/          # Layout-specific
│   └── features/        # Feature-specific (large components)
├── utils/               # Pure utility functions
├── stores/              # Global state
├── types/               # TypeScript types
└── server/              # Server-only code (DB, auth, etc.)
```

---

## 4. VSCode Extensions & Settings

### Essential VSCode Configuration

Optimize your development experience with the right tools!

**Real-World Scenario:** You want autocomplete, syntax highlighting, formatting, and error detection for Svelte and SvelteKit.

**What it does:** Configures VSCode for maximum productivity.

### 📦 Required Extensions

**1. Svelte for VS Code** (Official)

- ID: `svelte.svelte-vscode`
- Syntax highlighting, IntelliSense, diagnostics

**2. Tailwind CSS IntelliSense**

- ID: `bradlc.vscode-tailwindcss`
- Autocomplete for Tailwind classes

**3. ESLint**

- ID: `dbaeumer.vscode-eslint`
- Code linting

**4. Prettier**

- ID: `esbenp.prettier-vscode`
- Code formatting

**5. Pretty TypeScript Errors**

- ID: `yoavbls.pretty-ts-errors`
- Better TypeScript error messages

### Install All at Once

Create `.vscode/extensions.json`:

```json
{
  "recommendations": [
    "svelte.svelte-vscode",
    "bradlc.vscode-tailwindcss",
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "yoavbls.pretty-ts-errors"
  ]
}
```

VSCode will prompt teammates to install these extensions automatically!

### 📝 VSCode Settings

Create `.vscode/settings.json`:

```json
{
  // Svelte
  "svelte.enable-ts-plugin": true,

  // Editor
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "editor.codeActionsOnSave": {
    "source.fixAll.eslint": "explicit",
    "source.organizeImports": "never"
  },

  // File associations
  "files.associations": {
    "*.svelte": "svelte"
  },

  // Tailwind
  "tailwindCSS.experimental.classRegex": [
    ["class:\\s*[\"'`]([^\"'`]*)[\"'`]", "([^\"'`]*?)"],
    ["class\\s*=\\s*[\"'`]([^\"'`]*)[\"'`]", "([^\"'`]*?)"]
  ],

  // Emmet
  "emmet.includeLanguages": {
    "svelte": "html"
  },

  // TypeScript
  "typescript.tsdk": "node_modules/typescript/lib",
  "typescript.enablePromptUseWorkspaceTsdk": true
}
```

### 🎨 Prettier Configuration

Create `.prettierrc`:

```json
{
  "useTabs": true,
  "singleQuote": true,
  "trailingComma": "none",
  "printWidth": 100,
  "plugins": ["prettier-plugin-svelte"],
  "overrides": [
    {
      "files": "*.svelte",
      "options": {
        "parser": "svelte"
      }
    }
  ]
}
```

### 🔍 ESLint Configuration

The SvelteKit wizard already creates `.eslintrc.cjs`. Enhance it:

```javascript
module.exports = {
  root: true,
  extends: [
    "eslint:recommended",
    "plugin:@typescript-eslint/recommended",
    "plugin:svelte/recommended",
    "prettier",
  ],
  parser: "@typescript-eslint/parser",
  plugins: ["@typescript-eslint"],
  parserOptions: {
    sourceType: "module",
    ecmaVersion: 2020,
    extraFileExtensions: [".svelte"],
  },
  env: {
    browser: true,
    es2017: true,
    node: true,
  },
  overrides: [
    {
      files: ["*.svelte"],
      parser: "svelte-eslint-parser",
      parserOptions: {
        parser: "@typescript-eslint/parser",
      },
    },
  ],
};
```

### ⌨️ Useful Keyboard Shortcuts

Add to `.vscode/keybindings.json`:

```json
[
  {
    "key": "ctrl+shift+f",
    "command": "editor.action.formatDocument"
  },
  {
    "key": "ctrl+shift+o",
    "command": "workbench.action.gotoSymbol"
  }
]
```

### 🧪 Testing Extensions (Optional)

**Vitest:**

- ID: `vitest.explorer`

**Playwright:**

- ID: `ms-playwright.playwright`

### Demo: Verify Setup

Create a test page to verify everything works:

```svelte
<!-- src/routes/vscode-test/+page.svelte -->
<script lang="ts">
	// Test TypeScript
	interface User {
		name: string;
		email: string;
	}

	let user: User = {
		name: 'John Doe',
		email: 'john@example.com'
	};

	// Test reactivity
	let count = $state(0);

	// Test formatting (save to see Prettier work)
	function increment() {
		count++;
	}
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-5xl font-bold">VSCode Setup Test</h1>

			<!-- Test Tailwind IntelliSense (should autocomplete) -->
			<div class="stats shadow mt-8">
				<div class="stat">
					<div class="stat-title">Count</div>
					<div class="stat-value">{count}</div>
				</div>
			</div>

			<!-- Test DaisyUI components -->
			<button class="btn btn-primary mt-4" onclick={increment}> Increment </button>

			<!-- Test type checking -->
			<div class="alert alert-info mt-4">
				<span>User: {user.name} ({user.email})</span>
			</div>
		</div>
	</div>
</div>
```

**Verification Checklist:**

- ✅ Syntax highlighting works
- ✅ Tailwind classes autocomplete
- ✅ TypeScript errors show inline
- ✅ Format on save works (Prettier)
- ✅ Svelte runes (`$state`, `$derived`) are recognized
- ✅ Import paths use `$lib` alias

> 💡 **Best Practice**: Commit `.vscode/` to your repository so the entire team has the same settings.

**⚠️ Common Mistakes:**

- Don't skip the Svelte extension - it's essential
- Don't use multiple formatters - pick Prettier
- Don't ignore ESLint warnings - they prevent bugs

**⚡ Pro Tips:**

1. **IntelliSense for Tailwind**: Press `Ctrl+Space` in class attributes
2. **Go to Definition**: `F12` on imports to jump to source
3. **Quick Fix**: `Ctrl+.` to see available fixes
4. **Rename Symbol**: `F2` to rename across all files

---

## 📝 Key Takeaways

✅ SvelteKit is a full-stack framework built on Svelte  
✅ Tailwind 4 + DaisyUI = rapid UI development  
✅ File-based routing eliminates router configuration  
✅ `$lib` alias keeps imports clean and maintainable  
✅ Custom aliases improve code organization  
✅ src/lib is for shared code, src/routes is for pages  
✅ VSCode extensions dramatically improve DX  
✅ Format on save with Prettier keeps code consistent  
✅ ESLint catches errors before they reach production  
✅ DaisyUI provides theming out of the box

---

## 🚀 Next Steps

Now that you have a solid SvelteKit foundation, you're ready for:

- **Section 10**: Routing & Layouts - Master file-based routing, dynamic routes, layouts, and navigation
- Building multi-page applications
- Creating reusable layout hierarchies
- Understanding SvelteKit's powerful routing system
