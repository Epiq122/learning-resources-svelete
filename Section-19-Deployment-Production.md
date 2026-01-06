# Section 19: Deployment & Production

## 📚 Learning Objectives

By the end of this section, you will:

- Understand SvelteKit adapters and their use cases
- Set up production databases (Neon PostgreSQL)
- Configure and test Node adapter locally
- Deploy to VPS platforms (fly.io)
- Deploy to serverless platforms (Vercel)
- Manage environment variables in production
- Configure production-ready applications
- Choose the right deployment strategy

---

## Table of Contents

- [Section 19: Deployment \& Production](#section-19-deployment--production)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Deployment \& Adapters Overview](#1-deployment--adapters-overview)
    - [Understanding Adapter Types](#understanding-adapter-types)
  - [2. Setting Up a Neon Database](#2-setting-up-a-neon-database)
    - [Production PostgreSQL in the Cloud](#production-postgresql-in-the-cloud)
  - [3. Configuring the Node Adapter Locally](#3-configuring-the-node-adapter-locally)
    - [Testing Production Build](#testing-production-build)
  - [4. Deploying to fly.io](#4-deploying-to-flyio)
    - [VPS Deployment with Docker](#vps-deployment-with-docker)
  - [5. Deploying to Vercel](#5-deploying-to-vercel)
    - [Serverless Deployment](#serverless-deployment)
  - [6. Complete Example: Production-Ready Setup](#6-complete-example-production-ready-setup)
    - [Full Deployment Configuration](#full-deployment-configuration)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Deployment & Adapters Overview

### Understanding Adapter Types

SvelteKit adapters configure your app for different deployment targets.

**Available Adapters:**

| Adapter                        | Use Case                  | Platform Examples           |
| ------------------------------ | ------------------------- | --------------------------- |
| `@sveltejs/adapter-auto`       | Automatic detection       | Vercel, Netlify, Cloudflare |
| `@sveltejs/adapter-node`       | Node.js server            | VPS, fly.io, Railway        |
| `@sveltejs/adapter-static`     | Static sites only         | GitHub Pages, S3            |
| `@sveltejs/adapter-vercel`     | Vercel-specific features  | Vercel                      |
| `@sveltejs/adapter-netlify`    | Netlify-specific features | Netlify                     |
| `@sveltejs/adapter-cloudflare` | Cloudflare Workers        | Cloudflare Pages            |

**Choosing an adapter:**

```typescript
// svelte.config.js

// 1. Auto (recommended for most cases)
import adapter from '@sveltejs/adapter-auto';

// 2. Node (VPS, Docker, traditional hosting)
import adapter from '@sveltejs/adapter-node';

// 3. Static (pre-rendered only, no SSR)
import adapter from '@sveltejs/adapter-static';

// 4. Vercel (optimized for Vercel)
import adapter from '@sveltejs/adapter-vercel';

const config = {
	kit: {
		adapter: adapter()
	}
};

export default config;
```

**Comparison:**

```typescript
// adapter-node: Full Node.js server
// ✅ SSR, API routes, WebSockets
// ✅ Full control over server
// ✅ Works on any VPS
// ❌ You manage scaling

// adapter-vercel: Serverless functions
// ✅ Auto-scaling
// ✅ Edge network
// ✅ Zero config
// ❌ Function limits (size, duration)

// adapter-static: Static HTML only
// ✅ Cheapest hosting
// ✅ Fastest performance
// ✅ Works anywhere
// ❌ No SSR, no API routes (must pre-render everything)
```

> 💡 **Best Practice**: Use `adapter-auto` for flexibility, switch to specific adapter when you need platform-specific features.

---

## 2. Setting Up a Neon Database

### Production PostgreSQL in the Cloud

Neon provides serverless PostgreSQL with automatic scaling.

**Sign up for Neon:**

1. Go to [neon.tech](https://neon.tech)
2. Create account (GitHub OAuth)
3. Create new project

**Create database:**

```bash
# Via Neon Dashboard
1. Click "New Project"
2. Name: "my-app-production"
3. Region: Choose closest to your users
4. Plan: Free tier (great for starting)

# Copy connection string
postgresql://username:password@ep-xxx.region.aws.neon.tech/neondb?sslmode=require
```

**Configure environment variables:**

```env
# .env.production (DO NOT COMMIT)
DATABASE_URL=postgresql://username:password@ep-xxx.region.aws.neon.tech/neondb?sslmode=require
```

**Run migrations on production database:**

```bash
# Install Drizzle Kit
npm install -D drizzle-kit

# Generate migration SQL
npm run db:generate

# Push to production (one-time setup)
DATABASE_URL="your-neon-url" npm run db:push

# Or run migrations
DATABASE_URL="your-neon-url" npm run db:migrate
```

**Drizzle configuration for Neon:**

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';
import { config } from 'dotenv';

config(); // Load .env

export default {
	schema: './src/lib/server/db/schema.ts',
	out: './drizzle',
	driver: 'pg',
	dbCredentials: {
		connectionString: process.env.DATABASE_URL!
	},
	verbose: true,
	strict: true
} satisfies Config;
```

**Connection pooling for Neon:**

```typescript
// src/lib/server/db/index.ts
import { drizzle } from 'drizzle-orm/postgres-js';
import postgres from 'postgres';
import { DATABASE_URL } from '$env/dynamic/private';
import * as schema from './schema';

// Neon connection with pooling
const client = postgres(DATABASE_URL!, {
	max: 10, // Max connections
	idle_timeout: 20,
	connect_timeout: 10
});

export const db = drizzle(client, { schema });
```

**Neon-specific optimizations:**

```typescript
// Use connection pooler for serverless
const pooledUrl = DATABASE_URL!.replace('.aws.neon.tech', '-pooler.aws.neon.tech');

const client = postgres(pooledUrl, {
	prepare: false // Required for pooled connections
});
```

> 💡 **Best Practice**: Use Neon's connection pooler for serverless deployments (Vercel, Netlify).

---

## 3. Configuring the Node Adapter Locally

### Testing Production Build

Test your production build locally before deploying.

**Install Node adapter:**

```bash
npm install -D @sveltejs/adapter-node
```

**Configure adapter:**

```typescript
// svelte.config.js
import adapter from '@sveltejs/adapter-node';

const config = {
	kit: {
		adapter: adapter({
			// Output directory
			out: 'build',

			// Precompress files
			precompress: true,

			// Environment variable prefix
			envPrefix: '',

			// Polyfill fetch for Node < 18
			polyfill: false
		})
	}
};

export default config;
```

**Build for production:**

```bash
npm run build
```

**Output structure:**

```
build/
├── index.js          # Server entry point
├── handler.js        # Request handler
├── client/           # Static assets
│   ├── _app/
│   └── index.html
└── server/
    └── chunks/
```

**Run production build:**

```bash
# Set production environment
export NODE_ENV=production
export DATABASE_URL=your-database-url
export ORIGIN=http://localhost:3000

# Start server
node build/index.js

# Or with package.json script
npm run preview
```

**Production server script:**

```json
// package.json
{
	"scripts": {
		"dev": "vite dev",
		"build": "vite build",
		"preview": "node build/index.js",
		"start": "NODE_ENV=production node build/index.js"
	}
}
```

**Custom server configuration:**

```javascript
// server.js (optional)
import { handler } from './build/handler.js';
import express from 'express';

const app = express();
const port = process.env.PORT || 3000;

// Custom middleware
app.use((req, res, next) => {
	console.log(`${req.method} ${req.path}`);
	next();
});

// SvelteKit handler
app.use(handler);

app.listen(port, () => {
	console.log(`Server running on port ${port}`);
});
```

**Environment variables:**

```env
# .env.production
NODE_ENV=production
DATABASE_URL=postgresql://...
ORIGIN=https://yourdomain.com
PORT=3000
BODY_SIZE_LIMIT=5242880
```

> ⚠️ **Common Mistake**: Forgetting to set `ORIGIN` in production (required for CSRF protection).

---

## 4. Deploying to fly.io

### VPS Deployment with Docker

Fly.io provides global VPS with automatic SSL and easy deployment.

**Install Fly CLI:**

```bash
# macOS
brew install flyctl

# Linux
curl -L https://fly.io/install.sh | sh

# Windows
powershell -Command "iwr https://fly.io/install.ps1 -useb | iex"
```

**Login:**

```bash
flyctl auth login
```

**Create Dockerfile:**

```dockerfile
# Dockerfile
FROM node:20-alpine AS builder

WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm ci

# Copy source
COPY . .

# Build app
RUN npm run build

# Remove dev dependencies
RUN npm prune --production

# Production stage
FROM node:20-alpine

WORKDIR /app

# Copy built app and dependencies
COPY --from=builder /app/build ./build
COPY --from=builder /app/node_modules ./node_modules
COPY --from=builder /app/package.json ./

# Expose port
EXPOSE 3000

# Start app
CMD ["node", "build/index.js"]
```

**.dockerignore:**

```
node_modules
.svelte-kit
build
.env
.env.*
!.env.example
.git
.vscode
*.log
```

**Initialize Fly app:**

```bash
flyctl launch
```

This creates `fly.toml`:

```toml
# fly.toml
app = "my-sveltekit-app"
primary_region = "iad"

[build]
  dockerfile = "Dockerfile"

[env]
  NODE_ENV = "production"
  PORT = "8080"

[http_service]
  internal_port = 8080
  force_https = true
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 0

[[services]]
  protocol = "tcp"
  internal_port = 8080

  [[services.ports]]
    port = 80
    handlers = ["http"]

  [[services.ports]]
    port = 443
    handlers = ["tls", "http"]

[[services.http_checks]]
  interval = 10000
  timeout = 2000
  grace_period = "5s"
  method = "GET"
  path = "/api/health"
```

**Set secrets:**

```bash
# Set environment variables as secrets
flyctl secrets set DATABASE_URL="postgresql://..."
flyctl secrets set GITHUB_CLIENT_ID="your_id"
flyctl secrets set GITHUB_CLIENT_SECRET="your_secret"
flyctl secrets set RESEND_API_KEY="your_key"

# List secrets
flyctl secrets list
```

**Deploy:**

```bash
# Deploy to Fly
flyctl deploy

# View logs
flyctl logs

# SSH into machine
flyctl ssh console

# Check status
flyctl status

# Scale machines
flyctl scale count 2
```

**Custom health check:**

```typescript
// src/routes/api/health/+server.ts
import { json } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async () => {
	try {
		// Check database connection
		await db.execute('SELECT 1');

		return json({
			status: 'healthy',
			timestamp: new Date().toISOString(),
			database: 'connected'
		});
	} catch (error) {
		return json(
			{
				status: 'unhealthy',
				timestamp: new Date().toISOString(),
				database: 'disconnected',
				error: error.message
			},
			{ status: 503 }
		);
	}
};
```

**Zero-downtime deployments:**

```bash
# Deploy with health checks
flyctl deploy --ha=false

# Or configure in fly.toml
[http_service]
  auto_stop_machines = true
  auto_start_machines = true
  min_machines_running = 1  # Keep 1 running always
```

> 💡 **Best Practice**: Use Fly's connection pooler or set up your own with PgBouncer for database connections.

---

## 5. Deploying to Vercel

### Serverless Deployment

Vercel provides zero-config deployment with automatic scaling.

**Install Vercel CLI:**

```bash
npm install -g vercel
```

**Configure adapter:**

```typescript
// svelte.config.js
import adapter from '@sveltejs/adapter-vercel';

const config = {
	kit: {
		adapter: adapter({
			// Runtime for serverless functions
			runtime: 'nodejs20.x',

			// Edge functions for specific routes
			edge: false,

			// External packages (not bundled)
			external: [],

			// ISR (Incremental Static Regeneration)
			isr: {
				expiration: 60 // Revalidate after 60 seconds
			}
		})
	}
};

export default config;
```

**Deploy from CLI:**

```bash
# Login
vercel login

# Deploy to preview
vercel

# Deploy to production
vercel --prod
```

**Deploy from GitHub:**

1. Push code to GitHub
2. Import project in Vercel dashboard
3. Configure build settings (auto-detected)
4. Add environment variables
5. Deploy

**Environment variables in Vercel:**

```bash
# Via CLI
vercel env add DATABASE_URL
vercel env add GITHUB_CLIENT_ID
vercel env add GITHUB_CLIENT_SECRET

# Via Dashboard
Project Settings → Environment Variables
```

**Vercel configuration file:**

```json
// vercel.json (optional)
{
	"buildCommand": "npm run build",
	"devCommand": "npm run dev",
	"installCommand": "npm install",
	"framework": "sveltekit",
	"outputDirectory": ".vercel/output",
	"regions": ["iad1"],
	"env": {
		"NODE_ENV": "production"
	},
	"functions": {
		"api/**": {
			"memory": 1024,
			"maxDuration": 10
		}
	}
}
```

**Edge functions for specific routes:**

```typescript
// src/routes/api/fast/+server.ts
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const config = {
	runtime: 'edge' // Run on Edge network
};

export const GET: RequestHandler = async () => {
	return json({
		message: 'This runs on Vercel Edge Network',
		timestamp: Date.now()
	});
};
```

**ISR (Incremental Static Regeneration):**

```typescript
// src/routes/blog/[slug]/+page.server.ts
import type { PageServerLoad } from './$types';

export const config = {
	isr: {
		expiration: 60 // Revalidate every 60 seconds
	}
};

export const load: PageServerLoad = async ({ params }) => {
	// This page is cached and regenerated every 60 seconds
	const post = await fetchPost(params.slug);
	return { post };
};
```

**Cron jobs on Vercel:**

```json
// vercel.json
{
	"crons": [
		{
			"path": "/api/cron/cleanup",
			"schedule": "0 0 * * *"
		}
	]
}
```

```typescript
// src/routes/api/cron/cleanup/+server.ts
import { json } from '@sveltejs/kit';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ request }) => {
	// Verify cron secret
	const authHeader = request.headers.get('authorization');
	if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
		return json({ error: 'Unauthorized' }, { status: 401 });
	}

	// Cleanup logic
	await cleanupOldData();

	return json({ success: true });
};
```

> 💡 **Best Practice**: Use Neon's pooled connection string on Vercel to avoid connection limits.

---

## 6. Complete Example: Production-Ready Setup

### Full Deployment Configuration

Let's configure a production-ready application for multiple deployment targets!

**Features:**

- ✅ Environment-specific configuration
- ✅ Production database (Neon)
- ✅ Health checks and monitoring
- ✅ Error logging (Sentry)
- ✅ CDN for static assets
- ✅ Deployment scripts
- ✅ CI/CD pipeline

### 📁 Project Structure

```
.
├── .github/
│   └── workflows/
│       ├── deploy-fly.yml
│       └── deploy-vercel.yml
├── src/
├── Dockerfile
├── fly.toml
├── vercel.json
├── .env.example
└── package.json
```

### Environment Configuration

```env
# .env.example (commit this)
# Copy to .env and .env.production

# Database
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb

# Auth
GITHUB_CLIENT_ID=your_github_id
GITHUB_CLIENT_SECRET=your_github_secret
BETTER_AUTH_SECRET=your_secret_key

# Email
RESEND_API_KEY=your_resend_key

# App
PUBLIC_APP_URL=http://localhost:5173
NODE_ENV=development
PORT=3000

# Monitoring (optional)
SENTRY_DSN=your_sentry_dsn
```

### Multi-adapter Configuration

```typescript
// svelte.config.js
import adapter from '@sveltejs/adapter-auto';
import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

const isVercel = process.env.VERCEL === '1';
const isFly = process.env.FLY_APP_NAME !== undefined;

const config = {
	preprocess: vitePreprocess(),

	kit: {
		adapter: adapter(),

		env: {
			publicPrefix: 'PUBLIC_'
		},

		csrf: {
			checkOrigin: true
		},

		// Rate limiting
		inlineStyleThreshold: 1024,

		alias: {
			$lib: './src/lib',
			$components: './src/lib/components'
		}
	}
};

export default config;
```

### Production Hooks

```typescript
// src/hooks.server.ts
import { sequence } from '@sveltejs/kit/hooks';
import { auth } from '$lib/server/auth';
import { handleErrorWithSentry, sentryHandle } from '@sentry/sveltekit';
import * as Sentry from '@sentry/sveltekit';
import type { Handle, HandleServerError } from '@sveltejs/kit';

// Initialize Sentry in production
if (process.env.NODE_ENV === 'production') {
	Sentry.init({
		dsn: process.env.SENTRY_DSN,
		tracesSampleRate: 1.0,
		environment: process.env.VERCEL ? 'vercel' : 'fly'
	});
}

// Auth handler
const handleAuth: Handle = async ({ event, resolve }) => {
	const session = await auth.api.getSession({
		headers: event.request.headers
	});

	event.locals.user = session?.user || null;
	event.locals.session = session?.session || null;

	return resolve(event);
};

// Logging handler
const handleLogging: Handle = async ({ event, resolve }) => {
	const start = Date.now();

	const response = await resolve(event);

	const duration = Date.now() - start;

	console.log(`${event.request.method} ${event.url.pathname} - ${response.status} (${duration}ms)`);

	return response;
};

// Security headers
const handleSecurity: Handle = async ({ event, resolve }) => {
	const response = await resolve(event);

	response.headers.set('X-Frame-Options', 'SAMEORIGIN');
	response.headers.set('X-Content-Type-Options', 'nosniff');
	response.headers.set('Referrer-Policy', 'strict-origin-when-cross-origin');
	response.headers.set('Permissions-Policy', 'camera=(), microphone=(), geolocation=()');

	return response;
};

export const handle = sequence(sentryHandle(), handleAuth, handleLogging, handleSecurity);

export const handleError = handleErrorWithSentry();
```

### GitHub Actions - Fly.io

```yaml
# .github/workflows/deploy-fly.yml
name: Deploy to Fly.io

on:
  push:
    branches: [main]

jobs:
  deploy:
    name: Deploy app
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - uses: superfly/flyctl-actions/setup-flyctl@master

      - name: Deploy to Fly
        run: flyctl deploy --remote-only
        env:
          FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```

### GitHub Actions - Vercel

```yaml
# .github/workflows/deploy-vercel.yml
name: Deploy to Vercel

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy to Vercel
        run: vercel deploy --prebuilt --prod --token=${{ secrets.VERCEL_TOKEN }}
```

### Health Check Endpoint

```typescript
// src/routes/api/health/+server.ts
import { json } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import { version } from '$app/environment';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async () => {
	const checks: Record<string, any> = {
		status: 'healthy',
		timestamp: new Date().toISOString(),
		version,
		uptime: process.uptime(),
		memory: process.memoryUsage()
	};

	// Database check
	try {
		await db.execute('SELECT 1');
		checks.database = 'connected';
	} catch (error) {
		checks.database = 'disconnected';
		checks.status = 'unhealthy';
		return json(checks, { status: 503 });
	}

	return json(checks);
};
```

### Package.json Scripts

```json
{
	"name": "my-sveltekit-app",
	"version": "1.0.0",
	"scripts": {
		"dev": "vite dev",
		"build": "vite build",
		"preview": "vite preview",
		"start": "node build/index.js",
		"check": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json",
		"check:watch": "svelte-kit sync && svelte-check --tsconfig ./tsconfig.json --watch",
		"lint": "eslint .",
		"format": "prettier --write .",
		"db:generate": "drizzle-kit generate:pg",
		"db:push": "drizzle-kit push:pg",
		"db:migrate": "tsx src/lib/server/db/migrate.ts",
		"db:seed": "tsx src/lib/server/db/seed.ts",
		"deploy:fly": "flyctl deploy",
		"deploy:vercel": "vercel --prod"
	},
	"dependencies": {
		"@sveltejs/kit": "^2.0.0",
		"svelte": "^5.0.0",
		"drizzle-orm": "^0.30.0",
		"postgres": "^3.4.0",
		"better-auth": "^1.0.0"
	},
	"devDependencies": {
		"@sveltejs/adapter-auto": "^3.0.0",
		"@sveltejs/adapter-node": "^5.0.0",
		"@sveltejs/adapter-vercel": "^5.0.0",
		"@sveltejs/vite-plugin-svelte": "^4.0.0",
		"drizzle-kit": "^0.20.0",
		"vite": "^5.0.0"
	}
}
```

### Pre-deployment Checklist

```markdown
## Deployment Checklist

### Database

- [ ] Production database created (Neon/Supabase)
- [ ] Migrations run on production database
- [ ] Connection pooling configured
- [ ] Backups enabled

### Environment Variables

- [ ] All secrets added to hosting platform
- [ ] `ORIGIN` set to production URL
- [ ] `NODE_ENV=production`
- [ ] Database URL uses pooled connection

### Security

- [ ] CSRF protection enabled
- [ ] Security headers configured
- [ ] Rate limiting implemented
- [ ] Input validation on all endpoints

### Performance

- [ ] Static assets pre-compressed
- [ ] Database indexes created
- [ ] CDN configured (if applicable)
- [ ] Caching headers set

### Monitoring

- [ ] Error tracking (Sentry) configured
- [ ] Health check endpoint working
- [ ] Logging configured
- [ ] Analytics set up

### Testing

- [ ] Production build tested locally
- [ ] Database migrations tested
- [ ] Environment variables verified
- [ ] All features tested in staging
```

**Key Production Features:**

- ✅ **Multi-platform**: Works on Fly.io, Vercel, and other platforms
- ✅ **Database**: Neon PostgreSQL with connection pooling
- ✅ **Monitoring**: Sentry error tracking and health checks
- ✅ **Security**: CSRF protection, security headers
- ✅ **CI/CD**: Automated deployments with GitHub Actions
- ✅ **Logging**: Request logging and error handling
- ✅ **Performance**: Pre-compression and caching

---

## 📝 Key Takeaways

✅ Choose adapter based on deployment target  
✅ `adapter-auto` works for most platforms  
✅ `adapter-node` for VPS/Docker deployments  
✅ `adapter-vercel` for serverless with ISR  
✅ Use Neon for production PostgreSQL  
✅ Always use connection pooling  
✅ Set environment variables as secrets  
✅ Test production build locally first  
✅ Configure health check endpoints  
✅ Set up error monitoring (Sentry)  
✅ Use CI/CD for automated deployments  
✅ Always set `ORIGIN` in production

---

## 🚀 Next Steps

You've mastered deployment! Your app is production-ready!

**Further Topics:**

- **Monitoring**: Advanced observability with DataDog, New Relic
- **Scaling**: Load balancing, caching strategies
- **Security**: Rate limiting, DDoS protection
- **Performance**: Edge caching, CDN configuration
- **DevOps**: Infrastructure as Code, Kubernetes

**Congratulations on completing the SvelteKit course! 🎉**
