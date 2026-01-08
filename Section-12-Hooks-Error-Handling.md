# Section 12: Advanced Hooks & Error Handling

## 📚 Learning Objectives

By the end of this section, you will:

- Master the handle hook for request interception
- Implement authentication with locals object
- Configure resolve function options for performance
- Customize fetch behavior with handleFetch
- Handle expected and unexpected errors gracefully
- Use universal hooks for routing and transport
- Build a complete multi-tenant SaaS with role-based access

---

## Table of Contents

- [Section 12: Advanced Hooks \& Error Handling](#section-12-advanced-hooks--error-handling)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Hooks Overview](#1-hooks-overview)
    - [What are Hooks?](#what-are-hooks)
  - [2. Handle Hook Overview](#2-handle-hook-overview)
    - [The Main Server Hook](#the-main-server-hook)
  - [3. The Handle Hook in Action](#3-the-handle-hook-in-action)
    - [Request/Response Interception](#requestresponse-interception)
  - [4. The Locals Object](#4-the-locals-object)
    - [Passing Data from Hooks to Routes](#passing-data-from-hooks-to-routes)
  - [5. Authentication with Handle Hook \& Locals](#5-authentication-with-handle-hook--locals)
    - [Secure Route Protection](#secure-route-protection)
  - [6. Resolve Function Options](#6-resolve-function-options)
    - [Customizing Response Behavior](#customizing-response-behavior)
  - [7. Fetch Function Customization](#7-fetch-function-customization)
    - [filterSerializedResponseHeaders \& handleFetch](#filterserializedresponseheaders--handlefetch)
  - [8. Handling Expected Errors](#8-handling-expected-errors)
    - [User-Friendly Error Pages](#user-friendly-error-pages)
  - [9. Handling Unexpected Errors with Hooks](#9-handling-unexpected-errors-with-hooks)
    - [Global Error Handler](#global-error-handler)
  - [10. Universal Hooks: Reroute](#10-universal-hooks-reroute)
    - [URL Rewriting](#url-rewriting)
  - [11. Universal Hooks: Transport](#11-universal-hooks-transport)
    - [Custom Data Transport](#custom-data-transport)
  - [12. Complete Example: Multi-Tenant SaaS Platform](#12-complete-example-multi-tenant-saas-platform)
    - [Full Authentication \& Error Handling System](#full-authentication--error-handling-system)
    - [📁 Project Structure](#-project-structure)
    - [Type Definitions](#type-definitions)
    - [Database \& Auth](#database--auth)
    - [Rate Limiting](#rate-limiting)
    - [Logger](#logger)
    - [Server Hooks](#server-hooks)
    - [Universal Hooks](#universal-hooks)
    - [Login Page](#login-page)
    - [Dashboard](#dashboard)
    - [Error Page](#error-page)
    - [API Health Check](#api-health-check)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Hooks Overview

### What are Hooks?

Hooks are functions that run on **every request** before your routes are loaded. They're perfect for authentication, logging, and request modification.

**Real-World Scenario:** You need to check if a user is logged in on every page request.

**Hook Files:**

- `src/hooks.server.ts` - Server-side hooks (most common)
- `src/hooks.client.ts` - Client-side hooks (rare)
- `src/hooks.ts` - Universal hooks (reroute, transport)

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  console.log("Request:", event.url.pathname);

  const response = await resolve(event);

  console.log("Response:", response.status);
  return response;
};
```

> 💡 **Best Practice**: Use hooks for cross-cutting concerns that affect all routes.

---

## 2. Handle Hook Overview

### The Main Server Hook

The `handle` hook intercepts every server request.

**Real-World Scenario:** Adding custom headers to all responses.

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  // Before route handler
  event.locals.requestTime = Date.now();

  const response = await resolve(event);

  // After route handler
  const duration = Date.now() - event.locals.requestTime;
  response.headers.set("X-Response-Time", `${duration}ms`);

  return response;
};
```

**Key Concepts:**

- `event` - Request event with url, cookies, locals
- `resolve(event)` - Calls route handler and returns response
- Modify request before `resolve()`, response after

---

## 3. The Handle Hook in Action

### Request/Response Interception

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  // Log all requests
  console.log(`${event.request.method} ${event.url.pathname}`);

  // Add custom headers to request
  event.request.headers.set("X-Custom-Header", "value");

  // Resolve the request
  const response = await resolve(event);

  // Add CORS headers to response
  if (event.url.pathname.startsWith("/api")) {
    response.headers.set("Access-Control-Allow-Origin", "*");
    response.headers.set(
      "Access-Control-Allow-Methods",
      "GET, POST, PUT, DELETE"
    );
  }

  return response;
};
```

**Use cases:**

- ✅ Logging and analytics
- ✅ Adding security headers
- ✅ CORS configuration
- ✅ Request timing

---

## 4. The Locals Object

### Passing Data from Hooks to Routes

The `locals` object passes data from hooks to your load functions and endpoints.

**Real-World Scenario:** Making user data available everywhere.

```typescript
// src/app.d.ts
declare global {
  namespace App {
    interface Locals {
      user: {
        id: string;
        email: string;
        role: "admin" | "user";
      } | null;
      requestId: string;
    }
  }
}

export {};
```

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  // Set request ID for logging
  event.locals.requestId = crypto.randomUUID();

  // Check for session cookie
  const sessionId = event.cookies.get("session");
  if (sessionId) {
    event.locals.user = await getUserFromSession(sessionId);
  } else {
    event.locals.user = null;
  }

  return resolve(event);
};
```

```typescript
// src/routes/dashboard/+page.server.ts
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async ({ locals }) => {
  // Access user from locals!
  return {
    user: locals.user,
  };
};
```

> ⚠️ **Common Mistake**: Don't protect routes in layout load functions - use hooks instead! Layout loads can be bypassed during client-side navigation.

---

## 5. Authentication with Handle Hook & Locals

### Secure Route Protection

```typescript
// src/hooks.server.ts
import { redirect } from "@sveltejs/kit";
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  // Get session from cookie
  const sessionId = event.cookies.get("session");

  if (sessionId) {
    event.locals.user = await db.getUserBySession(sessionId);
  }

  // Protected routes
  const protectedRoutes = ["/dashboard", "/admin", "/settings"];
  const isProtected = protectedRoutes.some((route) =>
    event.url.pathname.startsWith(route)
  );

  if (isProtected && !event.locals.user) {
    throw redirect(303, `/login?redirect=${event.url.pathname}`);
  }

  // Admin-only routes
  if (event.url.pathname.startsWith("/admin")) {
    if (event.locals.user?.role !== "admin") {
      throw redirect(303, "/dashboard");
    }
  }

  return resolve(event);
};
```

**Why not protect in layout loads?**

- Layout loads run on client navigation
- Users can bypass by accessing child routes directly
- Hooks run on **every** request (server + client)

---

## 6. Resolve Function Options

### Customizing Response Behavior

The `resolve` function accepts options to customize how SvelteKit handles the request.

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  // Custom transformations
  const response = await resolve(event, {
    // Transform the HTML before sending
    transformPageChunk: ({ html }) => {
      return html.replace("%app.version%", "1.0.0");
    },

    // Filter which headers are serialized for client
    filterSerializedResponseHeaders: (name) => {
      return name === "x-custom-header";
    },
  });

  return response;
};
```

**Options:**

- `transformPageChunk` - Modify HTML before sending
- `filterSerializedResponseHeaders` - Control which headers go to client
- `preload` - Force preloading of resources

---

## 7. Fetch Function Customization

### filterSerializedResponseHeaders & handleFetch

Control how `fetch` works in load functions.

```typescript
// src/hooks.server.ts
import type { Handle } from "@sveltejs/kit";

export const handle: Handle = async ({ event, resolve }) => {
  return resolve(event, {
    // Allow these headers to be read on client
    filterSerializedResponseHeaders: (name) => {
      return name === "x-rate-limit" || name === "x-api-version";
    },
  });
};
```

**handleFetch**: Intercept all fetch requests in load functions.

```typescript
// src/hooks.server.ts
import type { HandleFetch } from "@sveltejs/kit";

export const handleFetch: HandleFetch = async ({ request, fetch, event }) => {
  // Add auth token to all API requests
  if (request.url.startsWith(process.env.API_URL)) {
    request.headers.set("Authorization", `Bearer ${event.locals.user?.token}`);
  }

  // Add request ID for tracing
  request.headers.set("X-Request-ID", event.locals.requestId);

  return fetch(request);
};
```

---

## 8. Handling Expected Errors

### User-Friendly Error Pages

Use `error()` for expected errors like 404s.

```typescript
// src/routes/blog/[slug]/+page.server.ts
import { error } from "@sveltejs/kit";
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async ({ params }) => {
  const post = await db.posts.findUnique({ where: { slug: params.slug } });

  if (!post) {
    throw error(404, {
      message: "Post not found",
      hint: "Check the URL or browse all posts",
    });
  }

  if (!post.published && !event.locals.user?.isAdmin) {
    throw error(403, "This post is not published yet");
  }

  return { post };
};
```

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-6xl font-bold text-error">{$page.status}</h1>
			<p class="text-xl mt-4">{$page.error?.message}</p>

			{#if $page.error?.hint}
				<p class="text-base-content/60 mt-2">{$page.error.hint}</p>
			{/if}

			<div class="mt-8">
				<a href="/" class="btn btn-primary">Go Home</a>
			</div>
		</div>
	</div>
</div>
```

---

## 9. Handling Unexpected Errors with Hooks

### Global Error Handler

Use `handleError` hook for unexpected errors (500s).

```typescript
// src/hooks.server.ts
import type { HandleServerError } from "@sveltejs/kit";

// handleError catches ALL unexpected errors (500s)
// Use error() helper for expected errors (404, 403, etc.)
export const handleError: HandleServerError = async ({ error, event }) => {
  // Log to console with full context for debugging
  // In production, this goes to your logging service
  console.error("Unexpected error:", {
    message: error.message,
    stack: error.stack,
    url: event.url.pathname,
    user: event.locals.user?.id,
    requestId: event.locals.requestId,
  });

  // Send to monitoring service (Sentry, Datadog, etc.)
  // Helps track errors in production
  await logToSentry({
    error,
    user: event.locals.user,
    url: event.url.pathname,
  });

  // Return user-friendly message (don't leak internals!)
  // NEVER return error.stack or sensitive details to client
  return {
    message: "An unexpected error occurred. Our team has been notified.",
    code: event.locals.requestId, // Give user a reference ID for support
  };
};
```

**Error Page:**

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';
</script>

{#if $page.status >= 500}
	<div class="alert alert-error">
		<div>
			<h3 class="font-bold">Something went wrong!</h3>
			<p>{$page.error?.message}</p>
			{#if $page.error?.code}
				<p class="text-sm mt-2">Reference ID: {$page.error.code}</p>
			{/if}
		</div>
	</div>
{/if}
```

---

## 10. Universal Hooks: Reroute

### URL Rewriting

The `reroute` hook rewrites URLs before routing.

```typescript
// src/hooks.ts (universal!)
import type { Reroute } from "@sveltejs/kit";

export const reroute: Reroute = ({ url }) => {
  // Redirect old blog URLs to new structure
  if (url.pathname.startsWith("/old-blog/")) {
    return url.pathname.replace("/old-blog/", "/blog/");
  }

  // Language routing
  const match = url.pathname.match(/^\/([a-z]{2})(\/.*)?$/);
  if (match) {
    const [, lang, path = "/"] = match;
    url.searchParams.set("lang", lang);
    return path;
  }

  // Default: no rerouting
  return url.pathname;
};
```

**Use cases:**

- URL migrations
- Language routing
- Backward compatibility

---

## 11. Universal Hooks: Transport

### Custom Data Transport

The `transport` hook customizes how data is sent from server to client.

```typescript
// src/hooks.ts
import type { Transport } from "@sveltejs/kit";

export const transport: Transport = ({ data }) => {
  // Compress large data before sending to client
  if (JSON.stringify(data).length > 10000) {
    return compress(data);
  }

  return data;
};
```

> 💡 **Advanced**: Most apps don't need this. Only use for custom serialization.

---

## 12. Complete Example: Multi-Tenant SaaS Platform

### Full Authentication & Error Handling System

Let's build a complete multi-tenant platform with role-based access, error handling, and request logging!

**Features:**

- ✅ Multi-tenant architecture (subdomain routing)
- ✅ Role-based authentication (admin, member, guest)
- ✅ Global error handling with logging
- ✅ Request tracking and performance monitoring
- ✅ API rate limiting
- ✅ Custom fetch interceptors
- ✅ Comprehensive error pages

### 📁 Project Structure

```
src/
├── lib/
│   ├── server/
│   │   ├── db.ts                    # Database with tenants
│   │   ├── auth.ts                  # Auth helpers
│   │   ├── logger.ts                # Error logging
│   │   └── rateLimit.ts             # Rate limiter
│   └── components/
│       ├── ErrorDisplay.svelte
│       └── TenantSwitcher.svelte
├── routes/
│   ├── +layout.svelte               # Root layout
│   ├── +error.svelte                # Error page
│   ├── (auth)/
│   │   ├── login/
│   │   │   ├── +page.svelte
│   │   │   └── +server.ts
│   │   └── signup/
│   │       ├── +page.svelte
│   │       └── +server.ts
│   ├── (app)/                       # Protected app
│   │   ├── +layout.server.ts
│   │   ├── dashboard/
│   │   │   └── +page.server.ts
│   │   ├── settings/
│   │   │   └── +page.server.ts
│   │   └── admin/                   # Admin only
│   │       └── +page.server.ts
│   └── api/
│       ├── health/+server.ts
│       └── [...catchall]/+server.ts
├── hooks.server.ts                  # Server hooks
├── hooks.ts                         # Universal hooks
└── app.d.ts                         # Type definitions
```

### Type Definitions

```typescript
// src/app.d.ts
interface Tenant {
  id: string;
  name: string;
  subdomain: string;
}

interface User {
  id: string;
  email: string;
  name: string;
  role: "admin" | "member" | "guest";
  tenantId: string;
}

declare global {
  namespace App {
    interface Locals {
      user: User | null;
      tenant: Tenant | null;
      requestId: string;
      requestTime: number;
    }
    interface Error {
      message: string;
      code?: string;
      hint?: string;
    }
  }
}

export {};
```

### Database & Auth

```typescript
// src/lib/server/db.ts
export interface Tenant {
  id: string;
  name: string;
  subdomain: string;
}

export interface User {
  id: string;
  email: string;
  name: string;
  role: "admin" | "member" | "guest";
  tenantId: string;
}

// Mock data
const tenants: Tenant[] = [
  { id: "1", name: "Acme Corp", subdomain: "acme" },
  { id: "2", name: "Tech Startup", subdomain: "techco" },
];

const users: User[] = [
  {
    id: "1",
    email: "admin@acme.com",
    name: "Admin User",
    role: "admin",
    tenantId: "1",
  },
  {
    id: "2",
    email: "member@acme.com",
    name: "Member User",
    role: "member",
    tenantId: "1",
  },
];

const sessions = new Map<string, string>(); // sessionId -> userId

export const db = {
  tenants: {
    findBySubdomain: async (subdomain: string) => {
      return tenants.find((t) => t.subdomain === subdomain);
    },
  },
  users: {
    findById: async (id: string) => {
      return users.find((u) => u.id === id);
    },
    findByEmail: async (email: string) => {
      return users.find((u) => u.email === email);
    },
  },
  sessions: {
    get: (sessionId: string) => sessions.get(sessionId),
    create: (userId: string) => {
      const sessionId = crypto.randomUUID();
      sessions.set(sessionId, userId);
      return sessionId;
    },
    delete: (sessionId: string) => sessions.delete(sessionId),
  },
};
```

```typescript
// src/lib/server/auth.ts
import { db } from "./db";

export async function validateSession(sessionId: string | undefined) {
  if (!sessionId) return null;

  const userId = db.sessions.get(sessionId);
  if (!userId) return null;

  return await db.users.findById(userId);
}

export async function login(email: string, password: string) {
  // In production: verify bcrypt hash
  const user = await db.users.findByEmail(email);
  if (!user || password !== "password123") return null;

  const sessionId = db.sessions.create(user.id);
  return { user, sessionId };
}
```

### Rate Limiting

```typescript
// src/lib/server/rateLimit.ts
interface RateLimitEntry {
  count: number;
  resetAt: number;
}

const limits = new Map<string, RateLimitEntry>();

export function checkRateLimit(
  key: string,
  maxRequests = 100,
  windowMs = 60000
): { allowed: boolean; remaining: number; resetAt: number } {
  const now = Date.now();
  const entry = limits.get(key);

  if (!entry || now > entry.resetAt) {
    const resetAt = now + windowMs;
    limits.set(key, { count: 1, resetAt });
    return { allowed: true, remaining: maxRequests - 1, resetAt };
  }

  if (entry.count >= maxRequests) {
    return { allowed: false, remaining: 0, resetAt: entry.resetAt };
  }

  entry.count++;
  return {
    allowed: true,
    remaining: maxRequests - entry.count,
    resetAt: entry.resetAt,
  };
}
```

### Logger

```typescript
// src/lib/server/logger.ts
interface LogEntry {
  level: "info" | "warn" | "error";
  message: string;
  timestamp: string;
  requestId?: string;
  userId?: string;
  tenantId?: string;
  error?: any;
}

export const logger = {
  info: (message: string, meta?: any) => {
    console.log(
      JSON.stringify({
        level: "info",
        message,
        ...meta,
        timestamp: new Date().toISOString(),
      })
    );
  },

  warn: (message: string, meta?: any) => {
    console.warn(
      JSON.stringify({
        level: "warn",
        message,
        ...meta,
        timestamp: new Date().toISOString(),
      })
    );
  },

  error: (message: string, error: any, meta?: any) => {
    console.error(
      JSON.stringify({
        level: "error",
        message,
        error: {
          message: error.message,
          stack: error.stack,
        },
        ...meta,
        timestamp: new Date().toISOString(),
      })
    );
  },
};
```

### Server Hooks

```typescript
// src/hooks.server.ts
import { redirect, error } from "@sveltejs/kit";
import type { Handle, HandleFetch, HandleServerError } from "@sveltejs/kit";
import { sequence } from "@sveltejs/kit/hooks";
import { validateSession } from "$lib/server/auth";
import { db } from "$lib/server/db";
import { checkRateLimit } from "$lib/server/rateLimit";
import { logger } from "$lib/server/logger";

// Hook 1: Request tracking & logging
const trackingHandle: Handle = async ({ event, resolve }) => {
  event.locals.requestId = crypto.randomUUID();
  event.locals.requestTime = Date.now();

  logger.info("Request started", {
    requestId: event.locals.requestId,
    method: event.request.method,
    path: event.url.pathname,
  });

  const response = await resolve(event);

  const duration = Date.now() - event.locals.requestTime;
  logger.info("Request completed", {
    requestId: event.locals.requestId,
    status: response.status,
    duration: `${duration}ms`,
  });

  response.headers.set("X-Request-ID", event.locals.requestId);
  response.headers.set("X-Response-Time", `${duration}ms`);

  return response;
};

// Hook 2: Multi-tenant resolution
const tenantHandle: Handle = async ({ event, resolve }) => {
  const host = event.request.headers.get("host") || "";
  const subdomain = host.split(".")[0];

  if (subdomain && subdomain !== "localhost") {
    event.locals.tenant = await db.tenants.findBySubdomain(subdomain);

    if (!event.locals.tenant) {
      throw error(404, "Tenant not found");
    }
  }

  return resolve(event);
};

// Hook 3: Authentication
const authHandle: Handle = async ({ event, resolve }) => {
  const sessionId = event.cookies.get("session");
  event.locals.user = await validateSession(sessionId);

  // Verify user belongs to tenant
  if (event.locals.user && event.locals.tenant) {
    if (event.locals.user.tenantId !== event.locals.tenant.id) {
      event.cookies.delete("session", { path: "/" });
      event.locals.user = null;
      throw redirect(303, "/login");
    }
  }

  return resolve(event);
};

// Hook 4: Authorization
const authzHandle: Handle = async ({ event, resolve }) => {
  const path = event.url.pathname;

  // Protected routes require authentication
  if (
    path.startsWith("/dashboard") ||
    path.startsWith("/settings") ||
    path.startsWith("/admin")
  ) {
    if (!event.locals.user) {
      throw redirect(303, `/login?redirect=${path}`);
    }
  }

  // Admin routes require admin role
  if (path.startsWith("/admin")) {
    if (event.locals.user?.role !== "admin") {
      throw error(403, "Admin access required");
    }
  }

  return resolve(event);
};

// Hook 5: Rate limiting
const rateLimitHandle: Handle = async ({ event, resolve }) => {
  // Rate limit API routes
  if (event.url.pathname.startsWith("/api")) {
    const key = event.locals.user?.id || event.getClientAddress();
    const limit = checkRateLimit(key, 100, 60000);

    if (!limit.allowed) {
      throw error(429, "Too many requests");
    }

    const response = await resolve(event);
    response.headers.set("X-RateLimit-Remaining", limit.remaining.toString());
    response.headers.set(
      "X-RateLimit-Reset",
      new Date(limit.resetAt).toISOString()
    );
    return response;
  }

  return resolve(event);
};

// Hook 6: Security headers
const securityHandle: Handle = async ({ event, resolve }) => {
  const response = await resolve(event);

  response.headers.set("X-Frame-Options", "DENY");
  response.headers.set("X-Content-Type-Options", "nosniff");
  response.headers.set("Referrer-Policy", "strict-origin-when-cross-origin");
  response.headers.set(
    "Permissions-Policy",
    "geolocation=(), microphone=(), camera=()"
  );

  return response;
};

// Combine all hooks in sequence
export const handle = sequence(
  trackingHandle,
  tenantHandle,
  authHandle,
  authzHandle,
  rateLimitHandle,
  securityHandle
);

// Customize fetch in load functions
export const handleFetch: HandleFetch = async ({ request, fetch, event }) => {
  // Add auth token to internal API calls
  if (request.url.startsWith(event.url.origin)) {
    if (event.locals.user) {
      request.headers.set("X-User-ID", event.locals.user.id);
    }
    if (event.locals.tenant) {
      request.headers.set("X-Tenant-ID", event.locals.tenant.id);
    }
    request.headers.set("X-Request-ID", event.locals.requestId);
  }

  return fetch(request);
};

// Global error handler
export const handleError: HandleServerError = async ({ error, event }) => {
  const errorId = crypto.randomUUID();

  logger.error("Unexpected error", error, {
    errorId,
    requestId: event.locals.requestId,
    userId: event.locals.user?.id,
    tenantId: event.locals.tenant?.id,
    path: event.url.pathname,
  });

  // In production: send to Sentry, Datadog, etc.

  return {
    message: "An unexpected error occurred. Please try again.",
    code: errorId,
  };
};
```

### Universal Hooks

```typescript
// src/hooks.ts
import type { Reroute } from "@sveltejs/kit";

export const reroute: Reroute = ({ url }) => {
  // Handle legacy URLs
  if (url.pathname.startsWith("/old-dashboard")) {
    return url.pathname.replace("/old-dashboard", "/dashboard");
  }

  return url.pathname;
};
```

### Login Page

```svelte
<!-- src/routes/(auth)/login/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
	let loading = $state(false);
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content flex-col">
		<div class="card w-full max-w-sm shadow-2xl bg-base-100">
			<form
				method="POST"
				class="card-body"
				use:enhance={() => {
					loading = true;
					return async ({ update }) => {
						await update();
						loading = false;
					};
				}}
			>
				<h2 class="card-title text-2xl justify-center">Login</h2>

				{#if form?.error}
					<div class="alert alert-error">
						<span>{form.error}</span>
					</div>
				{/if}

				<div class="form-control">
					<label class="label">
						<span class="label-text">Email</span>
					</label>
					<input
						type="email"
						name="email"
						class="input input-bordered"
						value={form?.email || 'admin@acme.com'}
						required
					/>
				</div>

				<div class="form-control">
					<label class="label">
						<span class="label-text">Password</span>
					</label>
					<input
						type="password"
						name="password"
						class="input input-bordered"
						value="password123"
						required
					/>
					<label class="label">
						<span class="label-text-alt">Demo: password123</span>
					</label>
				</div>

				<div class="form-control mt-6">
					<button type="submit" class="btn btn-primary" disabled={loading}>
						{loading ? 'Logging in...' : 'Login'}
					</button>
				</div>
			</form>
		</div>
	</div>
</div>
```

```typescript
// src/routes/(auth)/login/+server.ts
import { login } from "$lib/server/auth";
import { fail, redirect } from "@sveltejs/kit";
import type { Actions } from "./$types";

export const actions: Actions = {
  default: async ({ request, cookies, url }) => {
    const data = await request.formData();
    const email = data.get("email")?.toString();
    const password = data.get("password")?.toString();

    if (!email || !password) {
      return fail(400, { error: "Email and password required", email });
    }

    const result = await login(email, password);

    if (!result) {
      return fail(401, { error: "Invalid credentials", email });
    }

    cookies.set("session", result.sessionId, {
      path: "/",
      httpOnly: true,
      sameSite: "strict",
      secure: process.env.NODE_ENV === "production",
      maxAge: 60 * 60 * 24 * 7,
    });

    const redirectTo = url.searchParams.get("redirect") || "/dashboard";
    throw redirect(303, redirectTo);
  },
};
```

### Dashboard

```typescript
// src/routes/(app)/+layout.server.ts
import type { LayoutServerLoad } from "./$types";

export const load: LayoutServerLoad = async ({ locals }) => {
  return {
    user: locals.user,
    tenant: locals.tenant,
  };
};
```

```svelte
<!-- src/routes/(app)/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();
</script>

<div class="navbar bg-base-300">
	<div class="flex-1">
		<a href="/dashboard" class="btn btn-ghost text-xl">{data.tenant?.name || 'App'}</a>
	</div>
	<div class="flex-none gap-2">
		<span class="text-sm">{data.user?.name}</span>
		<div class="badge badge-primary">{data.user?.role}</div>
		<form action="/logout" method="POST">
			<button type="submit" class="btn btn-ghost btn-sm">Logout</button>
		</form>
	</div>
</div>

<div class="container mx-auto p-6">
	{@render children()}
</div>
```

```typescript
// src/routes/(app)/dashboard/+page.server.ts
import type { PageServerLoad } from "./$types";

export const load: PageServerLoad = async ({ locals }) => {
  // Mock dashboard stats
  return {
    stats: {
      users: 42,
      revenue: 15000,
      projects: 12,
    },
  };
};
```

```svelte
<!-- src/routes/(app)/dashboard/+page.svelte -->
<script lang="ts">
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();
</script>

<h1 class="text-3xl font-bold mb-6">Dashboard</h1>

<div class="grid md:grid-cols-3 gap-6">
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Users</div>
		<div class="stat-value text-primary">{data.stats.users}</div>
	</div>
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Revenue</div>
		<div class="stat-value text-success">${data.stats.revenue}</div>
	</div>
	<div class="stat bg-base-100 shadow">
		<div class="stat-title">Projects</div>
		<div class="stat-value text-secondary">{data.stats.projects}</div>
	</div>
</div>
```

### Error Page

```svelte
<!-- src/routes/+error.svelte -->
<script lang="ts">
	import { page } from '$app/stores';

	const statusMessages = {
		400: 'Bad Request',
		401: 'Unauthorized',
		403: 'Forbidden',
		404: 'Not Found',
		429: 'Too Many Requests',
		500: 'Internal Server Error'
	};

	let title = $derived(statusMessages[$page.status] || 'Error');
</script>

<div class="hero min-h-screen bg-base-200">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<h1 class="text-6xl font-bold" class:text-error={$page.status >= 500}>
				{$page.status}
			</h1>
			<h2 class="text-2xl font-semibold mt-4">{title}</h2>
			<p class="py-6">{$page.error?.message}</p>

			{#if $page.error?.hint}
				<div class="alert alert-info">
					<span>{$page.error.hint}</span>
				</div>
			{/if}

			{#if $page.error?.code}
				<p class="text-sm text-base-content/60 mt-4">
					Reference ID: <code>{$page.error.code}</code>
				</p>
			{/if}

			<div class="mt-8 flex gap-4 justify-center">
				<a href="/" class="btn btn-primary">Go Home</a>
				{#if $page.status === 404}
					<a href="/dashboard" class="btn btn-secondary">Go to Dashboard</a>
				{/if}
			</div>
		</div>
	</div>
</div>
```

### API Health Check

```typescript
// src/routes/api/health/+server.ts
import { json } from "@sveltejs/kit";
import type { RequestHandler } from "./$types";

export const GET: RequestHandler = async ({ locals }) => {
  return json({
    status: "ok",
    timestamp: new Date().toISOString(),
    requestId: locals.requestId,
    tenant: locals.tenant?.name,
  });
};
```

**Key Features Demonstrated:**

- ✅ **Multiple Hooks**: Chained with `sequence()`
- ✅ **Multi-Tenancy**: Subdomain-based tenant resolution
- ✅ **Authentication**: Session-based with role checks
- ✅ **Authorization**: Protected routes and admin-only areas
- ✅ **Rate Limiting**: Per-user API limits
- ✅ **Request Tracking**: UUID-based request tracing
- ✅ **Error Logging**: Structured logging with context
- ✅ **Security Headers**: Best practice HTTP headers
- ✅ **Custom Fetch**: Automatic header injection
- ✅ **Error Handling**: User-friendly error pages with IDs
- ✅ **Performance**: Response time tracking

---

## 📝 Key Takeaways

✅ Hooks run on every request before routes  
✅ `handle` hook intercepts all server requests  
✅ Use `locals` to pass data from hooks to routes  
✅ Never protect routes in layout loads - use hooks!  
✅ `sequence()` chains multiple hooks together  
✅ `resolve()` options customize response behavior  
✅ `handleFetch` intercepts fetch in load functions  
✅ `error()` for expected errors (404, 403)  
✅ `handleError` for unexpected errors (500)  
✅ `reroute` hook rewrites URLs  
✅ `transport` hook customizes data serialization  
✅ Always log errors with request context  
✅ Return sanitized error messages to users

---

## 🚀 Next Steps

You've mastered hooks and error handling! Next up:

- **Section 13**: Load Function Dependencies & Invalidation
- **Section 14**: Forms & Progressive Enhancement
- Building production-ready error monitoring systems
