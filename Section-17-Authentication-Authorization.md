# Section 17: Authentication & Authorization

## 📚 Learning Objectives

By the end of this section, you will:

- Install and configure Better Auth for SvelteKit
- Create sign-in and registration routes
- Implement email verification with Resend
- Populate locals object with session data
- Handle login, logout, and OAuth flows
- Use CASL for permission management
- Protect routes and actions based on permissions
- Build a complete auth system with role-based access

---

## Table of Contents

- [Section 17: Authentication \& Authorization](#section-17-authentication--authorization)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Installing \& Configuring Better Auth](#1-installing--configuring-better-auth)
    - [Modern Auth for SvelteKit](#modern-auth-for-sveltekit)
  - [2. Creating Sign In \& Register Routes](#2-creating-sign-in--register-routes)
    - [Auth UI Components](#auth-ui-components)
  - [3. The Register Action](#3-the-register-action)
    - [User Registration Flow](#user-registration-flow)
  - [4. Email Verification with Resend](#4-email-verification-with-resend)
    - [Sending Verification Emails](#sending-verification-emails)
  - [5. Populating Locals with Session](#5-populating-locals-with-session)
    - [Making User Available Everywhere](#making-user-available-everywhere)
  - [6. Login \& Logout Actions](#6-login--logout-actions)
    - [Session Management](#session-management)
  - [7. OAuth with GitHub](#7-oauth-with-github)
    - [Social Authentication](#social-authentication)
  - [8. Authorization Basics](#8-authorization-basics)
    - [Protecting Routes](#protecting-routes)
  - [9. Permission Management with CASL](#9-permission-management-with-casl)
    - [Role-Based Access Control](#role-based-access-control)
  - [10. Complete Example: Multi-Tenant Platform with Auth](#10-complete-example-multi-tenant-platform-with-auth)
    - [Full Authentication \& Authorization System](#full-authentication--authorization-system)
    - [Protected Workspace Layout](#protected-workspace-layout)
    - [Protected Workspace Settings](#protected-workspace-settings)
    - [Protected Page Actions](#protected-page-actions)
    - [Conditional UI Based on Permissions](#conditional-ui-based-on-permissions)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Installing & Configuring Better Auth

### Modern Auth for SvelteKit

Better Auth is a modern authentication library built for SvelteKit.

**Installation:**

```bash
npm install better-auth
npm install @better-auth/svelte
npm install drizzle-orm
npm install resend  # For email verification
```

**Configure Better Auth:**

```typescript
// src/lib/server/auth.ts
import { betterAuth } from 'better-auth';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { db } from './db';
import { DATABASE_URL } from '$env/dynamic/private';

export const auth = betterAuth({
	database: drizzleAdapter(db, {
		provider: 'pg'
	}),
	emailAndPassword: {
		enabled: true,
		requireEmailVerification: true
	},
	socialProviders: {
		github: {
			clientId: process.env.GITHUB_CLIENT_ID!,
			clientSecret: process.env.GITHUB_CLIENT_SECRET!
		}
	},
	session: {
		expiresIn: 60 * 60 * 24 * 7, // 7 days
		updateAge: 60 * 60 * 24 // 1 day
	}
});
```

**Database schema for auth:**

```typescript
// Add to src/lib/server/db/schema.ts
import { pgTable, text, timestamp, boolean, integer } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
	id: text('id').primaryKey(),
	email: text('email').notNull().unique(),
	emailVerified: boolean('email_verified').default(false),
	name: text('name'),
	avatarUrl: text('avatar_url'),
	createdAt: timestamp('created_at').defaultNow(),
	updatedAt: timestamp('updated_at').defaultNow()
});

export const sessions = pgTable('sessions', {
	id: text('id').primaryKey(),
	userId: text('user_id')
		.notNull()
		.references(() => users.id, { onDelete: 'cascade' }),
	expiresAt: timestamp('expires_at').notNull(),
	token: text('token').notNull().unique(),
	ipAddress: text('ip_address'),
	userAgent: text('user_agent')
});

export const accounts = pgTable('accounts', {
	id: text('id').primaryKey(),
	userId: text('user_id')
		.notNull()
		.references(() => users.id, { onDelete: 'cascade' }),
	accountId: text('account_id').notNull(),
	providerId: text('provider_id').notNull(),
	accessToken: text('access_token'),
	refreshToken: text('refresh_token'),
	expiresAt: timestamp('expires_at'),
	password: text('password') // Hashed password for email/password
});

export const verificationTokens = pgTable('verification_tokens', {
	id: text('id').primaryKey(),
	identifier: text('identifier').notNull(),
	token: text('token').notNull().unique(),
	expires: timestamp('expires').notNull()
});
```

**Run migrations:**

```bash
npm run db:push
```

**Environment variables:**

```env
DATABASE_URL=postgresql://user:password@localhost:5432/mydb
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
RESEND_API_KEY=your_resend_api_key
PUBLIC_APP_URL=http://localhost:5173
```

> 💡 **Best Practice**: Store all auth secrets in environment variables, never commit them.

---

## 2. Creating Sign In & Register Routes

### Auth UI Components

Build clean authentication forms.

```svelte
<!-- src/routes/(auth)/register/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
	let loading = $state(false);
</script>

<div class="min-h-screen flex items-center justify-center bg-base-200">
	<div class="card w-full max-w-md bg-base-100 shadow-xl">
		<div class="card-body">
			<h2 class="card-title text-3xl justify-center mb-6">Create Account</h2>

			{#if form?.error}
				<div class="alert alert-error">
					<span>{form.error}</span>
				</div>
			{/if}

			{#if form?.success}
				<div class="alert alert-success">
					<span>Account created! Check your email to verify.</span>
				</div>
			{:else}
				<form
					method="POST"
					class="space-y-4"
					use:enhance={() => {
						loading = true;
						return async ({ update }) => {
							await update();
							loading = false;
						};
					}}
				>
					<div class="form-control">
						<label class="label">
							<span class="label-text">Name</span>
						</label>
						<input
							type="text"
							name="name"
							class="input input-bordered"
							placeholder="John Doe"
							required
						/>
					</div>

					<div class="form-control">
						<label class="label">
							<span class="label-text">Email</span>
						</label>
						<input
							type="email"
							name="email"
							class="input input-bordered"
							placeholder="you@example.com"
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
							placeholder="••••••••"
							required
						/>
						<label class="label">
							<span class="label-text-alt">At least 8 characters</span>
						</label>
					</div>

					<button type="submit" class="btn btn-primary btn-block" disabled={loading}>
						{loading ? 'Creating account...' : 'Sign up'}
					</button>
				</form>

				<div class="divider">OR</div>

				<a href="/api/auth/github" class="btn btn-outline gap-2">
					<svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
						<path
							d="M12 0C5.37 0 0 5.37 0 12c0 5.31 3.435 9.795 8.205 11.385.6.105.825-.255.825-.57 0-.285-.015-1.23-.015-2.235-3.015.555-3.795-.735-4.035-1.41-.135-.345-.72-1.41-1.23-1.695-.42-.225-1.02-.78-.015-.795.945-.015 1.62.87 1.845 1.23 1.08 1.815 2.805 1.305 3.495.99.105-.78.42-1.305.765-1.605-2.67-.3-5.46-1.335-5.46-5.925 0-1.305.465-2.385 1.23-3.225-.12-.3-.54-1.53.12-3.18 0 0 1.005-.315 3.3 1.23.96-.27 1.98-.405 3-.405s2.04.135 3 .405c2.295-1.56 3.3-1.23 3.3-1.23.66 1.65.24 2.88.12 3.18.765.84 1.23 1.905 1.23 3.225 0 4.605-2.805 5.625-5.475 5.925.435.375.81 1.095.81 2.22 0 1.605-.015 2.895-.015 3.3 0 .315.225.69.825.57A12.02 12.02 0 0024 12c0-6.63-5.37-12-12-12z"
						/>
					</svg>
					Sign up with GitHub
				</a>

				<p class="text-center text-sm mt-4">
					Already have an account?
					<a href="/login" class="link link-primary">Sign in</a>
				</p>
			{/if}
		</div>
	</div>
</div>
```

```svelte
<!-- src/routes/(auth)/login/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import type { ActionData } from './$types';

	let { form }: { form: ActionData } = $props();
	let loading = $state(false);
</script>

<div class="min-h-screen flex items-center justify-center bg-base-200">
	<div class="card w-full max-w-md bg-base-100 shadow-xl">
		<div class="card-body">
			<h2 class="card-title text-3xl justify-center mb-6">Sign In</h2>

			{#if form?.error}
				<div class="alert alert-error">
					<span>{form.error}</span>
				</div>
			{/if}

			<form
				method="POST"
				class="space-y-4"
				use:enhance={() => {
					loading = true;
					return async ({ update }) => {
						await update();
						loading = false;
					};
				}}
			>
				<div class="form-control">
					<label class="label">
						<span class="label-text">Email</span>
					</label>
					<input
						type="email"
						name="email"
						class="input input-bordered"
						placeholder="you@example.com"
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
						placeholder="••••••••"
						required
					/>
					<label class="label">
						<a href="/forgot-password" class="label-text-alt link link-hover"> Forgot password? </a>
					</label>
				</div>

				<button type="submit" class="btn btn-primary btn-block" disabled={loading}>
					{loading ? 'Signing in...' : 'Sign in'}
				</button>
			</form>

			<div class="divider">OR</div>

			<a href="/api/auth/github" class="btn btn-outline gap-2">
				<svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
					<!-- GitHub icon path -->
				</svg>
				Sign in with GitHub
			</a>

			<p class="text-center text-sm mt-4">
				Don't have an account?
				<a href="/register" class="link link-primary">Sign up</a>
			</p>
		</div>
	</div>
</div>
```

---

## 3. The Register Action

### User Registration Flow

Handle user registration on the server.

```typescript
// src/routes/(auth)/register/+page.server.ts
import { fail } from '@sveltejs/kit';
import { auth } from '$lib/server/auth';
import { sendVerificationEmail } from '$lib/server/email';
import type { Actions } from './$types';

export const actions: Actions = {
	default: async ({ request }) => {
		const data = await request.formData();
		const name = data.get('name')?.toString();
		const email = data.get('email')?.toString();
		const password = data.get('password')?.toString();

		// Validation
		if (!name || name.length < 2) {
			return fail(400, { error: 'Name must be at least 2 characters' });
		}

		if (!email || !email.includes('@')) {
			return fail(400, { error: 'Please enter a valid email' });
		}

		if (!password || password.length < 8) {
			return fail(400, { error: 'Password must be at least 8 characters' });
		}

		try {
			// Create user with Better Auth
			const result = await auth.api.signUp.email({
				body: {
					name,
					email,
					password
				}
			});

			if (!result.user) {
				return fail(400, { error: 'Failed to create account' });
			}

			// Send verification email
			await sendVerificationEmail(result.user.email, result.user.id);

			return {
				success: true,
				message: 'Account created! Please check your email to verify.'
			};
		} catch (error: any) {
			if (error.message?.includes('duplicate')) {
				return fail(400, { error: 'An account with this email already exists' });
			}
			return fail(500, { error: 'Failed to create account. Please try again.' });
		}
	}
};
```

---

## 4. Email Verification with Resend

### Sending Verification Emails

Use Resend to send beautiful verification emails.

```typescript
// src/lib/server/email.ts
import { Resend } from 'resend';
import { RESEND_API_KEY, PUBLIC_APP_URL } from '$env/dynamic/private';
import { db } from './db';
import { verificationTokens } from './db/schema';
import { randomBytes } from 'crypto';

const resend = new Resend(RESEND_API_KEY);

export async function sendVerificationEmail(email: string, userId: string) {
	// Generate verification token
	const token = randomBytes(32).toString('hex');
	const expires = new Date(Date.now() + 24 * 60 * 60 * 1000); // 24 hours

	// Store token in database
	await db.insert(verificationTokens).values({
		id: randomBytes(16).toString('hex'),
		identifier: userId,
		token,
		expires
	});

	const verificationUrl = `${PUBLIC_APP_URL}/verify-email?token=${token}`;

	// Send email
	await resend.emails.send({
		from: 'noreply@yourdomain.com',
		to: email,
		subject: 'Verify your email address',
		html: `
			<div style="font-family: sans-serif; max-width: 600px; margin: 0 auto;">
				<h1>Verify Your Email</h1>
				<p>Thanks for signing up! Please verify your email address by clicking the button below:</p>
				<a
					href="${verificationUrl}"
					style="display: inline-block; background: #6366f1; color: white; padding: 12px 24px; text-decoration: none; border-radius: 6px; margin: 16px 0;"
				>
					Verify Email
				</a>
				<p style="color: #666; font-size: 14px;">
					This link will expire in 24 hours. If you didn't create an account, you can safely ignore this email.
				</p>
			</div>
		`
	});
}
```

**Verify email route:**

```typescript
// src/routes/verify-email/+page.server.ts
import { redirect, error } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import { verificationTokens, users } from '$lib/server/db/schema';
import { eq, and, gt } from 'drizzle-orm';
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ url }) => {
	const token = url.searchParams.get('token');

	if (!token) {
		throw error(400, 'Missing verification token');
	}

	// Find token
	const [verificationToken] = await db
		.select()
		.from(verificationTokens)
		.where(and(eq(verificationTokens.token, token), gt(verificationTokens.expires, new Date())))
		.limit(1);

	if (!verificationToken) {
		throw error(400, 'Invalid or expired verification token');
	}

	// Update user email verified status
	await db
		.update(users)
		.set({ emailVerified: true })
		.where(eq(users.id, verificationToken.identifier));

	// Delete used token
	await db.delete(verificationTokens).where(eq(verificationTokens.id, verificationToken.id));

	throw redirect(303, '/login?verified=true');
};
```

> 💡 **Best Practice**: Always expire verification tokens after a reasonable time period.

---

## 5. Populating Locals with Session

### Making User Available Everywhere

Add session data to `locals` in hooks.

```typescript
// src/hooks.server.ts
import { auth } from '$lib/server/auth';
import { sequence } from '@sveltejs/kit/hooks';
import type { Handle } from '@sveltejs/kit';

const handleAuth: Handle = async ({ event, resolve }) => {
	// Get session from Better Auth
	const session = await auth.api.getSession({
		headers: event.request.headers
	});

	if (session) {
		event.locals.user = session.user;
		event.locals.session = session.session;
	} else {
		event.locals.user = null;
		event.locals.session = null;
	}

	return resolve(event);
};

export const handle = sequence(handleAuth);
```

**Type definitions:**

```typescript
// src/app.d.ts
import type { User, Session } from 'better-auth';

declare global {
	namespace App {
		interface Locals {
			user: User | null;
			session: Session | null;
		}
	}
}

export {};
```

Now `locals.user` is available in all load functions and actions!

---

## 6. Login & Logout Actions

### Session Management

Handle login and logout flows.

```typescript
// src/routes/(auth)/login/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import { auth } from '$lib/server/auth';
import type { Actions } from './$types';

export const actions: Actions = {
	default: async ({ request, cookies }) => {
		const data = await request.formData();
		const email = data.get('email')?.toString();
		const password = data.get('password')?.toString();

		if (!email || !password) {
			return fail(400, { error: 'Email and password are required' });
		}

		try {
			const result = await auth.api.signIn.email({
				body: {
					email,
					password
				}
			});

			if (!result.user) {
				return fail(400, { error: 'Invalid email or password' });
			}

			// Check if email is verified
			if (!result.user.emailVerified) {
				return fail(400, {
					error: 'Please verify your email before signing in'
				});
			}

			// Set session cookie
			cookies.set('better-auth.session_token', result.token, {
				path: '/',
				httpOnly: true,
				sameSite: 'lax',
				secure: process.env.NODE_ENV === 'production',
				maxAge: 60 * 60 * 24 * 7 // 7 days
			});

			throw redirect(303, '/workspaces');
		} catch (error: any) {
			return fail(400, { error: 'Invalid email or password' });
		}
	}
};
```

```typescript
// src/routes/logout/+server.ts
import { redirect } from '@sveltejs/kit';
import { auth } from '$lib/server/auth';
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ cookies, locals }) => {
	if (locals.session) {
		await auth.api.signOut({
			headers: {
				authorization: `Bearer ${locals.session.token}`
			}
		});
	}

	cookies.delete('better-auth.session_token', { path: '/' });

	throw redirect(303, '/login');
};
```

---

## 7. OAuth with GitHub

### Social Authentication

Enable GitHub OAuth for easy sign-in.

**Setup GitHub OAuth App:**

1. Go to GitHub Settings → Developer settings → OAuth Apps
2. Create new OAuth App
3. Set callback URL: `http://localhost:5173/api/auth/github/callback`
4. Copy Client ID and Secret to `.env`

**Better Auth handles the OAuth flow automatically:**

```typescript
// src/routes/api/auth/github/+server.ts
import { auth } from '$lib/server/auth';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async (event) => {
	return auth.api.signIn.social({
		provider: 'github',
		request: event.request
	});
};
```

**Callback route (Better Auth handles automatically):**

```typescript
// src/routes/api/auth/github/callback/+server.ts
import { auth } from '$lib/server/auth';
import { redirect } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async (event) => {
	const result = await auth.api.callback.social({
		provider: 'github',
		request: event.request
	});

	if (result.user) {
		// Set session cookie
		event.cookies.set('better-auth.session_token', result.token, {
			path: '/',
			httpOnly: true,
			sameSite: 'lax',
			secure: process.env.NODE_ENV === 'production',
			maxAge: 60 * 60 * 24 * 7
		});

		throw redirect(303, '/workspaces');
	}

	throw redirect(303, '/login?error=oauth_failed');
};
```

> 💡 **Best Practice**: Always handle OAuth errors gracefully and provide feedback.

---

## 8. Authorization Basics

### Protecting Routes

Implement simple route protection.

```typescript
// src/lib/server/guards.ts
import { redirect, error } from '@sveltejs/kit';
import type { RequestEvent } from '@sveltejs/kit';

export function requireAuth(event: RequestEvent) {
	if (!event.locals.user) {
		throw redirect(303, `/login?redirect=${event.url.pathname}`);
	}
	return event.locals.user;
}

export function requireEmailVerified(event: RequestEvent) {
	const user = requireAuth(event);

	if (!user.emailVerified) {
		throw error(403, 'Please verify your email to access this page');
	}

	return user;
}
```

**Use in load functions:**

```typescript
// src/routes/(app)/workspaces/+layout.server.ts
import { requireAuth } from '$lib/server/guards';
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async (event) => {
	const user = requireAuth(event);

	// User is authenticated, continue loading
	return { user };
};
```

---

## 9. Permission Management with CASL

### Role-Based Access Control

Use CASL for fine-grained permissions.

**Installation:**

```bash
npm install @casl/ability
```

**Define abilities:**

```typescript
// src/lib/server/permissions.ts
import { AbilityBuilder, createMongoAbility, type MongoAbility } from '@casl/ability';
import type { User } from 'better-auth';

type Actions = 'create' | 'read' | 'update' | 'delete' | 'manage';
type Subjects = 'Workspace' | 'Page' | 'WorkspaceMember' | 'all';

export type AppAbility = MongoAbility<[Actions, Subjects]>;

export function defineAbilitiesFor(user: User, workspace?: any, membership?: any): AppAbility {
	const { can, cannot, build } = new AbilityBuilder<AppAbility>(createMongoAbility);

	if (!user) {
		// Unauthenticated users can't do anything
		return build();
	}

	// Everyone can read public workspaces
	can('read', 'Workspace', { isPublic: true });

	if (membership) {
		// Workspace member permissions
		const role = membership.role;

		// All members can read workspace content
		can('read', 'Workspace', { id: workspace.id });
		can('read', 'Page', { workspaceId: workspace.id });

		if (role === 'owner') {
			// Owners can do everything
			can('manage', 'all', { workspaceId: workspace.id });
			can('delete', 'Workspace', { id: workspace.id });
		} else if (role === 'admin') {
			// Admins can manage content but not delete workspace
			can('create', 'Page', { workspaceId: workspace.id });
			can('update', 'Page', { workspaceId: workspace.id });
			can('delete', 'Page', { workspaceId: workspace.id });
			can('update', 'Workspace', { id: workspace.id });
			can('create', 'WorkspaceMember', { workspaceId: workspace.id });
			can('update', 'WorkspaceMember', { workspaceId: workspace.id });
		} else if (role === 'member') {
			// Members can create and edit their own pages
			can('create', 'Page', { workspaceId: workspace.id });
			can('update', 'Page', { workspaceId: workspace.id, authorId: user.id });
			can('delete', 'Page', { workspaceId: workspace.id, authorId: user.id });
		} else if (role === 'viewer') {
			// Viewers can only read
			can('read', 'Workspace', { id: workspace.id });
			can('read', 'Page', { workspaceId: workspace.id });
		}
	}

	return build();
}
```

**Helper function to check access:**

```typescript
// src/lib/server/access.ts
import { error } from '@sveltejs/kit';
import { db } from './db';
import { workspaces, workspaceMembers, pages } from './db/schema';
import { eq, and } from 'drizzle-orm';
import { defineAbilitiesFor } from './permissions';
import type { RequestEvent } from '@sveltejs/kit';

export async function checkWorkspaceAccess(
	event: RequestEvent,
	workspaceSlug: string,
	action: string = 'read'
) {
	if (!event.locals.user) {
		throw error(401, 'You must be logged in');
	}

	// Get workspace
	const [workspace] = await db
		.select()
		.from(workspaces)
		.where(eq(workspaces.slug, workspaceSlug))
		.limit(1);

	if (!workspace) {
		throw error(404, 'Workspace not found');
	}

	// Get membership
	const [membership] = await db
		.select()
		.from(workspaceMembers)
		.where(
			and(
				eq(workspaceMembers.workspaceId, workspace.id),
				eq(workspaceMembers.userId, event.locals.user.id)
			)
		)
		.limit(1);

	// Define abilities
	const ability = defineAbilitiesFor(event.locals.user, workspace, membership);

	// Check permission
	if (!ability.can(action as any, 'Workspace', workspace)) {
		throw error(403, 'You do not have permission to perform this action');
	}

	return { workspace, membership, ability };
}

export async function checkPageAccess(
	event: RequestEvent,
	workspaceSlug: string,
	pageId: number,
	action: string = 'read'
) {
	const { workspace, membership, ability } = await checkWorkspaceAccess(
		event,
		workspaceSlug,
		'read'
	);

	// Get page
	const [page] = await db
		.select()
		.from(pages)
		.where(and(eq(pages.id, pageId), eq(pages.workspaceId, workspace.id)))
		.limit(1);

	if (!page) {
		throw error(404, 'Page not found');
	}

	// Check page permission
	if (!ability.can(action as any, 'Page', page)) {
		throw error(403, 'You do not have permission to perform this action');
	}

	return { workspace, membership, page, ability };
}
```

---

## 10. Complete Example: Multi-Tenant Platform with Auth

### Full Authentication & Authorization System

Let's build a complete authenticated multi-tenant workspace platform!

**Features:**

- ✅ Email/password registration with verification
- ✅ GitHub OAuth
- ✅ Session management
- ✅ Protected routes
- ✅ Role-based permissions (owner, admin, member, viewer)
- ✅ Workspace access control
- ✅ Page-level permissions
- ✅ Settings protection

### Protected Workspace Layout

```typescript
// src/routes/(app)/workspaces/[slug]/+layout.server.ts
import { checkWorkspaceAccess } from '$lib/server/access';
import { db } from '$lib/server/db';
import { pages } from '$lib/server/db/schema';
import { eq } from 'drizzle-orm';
import type { LayoutServerLoad } from './$types';

export const load: LayoutServerLoad = async (event) => {
	const { workspace, membership, ability } = await checkWorkspaceAccess(event, event.params.slug);

	// Load workspace pages
	const workspacePages = await db
		.select()
		.from(pages)
		.where(eq(pages.workspaceId, workspace.id))
		.orderBy(pages.updatedAt);

	return {
		workspace,
		pages: workspacePages,
		userRole: membership?.role || null,
		permissions: {
			canCreatePage: ability.can('create', 'Page', { workspaceId: workspace.id }),
			canUpdateWorkspace: ability.can('update', 'Workspace', workspace),
			canDeleteWorkspace: ability.can('delete', 'Workspace', workspace)
		}
	};
};
```

### Protected Workspace Settings

```typescript
// src/routes/(app)/workspaces/[slug]/settings/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import { checkWorkspaceAccess } from '$lib/server/access';
import { db } from '$lib/server/db';
import { workspaces } from '$lib/server/db/schema';
import { eq } from 'drizzle-orm';
import type { Actions, PageServerLoad } from './$types';

export const load: PageServerLoad = async (event) => {
	// Require update permission
	const { workspace, ability } = await checkWorkspaceAccess(event, event.params.slug, 'update');

	return {
		workspace,
		canDelete: ability.can('delete', 'Workspace', workspace)
	};
};

export const actions: Actions = {
	update: async (event) => {
		const { workspace } = await checkWorkspaceAccess(event, event.params.slug, 'update');

		const data = await event.request.formData();
		const name = data.get('name')?.toString();
		const description = data.get('description')?.toString();

		if (!name) {
			return fail(400, { error: 'Name is required' });
		}

		await db
			.update(workspaces)
			.set({
				name,
				description,
				updatedAt: new Date()
			})
			.where(eq(workspaces.id, workspace.id));

		return { success: true };
	},

	delete: async (event) => {
		const { workspace } = await checkWorkspaceAccess(event, event.params.slug, 'delete');

		await db.delete(workspaces).where(eq(workspaces.id, workspace.id));

		throw redirect(303, '/workspaces');
	}
};
```

### Protected Page Actions

```typescript
// src/routes/(app)/workspaces/[slug]/pages/[pageId]/edit/+page.server.ts
import { fail, redirect } from '@sveltejs/kit';
import { checkPageAccess } from '$lib/server/access';
import { db } from '$lib/server/db';
import { pages } from '$lib/server/db/schema';
import { eq } from 'drizzle-orm';
import type { Actions, PageServerLoad } from './$types';

export const load: PageServerLoad = async (event) => {
	const { page } = await checkPageAccess(
		event,
		event.params.slug,
		parseInt(event.params.pageId),
		'update'
	);

	return { page };
};

export const actions: Actions = {
	default: async (event) => {
		const { page, workspace } = await checkPageAccess(
			event,
			event.params.slug,
			parseInt(event.params.pageId),
			'update'
		);

		const data = await event.request.formData();
		const title = data.get('title')?.toString();
		const content = data.get('content')?.toString();

		if (!title) {
			return fail(400, { error: 'Title is required' });
		}

		await db
			.update(pages)
			.set({
				title,
				content,
				updatedAt: new Date()
			})
			.where(eq(pages.id, page.id));

		throw redirect(303, `/workspaces/${workspace.slug}/pages/${page.id}`);
	}
};
```

### Conditional UI Based on Permissions

```svelte
<!-- src/routes/(app)/workspaces/[slug]/+layout.svelte -->
<script lang="ts">
	import type { Snippet } from 'svelte';
	import type { LayoutData } from './$types';

	let { data, children }: { data: LayoutData; children: Snippet } = $props();
</script>

<div class="workspace-layout">
	<header class="navbar bg-base-200">
		<div class="flex-1">
			<h1 class="text-xl font-bold">{data.workspace.name}</h1>
			<div class="badge badge-outline ml-2">{data.userRole || 'Viewer'}</div>
		</div>
		<div class="flex-none gap-2">
			{#if data.permissions.canCreatePage}
				<a href="/workspaces/{data.workspace.slug}/pages/new" class="btn btn-primary btn-sm">
					+ New Page
				</a>
			{/if}

			{#if data.permissions.canUpdateWorkspace}
				<a href="/workspaces/{data.workspace.slug}/settings" class="btn btn-ghost btn-sm">
					Settings
				</a>
			{/if}
		</div>
	</header>

	<main class="p-6">
		{@render children()}
	</main>
</div>
```

**Key Security Features:**

- ✅ **Authentication**: Email/password + OAuth
- ✅ **Email Verification**: Resend integration
- ✅ **Session Management**: Secure cookies
- ✅ **Route Protection**: Guard functions
- ✅ **Permission Checks**: CASL abilities
- ✅ **Workspace Access**: Membership validation
- ✅ **Page Permissions**: Author-based access
- ✅ **Conditional UI**: Show/hide based on permissions
- ✅ **Action Protection**: Verify permissions in actions

---

## 📝 Key Takeaways

✅ Better Auth provides modern authentication  
✅ Always verify emails before full access  
✅ Store session in `locals` via hooks  
✅ Protect routes with guard functions  
✅ Use CASL for fine-grained permissions  
✅ Check permissions in both load functions and actions  
✅ Implement role-based access control  
✅ Always validate user has access to resources  
✅ Use OAuth for easier sign-in  
✅ Never expose auth secrets to client  
✅ Expire verification tokens  
✅ Show conditional UI based on permissions

---

## 🚀 Next Steps

You've mastered authentication and authorization! Next up:

- **Section 18**: File Uploads & Storage
- **Section 19**: Real-time Features with WebSockets
- Advanced permission patterns
- Two-factor authentication
- Session management strategies
