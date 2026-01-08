# Section 20: Testing - Vitest & Playwright

> **Critical Career Skill:** Testing is non-negotiable for junior developer roles. This section covers unit testing with Vitest and end-to-end testing with Playwright - both essential for production-grade applications.

## 📚 Learning Objectives

By the end of this section, you will be able to:

- Write unit tests for utilities and business logic with Vitest
- Test Svelte components with @testing-library/svelte
- Write integration tests for SvelteKit routes and API endpoints
- Create end-to-end tests with Playwright
- Implement TDD (Test-Driven Development) workflow
- Achieve code coverage goals (>80%)
- Mock external dependencies and APIs
- Test reactive state and user interactions

---

## Table of Contents

- [Section 20: Testing - Vitest \& Playwright](#section-20-testing---vitest--playwright)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Testing Philosophy \& Setup](#1-testing-philosophy--setup)
    - [Why Testing Matters](#why-testing-matters)
    - [Testing Pyramid](#testing-pyramid)
    - [Project Setup](#project-setup)
  - [2. Unit Testing with Vitest](#2-unit-testing-with-vitest)
    - [Testing Pure Functions](#testing-pure-functions)
    - [Testing with Dependencies](#testing-with-dependencies)
    - [Testing Async Functions](#testing-async-functions)
  - [3. Component Testing](#3-component-testing)
    - [Testing Svelte Components](#testing-svelte-components)
    - [Testing User Interactions](#testing-user-interactions)
    - [Testing Reactive State](#testing-reactive-state)
  - [4. Integration Testing](#4-integration-testing)
    - [Testing API Routes](#testing-api-routes)
    - [Testing Form Actions](#testing-form-actions)
    - [Testing Load Functions](#testing-load-functions)
  - [5. End-to-End Testing with Playwright](#5-end-to-end-testing-with-playwright)
    - [First E2E Test](#first-e2e-test)
    - [Testing User Flows](#testing-user-flows)
    - [Testing Authentication](#testing-authentication)
  - [6. Mocking \& Test Doubles](#6-mocking--test-doubles)
    - [Mocking External APIs](#mocking-external-apis)
    - [Mocking Database Calls](#mocking-database-calls)
    - [Mock Service Worker (MSW)](#mock-service-worker-msw)
  - [7. Code Coverage \& CI/CD](#7-code-coverage--cicd)
    - [Coverage Reports](#coverage-reports)
    - [GitHub Actions Setup](#github-actions-setup)
  - [8. Best Practices \& Patterns](#8-best-practices--patterns)
    - [Test Organization](#test-organization)
    - [Common Pitfalls](#common-pitfalls)
  - [🎯 Practice Exercises](#-practice-exercises)
    - [Exercise 1: Test a Form Component](#exercise-1-test-a-form-component)
    - [Exercise 2: Test an API Client](#exercise-2-test-an-api-client)
    - [Exercise 3: E2E Shopping Cart](#exercise-3-e2e-shopping-cart)
    - [Exercise 4: Test Authentication Flow](#exercise-4-test-authentication-flow)
  - [🎓 Summary](#-summary)

---

## 1. Testing Philosophy & Setup

### Why Testing Matters

**Career Impact:**

- 95% of professional jobs require testing skills
- Tests prevent regressions and bugs in production
- Tests serve as living documentation
- Tests enable confident refactoring
- Tests reduce debugging time by 60-80%

**Testing Triangle:**

```
        E2E Tests (10%)
       /            \
    Integration (20%)
   /                  \
Unit Tests (70%)
```

### Testing Pyramid

1. **Unit Tests (70%)**: Fast, isolated, test single functions/components
2. **Integration Tests (20%)**: Test how modules work together
3. **E2E Tests (10%)**: Slow, expensive, test critical user journeys

### Project Setup

**Install Testing Dependencies:**

```bash
npm install -D vitest @vitest/ui
npm install -D @testing-library/svelte @testing-library/jest-dom
npm install -D @playwright/test
npm install -D happy-dom # Fast DOM implementation for Vitest
```

**Configure Vitest** (`vite.config.ts`):

```typescript
import { sveltekit } from "@sveltejs/kit/vite";
import { defineConfig } from "vitest/config";

export default defineConfig({
  plugins: [sveltekit()],
  test: {
    // DOM environment for component tests (happy-dom is faster than jsdom)
    environment: "happy-dom",

    // Find test files anywhere in src/ (convention: *.test.ts or *.spec.ts)
    include: ["src/**/*.{test,spec}.{js,ts}"],

    // Run before all tests (setup matchers, cleanup, etc.)
    setupFiles: ["./src/tests/setup.ts"],

    // Code coverage tracking
    coverage: {
      provider: "v8", // v8 is faster than istanbul
      reporter: ["text", "html", "json"], // Output formats
      exclude: ["node_modules/", "src/tests/", "**/*.spec.ts", "**/*.test.ts"], // Don't include tests in coverage
    },

    // Fail slow tests after 10 seconds (prevent infinite loops)
    testTimeout: 10000,

    // Use describe(), it(), expect() without imports
    globals: true,
  },
});
```

**Setup File** (`src/tests/setup.ts`):

```typescript
import "@testing-library/jest-dom";
import { expect, afterEach } from "vitest";
import { cleanup } from "@testing-library/svelte";
import * as matchers from "@testing-library/jest-dom/matchers";

// Extend Vitest's expect with jest-dom matchers
expect.extend(matchers);

// Cleanup after each test
afterEach(() => {
  cleanup();
});
```

**Configure Playwright** (`playwright.config.ts`):

```typescript
import { defineConfig, devices } from "@playwright/test";

export default defineConfig({
  // Where E2E tests live (separate from unit tests)
  testDir: "./e2e",

  // Run tests in parallel (faster, but ensure tests are independent)
  fullyParallel: true,

  // Prevent test.only from passing in CI (catches dev mistakes)
  forbidOnly: !!process.env.CI,

  // Retry flaky tests on CI (network issues, timing)
  retries: process.env.CI ? 2 : 0,

  // Use fewer workers on CI (avoid resource contention)
  workers: process.env.CI ? 1 : undefined,

  // Generate HTML report (playwright-report/index.html)
  reporter: "html",

  use: {
    // Base URL for all page.goto() calls
    baseURL: "http://localhost:5173",

    // Record trace for debugging (only on retry to save space)
    trace: "on-first-retry",

    // Capture screenshots only when tests fail
    screenshot: "only-on-failure",
  },

  projects: [
    {
      name: "chromium",
      use: { ...devices["Desktop Chrome"] },
    },
    {
      name: "firefox",
      use: { ...devices["Desktop Firefox"] },
    },
    {
      name: "webkit",
      use: { ...devices["Desktop Safari"] },
    },
    {
      name: "mobile-chrome",
      use: { ...devices["Pixel 5"] },
    },
  ],

  // Start dev server before tests
  webServer: {
    command: "npm run dev",
    port: 5173,
    reuseExistingServer: !process.env.CI,
  },
});
```

**Update `package.json` scripts:**

```json
{
  "scripts": {
    "test": "vitest",
    "test:ui": "vitest --ui",
    "test:coverage": "vitest --coverage",
    "test:e2e": "playwright test",
    "test:e2e:ui": "playwright test --ui",
    "test:e2e:debug": "playwright test --debug"
  }
}
```

---

## 2. Unit Testing with Vitest

### Testing Pure Functions

**Example: Testing formatters** (`src/lib/utils/formatters.test.ts`):

```typescript
import { describe, it, expect } from "vitest";
import { formatDate, formatCurrency, truncate } from "./formatters";

describe("formatDate", () => {
  it("formats dates correctly", () => {
    const date = new Date("2026-01-06");
    expect(formatDate(date)).toBe("January 6, 2026");
  });

  it("handles invalid dates", () => {
    expect(formatDate(new Date("invalid"))).toBe("Invalid Date");
  });
});

describe("formatCurrency", () => {
  it("formats USD correctly", () => {
    expect(formatCurrency(1234.56)).toBe("$1,234.56");
  });

  it("handles zero", () => {
    expect(formatCurrency(0)).toBe("$0.00");
  });

  it("handles negative values", () => {
    expect(formatCurrency(-50)).toBe("-$50.00");
  });
});

describe("truncate", () => {
  it("truncates long strings", () => {
    expect(truncate("Hello World", 5)).toBe("Hello...");
  });

  it("does not truncate short strings", () => {
    expect(truncate("Hi", 10)).toBe("Hi");
  });

  it("handles custom suffix", () => {
    expect(truncate("Hello World", 5, "…")).toBe("Hello…");
  });
});
```

**Example: Testing validators** (`src/lib/utils/validators.test.ts`):

```typescript
import { describe, it, expect } from "vitest";
import { isValidEmail, isStrongPassword, isValidUrl } from "./validators";

describe("isValidEmail", () => {
  it("accepts valid emails", () => {
    expect(isValidEmail("test@example.com")).toBe(true);
    expect(isValidEmail("user+tag@domain.co.uk")).toBe(true);
  });

  it("rejects invalid emails", () => {
    expect(isValidEmail("not-an-email")).toBe(false);
    expect(isValidEmail("@example.com")).toBe(false);
    expect(isValidEmail("test@")).toBe(false);
  });
});

describe("isStrongPassword", () => {
  it("accepts strong passwords", () => {
    expect(isStrongPassword("MyP@ssw0rd!")).toBe(true);
  });

  it("rejects weak passwords", () => {
    expect(isStrongPassword("password")).toBe(false);
    expect(isStrongPassword("12345678")).toBe(false);
    expect(isStrongPassword("short")).toBe(false);
  });
});
```

### Testing with Dependencies

**Example: Testing storage utility** (`src/lib/utils/storage.test.ts`):

```typescript
import { describe, it, expect, beforeEach, vi } from "vitest";
import { getItem, setItem, removeItem } from "./storage";

// Mock localStorage
const localStorageMock = (() => {
  let store: Record<string, string> = {};

  return {
    getItem: (key: string) => store[key] || null,
    setItem: (key: string, value: string) => {
      store[key] = value;
    },
    removeItem: (key: string) => {
      delete store[key];
    },
    clear: () => {
      store = {};
    },
  };
})();

// Replace global localStorage
Object.defineProperty(global, "localStorage", {
  value: localStorageMock,
});

describe("storage utilities", () => {
  beforeEach(() => {
    localStorage.clear();
  });

  it("sets and gets items", () => {
    setItem("key", "value");
    expect(getItem("key")).toBe("value");
  });

  it("removes items", () => {
    setItem("key", "value");
    removeItem("key");
    expect(getItem("key")).toBeNull();
  });

  it("handles JSON serialization", () => {
    const obj = { name: "Test", count: 42 };
    setItem("object", JSON.stringify(obj));
    const retrieved = JSON.parse(getItem("object")!);
    expect(retrieved).toEqual(obj);
  });
});
```

### Testing Async Functions

**Example: Testing API client** (`src/lib/services/apiClient.test.ts`):

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import { fetchUser, createPost, updateUser } from "./apiClient";

// Mock fetch globally
global.fetch = vi.fn();

describe("apiClient", () => {
  // Reset mocks before each test to prevent test pollution
  beforeEach(() => {
    vi.resetAllMocks();
  });

  describe("fetchUser", () => {
    it("fetches user successfully", async () => {
      const mockUser = { id: 1, name: "John" };

      // Mock fetch to return fake data (don't hit real API in tests)
      (global.fetch as any).mockResolvedValueOnce({
        ok: true,
        json: async () => mockUser,
      });

      const user = await fetchUser(1);

      expect(fetch).toHaveBeenCalledWith("/api/users/1");
      expect(user).toEqual(mockUser);
    });

    it("throws on error", async () => {
      (global.fetch as any).mockResolvedValueOnce({
        ok: false,
        status: 404,
      });

      await expect(fetchUser(999)).rejects.toThrow();
    });
  });

  describe("createPost", () => {
    it("creates post with correct payload", async () => {
      const newPost = { title: "Test", content: "Content" };
      const createdPost = { id: 1, ...newPost };

      (global.fetch as any).mockResolvedValueOnce({
        ok: true,
        json: async () => createdPost,
      });

      const result = await createPost(newPost);

      expect(fetch).toHaveBeenCalledWith("/api/posts", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(newPost),
      });
      expect(result).toEqual(createdPost);
    });
  });
});
```

---

## 3. Component Testing

### Testing Svelte Components

**Example: Simple component** (`src/lib/components/Button.svelte`):

```svelte
<script lang="ts">
  import type { Snippet } from 'svelte';

  type Variant = 'primary' | 'secondary' | 'danger';

  let {
    variant = 'primary',
    disabled = false,
    onclick,
    children
  }: {
    variant?: Variant;
    disabled?: boolean;
    onclick?: () => void;
    children: Snippet;
  } = $props();
</script>

<button
  class="btn btn-{variant}"
  {disabled}
  onclick={onclick}
>
  {#render children()}
</button>
```

**Test file** (`src/lib/components/Button.test.ts`):

```typescript
import { describe, it, expect, vi } from "vitest";
import { render, screen } from "@testing-library/svelte";
import userEvent from "@testing-library/user-event";
import Button from "./Button.svelte";

describe("Button", () => {
  it("renders with text", () => {
    render(Button, { children: "Click me" });
    expect(screen.getByRole("button")).toHaveTextContent("Click me");
  });

  it("applies variant class", () => {
    render(Button, {
      variant: "danger",
      children: "Delete",
    });
    const button = screen.getByRole("button");
    expect(button).toHaveClass("btn-danger");
  });

  it("handles disabled state", () => {
    render(Button, {
      disabled: true,
      children: "Disabled",
    });
    expect(screen.getByRole("button")).toBeDisabled();
  });

  it("calls onclick handler", async () => {
    const handleClick = vi.fn();
    const user = userEvent.setup();

    render(Button, {
      onclick: handleClick,
      children: "Click",
    });

    await user.click(screen.getByRole("button"));
    expect(handleClick).toHaveBeenCalledOnce();
  });
});
```

### Testing User Interactions

**Example: Counter component** (`src/lib/components/Counter.svelte`):

```svelte
<script lang="ts">
	let count = $state(0);

	function increment() {
		count++;
	}

	function decrement() {
		count--;
	}

	function reset() {
		count = 0;
	}
</script>

<div>
	<p>Count: <span data-testid="count">{count}</span></p>
	<button onclick={increment}>Increment</button>
	<button onclick={decrement}>Decrement</button>
	<button onclick={reset}>Reset</button>
</div>
```

**Test file** (`src/lib/components/Counter.test.ts`):

```typescript
import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/svelte";
import userEvent from "@testing-library/user-event";
import Counter from "./Counter.svelte";

describe("Counter", () => {
  it("starts at zero", () => {
    render(Counter);
    expect(screen.getByTestId("count")).toHaveTextContent("0");
  });

  it("increments count", async () => {
    const user = userEvent.setup();
    render(Counter);

    await user.click(screen.getByRole("button", { name: "Increment" }));
    expect(screen.getByTestId("count")).toHaveTextContent("1");

    await user.click(screen.getByRole("button", { name: "Increment" }));
    expect(screen.getByTestId("count")).toHaveTextContent("2");
  });

  it("decrements count", async () => {
    const user = userEvent.setup();
    render(Counter);

    await user.click(screen.getByRole("button", { name: "Decrement" }));
    expect(screen.getByTestId("count")).toHaveTextContent("-1");
  });

  it("resets count", async () => {
    const user = userEvent.setup();
    render(Counter);

    await user.click(screen.getByRole("button", { name: "Increment" }));
    await user.click(screen.getByRole("button", { name: "Increment" }));
    await user.click(screen.getByRole("button", { name: "Reset" }));

    expect(screen.getByTestId("count")).toHaveTextContent("0");
  });
});
```

### Testing Reactive State

**Example: Todo list** (`src/lib/components/TodoList.test.ts`):

```typescript
import { describe, it, expect } from "vitest";
import { render, screen } from "@testing-library/svelte";
import userEvent from "@testing-library/user-event";
import TodoList from "./TodoList.svelte";

describe("TodoList", () => {
  it("adds new todos", async () => {
    const user = userEvent.setup();
    render(TodoList);

    const input = screen.getByPlaceholderText("Add todo...");
    const addButton = screen.getByRole("button", { name: "Add" });

    await user.type(input, "Buy milk");
    await user.click(addButton);

    expect(screen.getByText("Buy milk")).toBeInTheDocument();
  });

  it("toggles todo completion", async () => {
    const user = userEvent.setup();
    render(TodoList);

    // Add a todo first
    await user.type(screen.getByPlaceholderText("Add todo..."), "Buy milk");
    await user.click(screen.getByRole("button", { name: "Add" }));

    // Toggle completion
    const checkbox = screen.getByRole("checkbox");
    await user.click(checkbox);

    expect(checkbox).toBeChecked();
    expect(screen.getByText("Buy milk")).toHaveClass("completed");
  });

  it("deletes todos", async () => {
    const user = userEvent.setup();
    render(TodoList);

    // Add a todo
    await user.type(screen.getByPlaceholderText("Add todo..."), "Buy milk");
    await user.click(screen.getByRole("button", { name: "Add" }));

    // Delete it
    await user.click(screen.getByRole("button", { name: "Delete" }));

    expect(screen.queryByText("Buy milk")).not.toBeInTheDocument();
  });
});
```

---

## 4. Integration Testing

### Testing API Routes

**Example: API route** (`src/routes/api/posts/+server.ts`):

```typescript
import { json } from "@sveltejs/kit";
import type { RequestHandler } from "./$types";
import { db } from "$lib/server/db";
import { posts } from "$lib/server/db/schema";

export const GET: RequestHandler = async () => {
  const allPosts = await db.select().from(posts);
  return json(allPosts);
};

export const POST: RequestHandler = async ({ request }) => {
  const data = await request.json();

  const [newPost] = await db.insert(posts).values(data).returning();

  return json(newPost, { status: 201 });
};
```

**Test file** (`src/routes/api/posts/+server.test.ts`):

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import { GET, POST } from "./+server";

// Mock the database
vi.mock("$lib/server/db", () => ({
  db: {
    select: vi.fn(() => ({
      from: vi.fn(),
    })),
    insert: vi.fn(() => ({
      values: vi.fn(() => ({
        returning: vi.fn(),
      })),
    })),
  },
}));

import { db } from "$lib/server/db";

describe("POST /api/posts", () => {
  beforeEach(() => {
    vi.resetAllMocks();
  });

  it("returns all posts", async () => {
    const mockPosts = [
      { id: 1, title: "Post 1" },
      { id: 2, title: "Post 2" },
    ];

    // Mock the database query
    (db.select as any).mockReturnValue({
      from: vi.fn().mockResolvedValue(mockPosts),
    });

    const response = await GET();
    const data = await response.json();

    expect(data).toEqual(mockPosts);
  });

  it("creates new post", async () => {
    const newPost = { title: "New Post", content: "Content" };
    const createdPost = { id: 1, ...newPost };

    // Mock the database insert
    (db.insert as any).mockReturnValue({
      values: vi.fn().mockReturnValue({
        returning: vi.fn().mockResolvedValue([createdPost]),
      }),
    });

    const request = new Request("http://localhost/api/posts", {
      method: "POST",
      body: JSON.stringify(newPost),
    });

    const response = await POST({ request } as any);
    const data = await response.json();

    expect(response.status).toBe(201);
    expect(data).toEqual(createdPost);
  });
});
```

### Testing Form Actions

**Example: Form actions** (`src/routes/login/+page.server.ts`):

```typescript
import { fail, redirect } from "@sveltejs/kit";
import type { Actions } from "./$types";
import { auth } from "$lib/server/auth";

export const actions: Actions = {
  login: async ({ request, cookies }) => {
    const data = await request.formData();
    const email = data.get("email")?.toString();
    const password = data.get("password")?.toString();

    if (!email || !password) {
      return fail(400, {
        email,
        error: "Email and password required",
      });
    }

    const result = await auth.signIn({ email, password });

    if (!result.success) {
      return fail(401, {
        email,
        error: "Invalid credentials",
      });
    }

    cookies.set("session", result.sessionId, { path: "/" });
    throw redirect(303, "/dashboard");
  },
};
```

**Test file** (`src/routes/login/+page.server.test.ts`):

```typescript
import { describe, it, expect, vi } from "vitest";
import { actions } from "./+page.server";

vi.mock("$lib/server/auth", () => ({
  auth: {
    signIn: vi.fn(),
  },
}));

import { auth } from "$lib/server/auth";

describe("login action", () => {
  it("fails with missing fields", async () => {
    const formData = new FormData();
    formData.append("email", "");

    const request = new Request("http://localhost", {
      method: "POST",
      body: formData,
    });

    const result = await actions.login({ request } as any);

    expect(result).toMatchObject({
      status: 400,
      data: { error: "Email and password required" },
    });
  });

  it("fails with invalid credentials", async () => {
    (auth.signIn as any).mockResolvedValue({ success: false });

    const formData = new FormData();
    formData.append("email", "test@example.com");
    formData.append("password", "wrong");

    const request = new Request("http://localhost", {
      method: "POST",
      body: formData,
    });

    const result = await actions.login({ request } as any);

    expect(result).toMatchObject({
      status: 401,
      data: { error: "Invalid credentials" },
    });
  });

  it("redirects on success", async () => {
    (auth.signIn as any).mockResolvedValue({
      success: true,
      sessionId: "abc123",
    });

    const formData = new FormData();
    formData.append("email", "test@example.com");
    formData.append("password", "correct");

    const cookies = {
      set: vi.fn(),
    };

    const request = new Request("http://localhost", {
      method: "POST",
      body: formData,
    });

    try {
      await actions.login({ request, cookies } as any);
    } catch (e: any) {
      expect(e.status).toBe(303);
      expect(e.location).toBe("/dashboard");
    }

    expect(cookies.set).toHaveBeenCalledWith("session", "abc123", {
      path: "/",
    });
  });
});
```

### Testing Load Functions

**Example: Page load** (`src/routes/posts/[id]/+page.server.ts`):

```typescript
import { error } from "@sveltejs/kit";
import type { PageServerLoad } from "./$types";
import { db } from "$lib/server/db";
import { posts } from "$lib/server/db/schema";
import { eq } from "drizzle-orm";

export const load: PageServerLoad = async ({ params }) => {
  const [post] = await db
    .select()
    .from(posts)
    .where(eq(posts.id, parseInt(params.id)));

  if (!post) {
    throw error(404, "Post not found");
  }

  return { post };
};
```

**Test file** (`src/routes/posts/[id]/+page.server.test.ts`):

```typescript
import { describe, it, expect, vi } from "vitest";
import { load } from "./+page.server";

vi.mock("$lib/server/db");
import { db } from "$lib/server/db";

describe("post detail load", () => {
  it("returns post when found", async () => {
    const mockPost = { id: 1, title: "Test Post" };

    (db.select as any).mockReturnValue({
      from: vi.fn().mockReturnValue({
        where: vi.fn().mockResolvedValue([mockPost]),
      }),
    });

    const result = await load({ params: { id: "1" } } as any);

    expect(result).toEqual({ post: mockPost });
  });

  it("throws 404 when not found", async () => {
    (db.select as any).mockReturnValue({
      from: vi.fn().mockReturnValue({
        where: vi.fn().mockResolvedValue([]),
      }),
    });

    await expect(load({ params: { id: "999" } } as any)).rejects.toThrow(
      "Post not found"
    );
  });
});
```

---

## 5. End-to-End Testing with Playwright

### First E2E Test

**Example: Homepage test** (`e2e/homepage.spec.ts`):

```typescript
import { test, expect } from "@playwright/test";

test("homepage loads", async ({ page }) => {
  await page.goto("/");

  // Check title
  await expect(page).toHaveTitle(/My App/);

  // Check heading
  const heading = page.getByRole("heading", { level: 1 });
  await expect(heading).toHaveText("Welcome");

  // Check navigation
  const nav = page.getByRole("navigation");
  await expect(nav).toBeVisible();
});

test("navigation works", async ({ page }) => {
  await page.goto("/");

  // Click "About" link
  await page.getByRole("link", { name: "About" }).click();

  // Verify URL changed
  await expect(page).toHaveURL("/about");

  // Verify content loaded
  await expect(page.getByText("About Us")).toBeVisible();
});
```

### Testing User Flows

**Example: Todo app flow** (`e2e/todos.spec.ts`):

```typescript
import { test, expect } from "@playwright/test";

test.describe("Todo App", () => {
  test.beforeEach(async ({ page }) => {
    await page.goto("/todos");
  });

  test("creates a new todo", async ({ page }) => {
    // Type in the input
    const input = page.getByPlaceholder("Add todo...");
    await input.fill("Buy groceries");

    // Submit the form
    await page.getByRole("button", { name: "Add" }).click();

    // Verify todo appears
    await expect(page.getByText("Buy groceries")).toBeVisible();

    // Input should be cleared
    await expect(input).toHaveValue("");
  });

  test("completes a todo", async ({ page }) => {
    // Add a todo first
    await page.getByPlaceholder("Add todo...").fill("Buy groceries");
    await page.getByRole("button", { name: "Add" }).click();

    // Click the checkbox
    const checkbox = page.getByRole("checkbox").first();
    await checkbox.check();

    // Verify it's checked
    await expect(checkbox).toBeChecked();

    // Verify visual styling
    const todoItem = page.getByText("Buy groceries");
    await expect(todoItem).toHaveClass(/completed/);
  });

  test("deletes a todo", async ({ page }) => {
    // Add a todo
    await page.getByPlaceholder("Add todo...").fill("Buy groceries");
    await page.getByRole("button", { name: "Add" }).click();

    // Click delete button
    await page.getByRole("button", { name: "Delete" }).first().click();

    // Verify it's gone
    await expect(page.getByText("Buy groceries")).not.toBeVisible();
  });

  test("persists todos after refresh", async ({ page }) => {
    // Add a todo
    await page.getByPlaceholder("Add todo...").fill("Buy groceries");
    await page.getByRole("button", { name: "Add" }).click();

    // Refresh the page
    await page.reload();

    // Todo should still be there
    await expect(page.getByText("Buy groceries")).toBeVisible();
  });
});
```

### Testing Authentication

**Example: Login flow** (`e2e/auth.spec.ts`):

```typescript
import { test, expect } from "@playwright/test";

test.describe("Authentication", () => {
  test("logs in successfully", async ({ page }) => {
    await page.goto("/login");

    // Fill in the form
    await page.getByLabel("Email").fill("test@example.com");
    await page.getByLabel("Password").fill("password123");

    // Submit
    await page.getByRole("button", { name: "Log In" }).click();

    // Should redirect to dashboard
    await expect(page).toHaveURL("/dashboard");

    // Should show user menu
    await expect(page.getByText("test@example.com")).toBeVisible();
  });

  test("shows error for invalid credentials", async ({ page }) => {
    await page.goto("/login");

    await page.getByLabel("Email").fill("test@example.com");
    await page.getByLabel("Password").fill("wrongpassword");
    await page.getByRole("button", { name: "Log In" }).click();

    // Should show error message
    await expect(page.getByText("Invalid credentials")).toBeVisible();

    // Should stay on login page
    await expect(page).toHaveURL("/login");
  });

  test("protects authenticated routes", async ({ page }) => {
    // Try to access dashboard without logging in
    await page.goto("/dashboard");

    // Should redirect to login
    await expect(page).toHaveURL("/login");
  });

  test("logs out successfully", async ({ page, context }) => {
    // Login first
    await page.goto("/login");
    await page.getByLabel("Email").fill("test@example.com");
    await page.getByLabel("Password").fill("password123");
    await page.getByRole("button", { name: "Log In" }).click();

    // Wait for dashboard
    await expect(page).toHaveURL("/dashboard");

    // Click logout
    await page.getByRole("button", { name: "Log Out" }).click();

    // Should redirect to home
    await expect(page).toHaveURL("/");

    // Try to access dashboard again
    await page.goto("/dashboard");
    await expect(page).toHaveURL("/login");
  });
});

// Helper for authenticated tests
test.describe("Authenticated Tests", () => {
  test.use({
    storageState: "playwright/.auth/user.json",
  });

  test("accesses dashboard when logged in", async ({ page }) => {
    await page.goto("/dashboard");
    await expect(page.getByText("Welcome back!")).toBeVisible();
  });
});
```

**Setup authenticated storage** (`e2e/auth.setup.ts`):

```typescript
import { test as setup, expect } from "@playwright/test";

const authFile = "playwright/.auth/user.json";

setup("authenticate", async ({ page }) => {
  await page.goto("/login");
  await page.getByLabel("Email").fill("test@example.com");
  await page.getByLabel("Password").fill("password123");
  await page.getByRole("button", { name: "Log In" }).click();

  await expect(page).toHaveURL("/dashboard");

  await page.context().storageState({ path: authFile });
});
```

---

## 6. Mocking & Test Doubles

### Mocking External APIs

**Example: Weather API mock** (`src/lib/services/weather.test.ts`):

```typescript
import { describe, it, expect, vi, beforeEach } from "vitest";
import { getWeather } from "./weather";

global.fetch = vi.fn();

describe("getWeather", () => {
  beforeEach(() => {
    vi.resetAllMocks();
  });

  it("fetches weather data", async () => {
    const mockWeather = {
      temperature: 72,
      condition: "sunny",
      humidity: 45,
    };

    (global.fetch as any).mockResolvedValueOnce({
      ok: true,
      json: async () => mockWeather,
    });

    const result = await getWeather("New York");

    expect(fetch).toHaveBeenCalledWith(
      expect.stringContaining("/api/weather?city=New York")
    );
    expect(result).toEqual(mockWeather);
  });

  it("handles API errors gracefully", async () => {
    (global.fetch as any).mockResolvedValueOnce({
      ok: false,
      status: 500,
    });

    await expect(getWeather("Unknown")).rejects.toThrow();
  });

  it("retries on failure", async () => {
    // Fail twice, succeed on third try
    (global.fetch as any)
      .mockRejectedValueOnce(new Error("Network error"))
      .mockRejectedValueOnce(new Error("Network error"))
      .mockResolvedValueOnce({
        ok: true,
        json: async () => ({ temperature: 72 }),
      });

    const result = await getWeather("New York");
    expect(result.temperature).toBe(72);
    expect(fetch).toHaveBeenCalledTimes(3);
  });
});
```

### Mocking Database Calls

**Example: User repository mock**:

```typescript
import { describe, it, expect, vi } from "vitest";
import type { User } from "$lib/types/user";

// Mock the database module
vi.mock("$lib/server/db", () => ({
  db: {
    select: vi.fn(),
    insert: vi.fn(),
    update: vi.fn(),
    delete: vi.fn(),
  },
}));

import { db } from "$lib/server/db";
import { UserRepository } from "./userRepository";

describe("UserRepository", () => {
  it("finds user by id", async () => {
    const mockUser: User = {
      id: 1,
      email: "test@example.com",
      name: "Test User",
    };

    (db.select as any).mockReturnValue({
      from: vi.fn().mockReturnValue({
        where: vi.fn().mockResolvedValue([mockUser]),
      }),
    });

    const repo = new UserRepository();
    const user = await repo.findById(1);

    expect(user).toEqual(mockUser);
  });

  it("creates new user", async () => {
    const newUser = {
      email: "new@example.com",
      name: "New User",
    };

    const createdUser = { id: 1, ...newUser };

    (db.insert as any).mockReturnValue({
      values: vi.fn().mockReturnValue({
        returning: vi.fn().mockResolvedValue([createdUser]),
      }),
    });

    const repo = new UserRepository();
    const user = await repo.create(newUser);

    expect(user).toEqual(createdUser);
  });
});
```

### Mock Service Worker (MSW)

**Setup MSW for browser-like mocking** (`src/tests/mocks/handlers.ts`):

```typescript
import { http, HttpResponse } from "msw";

export const handlers = [
  // Mock GET /api/users
  http.get("/api/users", () => {
    return HttpResponse.json([
      { id: 1, name: "John" },
      { id: 2, name: "Jane" },
    ]);
  }),

  // Mock POST /api/users
  http.post("/api/users", async ({ request }) => {
    const newUser = await request.json();
    return HttpResponse.json({ id: 3, ...newUser }, { status: 201 });
  }),

  // Mock error
  http.get("/api/error", () => {
    return HttpResponse.json(
      { error: "Internal Server Error" },
      { status: 500 }
    );
  }),
];
```

**Setup server** (`src/tests/mocks/server.ts`):

```typescript
import { setupServer } from "msw/node";
import { handlers } from "./handlers";

export const server = setupServer(...handlers);
```

**Configure in setup file** (`src/tests/setup.ts`):

```typescript
import { beforeAll, afterEach, afterAll } from "vitest";
import { server } from "./mocks/server";

// Start server before all tests
beforeAll(() => server.listen());

// Reset handlers after each test
afterEach(() => server.resetHandlers());

// Clean up after all tests
afterAll(() => server.close());
```

**Use in tests**:

```typescript
import { describe, it, expect } from "vitest";
import { server } from "../tests/mocks/server";
import { http, HttpResponse } from "msw";
import { fetchUsers } from "./api";

describe("fetchUsers", () => {
  it("fetches users successfully", async () => {
    const users = await fetchUsers();
    expect(users).toHaveLength(2);
  });

  it("handles custom response", async () => {
    // Override default handler for this test
    server.use(
      http.get("/api/users", () => {
        return HttpResponse.json([{ id: 99, name: "Custom User" }]);
      })
    );

    const users = await fetchUsers();
    expect(users[0].name).toBe("Custom User");
  });
});
```

---

## 7. Code Coverage & CI/CD

### Coverage Reports

**Run coverage:**

```bash
npm run test:coverage
```

**Coverage output:**

```
 % Coverage report from v8
-------------------------------|---------|----------|---------|---------|
File                           | % Stmts | % Branch | % Funcs | % Lines |
-------------------------------|---------|----------|---------|---------|
All files                      |   87.5  |    82.1  |   91.3  |   87.5  |
 utils                         |   95.2  |    90.0  |   100   |   95.2  |
  formatters.ts                |   100   |    100   |   100   |   100   |
  validators.ts                |   91.6  |    85.7  |   100   |   91.6  |
 components                    |   82.1  |    75.0  |   85.7  |   82.1  |
  Button.svelte                |   100   |    100   |   100   |   100   |
  Counter.svelte               |   90.0  |    80.0  |   100   |   90.0  |
-------------------------------|---------|----------|---------|---------|
```

**Coverage thresholds** (add to `vite.config.ts`):

```typescript
export default defineConfig({
  test: {
    coverage: {
      provider: "v8",
      reporter: ["text", "html", "json", "lcov"],

      // Enforce minimum coverage
      thresholds: {
        lines: 80,
        functions: 80,
        branches: 75,
        statements: 80,
      },

      exclude: [
        "node_modules/",
        "src/tests/",
        "**/*.spec.ts",
        "**/*.test.ts",
        "**/*.config.ts",
        "**/types/**",
      ],
    },
  },
});
```

### GitHub Actions Setup

**CI workflow** (`.github/workflows/test.yml`):

```yaml
name: Test

on:
  push:
    branches: [main, dev]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Run linter
        run: npm run lint

      - name: Run type check
        run: npm run check

      - name: Run unit tests
        run: npm run test:coverage

      - name: Upload coverage to Codecov
        uses: codecov/codecov-action@v3
        with:
          files: ./coverage/lcov.info
          fail_ci_if_error: true

  e2e:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - name: Install dependencies
        run: npm ci

      - name: Install Playwright browsers
        run: npx playwright install --with-deps

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Upload test results
        if: always()
        uses: actions/upload-artifact@v3
        with:
          name: playwright-report
          path: playwright-report/
          retention-days: 30
```

---

## 8. Best Practices & Patterns

### Test Organization

**File structure:**

```
src/
  lib/
    utils/
      formatters.ts
      formatters.test.ts      # Co-located with source
    components/
      Button.svelte
      Button.test.ts
  routes/
    api/
      posts/
        +server.ts
        +server.test.ts       # Co-located with routes
tests/
  setup.ts                    # Global test setup
  mocks/
    handlers.ts               # MSW handlers
    server.ts
e2e/
  homepage.spec.ts            # E2E tests separate
  auth.spec.ts
  auth.setup.ts
```

**Test naming conventions:**

```typescript
// ✅ Good: Descriptive test names
describe("formatCurrency", () => {
  it("formats positive numbers with dollar sign", () => {});
  it("formats negative numbers with minus sign", () => {});
  it("rounds to two decimal places", () => {});
});

// ❌ Bad: Vague test names
describe("formatCurrency", () => {
  it("works", () => {});
  it("test 1", () => {});
});
```

**AAA Pattern (Arrange, Act, Assert):**

```typescript
it("increments counter", async () => {
  // Arrange
  const user = userEvent.setup();
  render(Counter);

  // Act
  await user.click(screen.getByRole("button", { name: "Increment" }));

  // Assert
  expect(screen.getByTestId("count")).toHaveTextContent("1");
});
```

### Common Pitfalls

**❌ Don't test implementation details:**

```typescript
// Bad: Testing internal state
it("updates state variable", () => {
  const counter = new Counter();
  counter.increment();
  expect(counter._internalCount).toBe(1); // Internal detail
});

// Good: Test observable behavior
it("displays incremented count", async () => {
  render(Counter);
  await user.click(screen.getByRole("button", { name: "Increment" }));
  expect(screen.getByTestId("count")).toHaveTextContent("1");
});
```

**❌ Don't make tests depend on each other:**

```typescript
// Bad: Tests depend on order
describe("Counter", () => {
  let count = 0;

  it("increments", () => {
    count++;
    expect(count).toBe(1);
  });

  it("increments again", () => {
    count++;
    expect(count).toBe(2); // Fails if run alone
  });
});

// Good: Each test is independent
describe("Counter", () => {
  it("increments from zero", () => {
    let count = 0;
    count++;
    expect(count).toBe(1);
  });

  it("increments from any value", () => {
    let count = 5;
    count++;
    expect(count).toBe(6);
  });
});
```

**✅ Use factories for test data:**

```typescript
// Test data factory
function createMockUser(overrides = {}) {
  return {
    id: 1,
    email: "test@example.com",
    name: "Test User",
    role: "user",
    ...overrides,
  };
}

// Use in tests
it("displays admin badge for admin users", () => {
  const admin = createMockUser({ role: "admin" });
  render(UserCard, { user: admin });
  expect(screen.getByText("Admin")).toBeInTheDocument();
});
```

**✅ Test edge cases:**

```typescript
describe("divide", () => {
  it("divides positive numbers", () => {
    expect(divide(10, 2)).toBe(5);
  });

  it("handles zero dividend", () => {
    expect(divide(0, 5)).toBe(0);
  });

  it("throws on division by zero", () => {
    expect(() => divide(10, 0)).toThrow("Division by zero");
  });

  it("handles negative numbers", () => {
    expect(divide(-10, 2)).toBe(-5);
  });

  it("handles decimal results", () => {
    expect(divide(10, 3)).toBeCloseTo(3.333, 2);
  });
});
```

---

## 🎯 Practice Exercises

### Exercise 1: Test a Form Component

Create a `ContactForm.svelte` component with name, email, and message fields. Write tests for:

- Form validation
- Submission with valid data
- Error display for invalid data
- Loading state during submission

### Exercise 2: Test an API Client

Create an API client for a blog with:

- `getPosts()` - fetch all posts
- `getPost(id)` - fetch single post
- `createPost(data)` - create new post
- `updatePost(id, data)` - update post
- `deletePost(id)` - delete post

Write comprehensive tests including error handling.

### Exercise 3: E2E Shopping Cart

Create E2E tests for a shopping cart:

- Add items to cart
- Update quantities
- Remove items
- Apply discount codes
- Checkout flow

### Exercise 4: Test Authentication Flow

Write integration tests for:

- User registration
- Email verification
- Login/logout
- Password reset
- Session management

---

## 🎓 Summary

You now know how to:

- ✅ Set up Vitest and Playwright in a SvelteKit project
- ✅ Write unit tests for functions and utilities
- ✅ Test Svelte components with Testing Library
- ✅ Write integration tests for API routes and actions
- ✅ Create end-to-end tests with Playwright
- ✅ Mock external dependencies and APIs
- ✅ Measure and enforce code coverage
- ✅ Set up CI/CD pipelines for automated testing
- ✅ Follow testing best practices and avoid common pitfalls

**Next Steps:**

- Practice TDD (write tests first, then implementation)
- Aim for 80%+ code coverage on critical paths
- Write E2E tests for all critical user journeys
- Set up pre-commit hooks to run tests
- Integrate testing into your CI/CD pipeline

**Career Impact:**
Testing skills are **mandatory** for professional development roles. This section makes you immediately more hireable and prepares you for real-world development where untested code is unacceptable.
