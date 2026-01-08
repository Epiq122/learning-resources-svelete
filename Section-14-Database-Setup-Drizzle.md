# Section 14: Database Setup with Drizzle & PostgreSQL

## 📚 Learning Objectives

By the end of this section, you will:

- Use environment variables securely in SvelteKit
- Create server-only modules for database access
- Set up PostgreSQL locally and with Docker
- Configure Drizzle ORM with type-safe schemas
- Create and run database migrations
- Seed databases with realistic test data
- Build a complete workspace management system

---

## Table of Contents

- [Section 14: Database Setup with Drizzle \& PostgreSQL](#section-14-database-setup-with-drizzle--postgresql)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Using Environment Variables](#1-using-environment-variables)
    - [Managing Secrets \& Configuration](#managing-secrets--configuration)
  - [2. Server Only Modules](#2-server-only-modules)
    - [Protecting Database Code](#protecting-database-code)
  - [3. Application \& Database Structure Overview](#3-application--database-structure-overview)
    - [Planning Your Schema](#planning-your-schema)
  - [4. Setting Up PostgreSQL Locally](#4-setting-up-postgresql-locally)
    - [Installing \& Running Postgres](#installing--running-postgres)
  - [5. Setting Up PostgreSQL with Docker](#5-setting-up-postgresql-with-docker)
    - [Containerized Database](#containerized-database)
  - [6. Configuring Drizzle ORM](#6-configuring-drizzle-orm)
    - [Type-Safe Database Access](#type-safe-database-access)
  - [7. Creating \& Migrating Schema](#7-creating--migrating-schema)
    - [Database Migrations](#database-migrations)
  - [8. Seeding the Database](#8-seeding-the-database)
    - [Adding Test Data](#adding-test-data)
  - [9. Complete Example: Workspace Management System](#9-complete-example-workspace-management-system)
    - [Full Database Setup \& Configuration](#full-database-setup--configuration)
    - [📁 Project Structure](#-project-structure)
    - [Environment Setup](#environment-setup)
    - [Docker Compose](#docker-compose)
    - [Database Queries](#database-queries)
    - [Health Check API](#health-check-api)
    - [Package Scripts](#package-scripts)
    - [Setup Instructions](#setup-instructions)
    - [Verify Setup](#verify-setup)
  - [📝 Key Takeaways](#-key-takeaways)
  - [🚀 Next Steps](#-next-steps)

---

## 1. Using Environment Variables

### Managing Secrets & Configuration

Environment variables keep secrets out of your code.

**Real-World Scenario:** Store database connection strings securely.

```bash
# .env (DO NOT commit this!)
DATABASE_URL="postgresql://user:password@localhost:5432/myapp"
DATABASE_AUTH_TOKEN="secret-token-here"
PUBLIC_API_URL="https://api.example.com"
```

```typescript
// src/lib/server/env.ts
import { env } from "$env/dynamic/private";
import { PUBLIC_API_URL } from "$env/static/public";

// Private env vars (server-only)
// NEVER sent to browser, safe for secrets
// Dynamic import allows runtime environment variable changes
export const DATABASE_URL = env.DATABASE_URL;
if (!DATABASE_URL) {
  // Fail fast at startup if critical config is missing
  // Better than cryptic database connection errors later
  throw new Error("DATABASE_URL is not set");
}

// Public env vars (available on client)
// These are embedded in client bundle during build
// NEVER put secrets in PUBLIC_* variables!
export { PUBLIC_API_URL };
```

```typescript
// src/lib/server/db.ts
import { DATABASE_URL } from "./env";

export const db = createConnection(DATABASE_URL);
```

**Environment variable types:**

| Import                 | Usage                   | Available |
| ---------------------- | ----------------------- | --------- |
| `$env/static/private`  | Server-only, build-time | Server    |
| `$env/dynamic/private` | Server-only, runtime    | Server    |
| `$env/static/public`   | Public, build-time      | Both      |
| `$env/dynamic/public`  | Public, runtime         | Both      |

> ⚠️ **Security**: Never put secrets in `PUBLIC_*` variables - they're sent to the browser!

**Different environments:**

```bash
# .env.development
DATABASE_URL="postgresql://localhost:5432/myapp_dev"

# .env.production
DATABASE_URL="postgresql://prod-server:5432/myapp_prod"

# .env.test
DATABASE_URL="postgresql://localhost:5432/myapp_test"
```

---

## 2. Server Only Modules

### Protecting Database Code

Files in `$lib/server` are **never** sent to the browser.

**Real-World Scenario:** Database credentials must stay on the server.

```
src/lib/
├── server/              # ✅ Server-only code
│   ├── db.ts           # Database instance
│   ├── schema.ts       # Drizzle schema
│   ├── queries.ts      # Database queries
│   └── auth.ts         # Auth logic
└── components/         # ❌ Can be sent to client
    └── Button.svelte
```

```typescript
// src/lib/server/db.ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import { DATABASE_URL } from "./env";

// This code ONLY runs on server
const client = postgres(DATABASE_URL);
export const db = drizzle(client);
```

```typescript
// src/routes/posts/+page.server.ts
import { db } from "$lib/server/db"; // ✅ Allowed in .server.ts

export const load = async () => {
  const posts = await db.select().from(postsTable);
  return { posts };
};
```

```typescript
// src/routes/posts/+page.ts
import { db } from "$lib/server/db"; // ❌ ERROR! Cannot import server code in universal load
```

> 💡 **Best Practice**: Keep ALL database code in `$lib/server/` to prevent accidental exposure.

---

## 3. Application & Database Structure Overview

### Planning Your Schema

Before coding, plan your database structure.

**Real-World Scenario:** Workspace management with users, workspaces, and pages.

**Entity Relationships:**

```
users
  ├── id (primary key)
  ├── email (unique)
  ├── name
  └── created_at

workspaces
  ├── id (primary key)
  ├── name
  ├── slug (unique)
  ├── owner_id → users.id
  └── created_at

workspace_members
  ├── workspace_id → workspaces.id
  ├── user_id → users.id
  ├── role (owner, admin, member)
  └── joined_at

pages
  ├── id (primary key)
  ├── workspace_id → workspaces.id
  ├── title
  ├── content
  ├── author_id → users.id
  ├── created_at
  └── updated_at
```

**Design decisions:**

- `users` table for authentication
- `workspaces` for multi-tenancy
- `workspace_members` for permissions (many-to-many)
- `pages` for content within workspaces
- All timestamps for audit trail

---

## 4. Setting Up PostgreSQL Locally

### Installing & Running Postgres

**macOS (Homebrew):**

```bash
# Install PostgreSQL
brew install postgresql@15

# Start PostgreSQL
brew services start postgresql@15

# Create database
createdb myapp_dev

# Connect to database
psql myapp_dev
```

**Ubuntu/Debian:**

```bash
# Install PostgreSQL
sudo apt update
sudo apt install postgresql postgresql-contrib

# Start PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql

# Create user and database
sudo -u postgres createuser myapp_user -P
sudo -u postgres createdb myapp_dev -O myapp_user

# Connect
psql -U myapp_user -d myapp_dev
```

**Windows:**

1. Download from [postgresql.org](https://www.postgresql.org/download/windows/)
2. Run installer
3. Use pgAdmin or psql from command line

**Connection string:**

```bash
# .env.local
DATABASE_URL="postgresql://myapp_user:password@localhost:5432/myapp_dev"
```

**Verify connection:**

```bash
psql $DATABASE_URL -c "SELECT version();"
```

---

## 5. Setting Up PostgreSQL with Docker

### Containerized Database

Docker provides isolated, reproducible database environments.

```yaml
# docker-compose.yml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    container_name: myapp_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: myapp_password
      POSTGRES_DB: myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./init.sql:/docker-entrypoint-initdb.d/init.sql

  # Optional: pgAdmin for database management
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: myapp_pgadmin
    restart: unless-stopped
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@example.com
      PGADMIN_DEFAULT_PASSWORD: admin
    ports:
      - "5050:80"
    depends_on:
      - postgres

volumes:
  postgres_data:
```

**Start database:**

```bash
# Start containers
docker-compose up -d

# Check status
docker-compose ps

# View logs
docker-compose logs postgres

# Stop containers
docker-compose down

# Remove data (reset database)
docker-compose down -v
```

**Connection string:**

```bash
# .env
DATABASE_URL="postgresql://myapp_user:myapp_password@localhost:5432/myapp_dev"
```

**Access pgAdmin:**

- Open `http://localhost:5050`
- Login: `admin@example.com` / `admin`
- Add server: `postgres` / `myapp_user` / `myapp_password`

---

## 6. Configuring Drizzle ORM

### Type-Safe Database Access

Drizzle provides TypeScript-first ORM with excellent DX.

**Install dependencies:**

```bash
npm install drizzle-orm postgres
npm install -D drizzle-kit
```

**Drizzle configuration:**

```typescript
// drizzle.config.ts
import type { Config } from "drizzle-kit";
import { env } from "$env/dynamic/private";

export default {
  schema: "./src/lib/server/db/schema.ts",
  out: "./drizzle",
  driver: "pg",
  dbCredentials: {
    connectionString: env.DATABASE_URL!,
  },
  verbose: true,
  strict: true,
} satisfies Config;
```

**Database instance:**

```typescript
// src/lib/server/db/index.ts
import { drizzle } from "drizzle-orm/postgres-js";
import postgres from "postgres";
import { DATABASE_URL } from "../env";
import * as schema from "./schema";

// Connection
const client = postgres(DATABASE_URL, {
  max: 10, // Connection pool size
  idle_timeout: 20,
});

// Drizzle instance with schema
export const db = drizzle(client, { schema });

// Close connection (for cleanup in tests)
export const closeDb = () => client.end();
```

---

## 7. Creating & Migrating Schema

### Database Migrations

Define your schema in TypeScript, generate SQL migrations.

**Define schema:**

```typescript
// src/lib/server/db/schema.ts
import {
  pgTable,
  serial,
  varchar,
  text,
  timestamp,
  integer,
  boolean,
} from "drizzle-orm/pg-core";
import { relations } from "drizzle-orm";

// Users table
// Drizzle schema maps to PostgreSQL table definition
export const users = pgTable("users", {
  // serial = auto-incrementing integer, perfect for IDs
  id: serial("id").primaryKey(),
  // varchar with length limit, unique constraint enforced at DB level
  email: varchar("email", { length: 255 }).notNull().unique(),
  name: varchar("name", { length: 255 }).notNull(),
  // text = unlimited length, use for long strings
  // Store hashed password, NEVER plain text!
  passwordHash: text("password_hash").notNull(),
  avatarUrl: text("avatar_url"),
  // defaultNow() = automatic timestamp on insert
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// Workspaces table
export const workspaces = pgTable("workspaces", {
  id: serial("id").primaryKey(),
  name: varchar("name", { length: 255 }).notNull(),
  // slug for clean URLs: "my-workspace" instead of id
  slug: varchar("slug", { length: 255 }).notNull().unique(),
  description: text("description"),
  // Foreign key to users table
  // references() creates relationship, enables joins
  // onDelete: 'cascade' = when user deleted, delete their workspaces
  ownerId: integer("owner_id")
    .notNull()
    .references(() => users.id, { onDelete: "cascade" }),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// Workspace members (many-to-many)
export const workspaceMembers = pgTable("workspace_members", {
  id: serial("id").primaryKey(),
  workspaceId: integer("workspace_id")
    .notNull()
    .references(() => workspaces.id, { onDelete: "cascade" }),
  userId: integer("user_id")
    .notNull()
    .references(() => users.id, { onDelete: "cascade" }),
  role: varchar("role", { length: 50 }).notNull().default("member"), // owner, admin, member
  joinedAt: timestamp("joined_at").defaultNow().notNull(),
});

// Pages table
export const pages = pgTable("pages", {
  id: serial("id").primaryKey(),
  workspaceId: integer("workspace_id")
    .notNull()
    .references(() => workspaces.id, { onDelete: "cascade" }),
  title: varchar("title", { length: 255 }).notNull(),
  content: text("content"),
  authorId: integer("author_id")
    .notNull()
    .references(() => users.id),
  isPublished: boolean("is_published").default(false).notNull(),
  createdAt: timestamp("created_at").defaultNow().notNull(),
  updatedAt: timestamp("updated_at").defaultNow().notNull(),
});

// Relations (for queries)
export const usersRelations = relations(users, ({ many }) => ({
  ownedWorkspaces: many(workspaces),
  memberships: many(workspaceMembers),
  pages: many(pages),
}));

export const workspacesRelations = relations(workspaces, ({ one, many }) => ({
  owner: one(users, {
    fields: [workspaces.ownerId],
    references: [users.id],
  }),
  members: many(workspaceMembers),
  pages: many(pages),
}));

export const workspaceMembersRelations = relations(
  workspaceMembers,
  ({ one }) => ({
    workspace: one(workspaces, {
      fields: [workspaceMembers.workspaceId],
      references: [workspaces.id],
    }),
    user: one(users, {
      fields: [workspaceMembers.userId],
      references: [users.id],
    }),
  })
);

export const pagesRelations = relations(pages, ({ one }) => ({
  workspace: one(workspaces, {
    fields: [pages.workspaceId],
    references: [workspaces.id],
  }),
  author: one(users, {
    fields: [pages.authorId],
    references: [users.id],
  }),
}));

// TypeScript types (inferred from schema)
export type User = typeof users.$inferSelect;
export type NewUser = typeof users.$inferInsert;
export type Workspace = typeof workspaces.$inferSelect;
export type NewWorkspace = typeof workspaces.$inferInsert;
export type Page = typeof pages.$inferSelect;
export type NewPage = typeof pages.$inferInsert;
```

**Generate migration:**

```bash
# Generate migration from schema
npm run db:generate

# Or add to package.json:
# "db:generate": "drizzle-kit generate:pg"
```

**Run migration:**

```typescript
// scripts/migrate.ts
import { drizzle } from "drizzle-orm/postgres-js";
import { migrate } from "drizzle-orm/postgres-js/migrator";
import postgres from "postgres";
import { DATABASE_URL } from "../src/lib/server/env";

const runMigrations = async () => {
  const connection = postgres(DATABASE_URL, { max: 1 });
  const db = drizzle(connection);

  console.log("Running migrations...");
  await migrate(db, { migrationsFolder: "./drizzle" });
  console.log("Migrations complete!");

  await connection.end();
};

runMigrations().catch(console.error);
```

```json
// package.json
{
  "scripts": {
    "db:generate": "drizzle-kit generate:pg",
    "db:migrate": "tsx scripts/migrate.ts",
    "db:studio": "drizzle-kit studio",
    "db:push": "drizzle-kit push:pg"
  }
}
```

**Run migrations:**

```bash
npm run db:migrate
```

---

## 8. Seeding the Database

### Adding Test Data

Seed realistic test data for development.

```typescript
// scripts/seed.ts
import { db } from "../src/lib/server/db";
import {
  users,
  workspaces,
  workspaceMembers,
  pages,
} from "../src/lib/server/db/schema";
import { hash } from "@node-rs/argon2";

async function seed() {
  console.log("🌱 Seeding database...");

  // Clear existing data
  await db.delete(pages);
  await db.delete(workspaceMembers);
  await db.delete(workspaces);
  await db.delete(users);

  // Create users
  const [alice, bob, charlie] = await db
    .insert(users)
    .values([
      {
        email: "alice@example.com",
        name: "Alice Johnson",
        passwordHash: await hash("password123"),
        avatarUrl: "https://i.pravatar.cc/150?img=1",
      },
      {
        email: "bob@example.com",
        name: "Bob Smith",
        passwordHash: await hash("password123"),
        avatarUrl: "https://i.pravatar.cc/150?img=2",
      },
      {
        email: "charlie@example.com",
        name: "Charlie Brown",
        passwordHash: await hash("password123"),
        avatarUrl: "https://i.pravatar.cc/150?img=3",
      },
    ])
    .returning();

  console.log("✅ Created users:", alice.name, bob.name, charlie.name);

  // Create workspaces
  const [techWorkspace, designWorkspace] = await db
    .insert(workspaces)
    .values([
      {
        name: "Tech Team",
        slug: "tech-team",
        description: "Engineering and development workspace",
        ownerId: alice.id,
      },
      {
        name: "Design Studio",
        slug: "design-studio",
        description: "Creative design projects",
        ownerId: bob.id,
      },
    ])
    .returning();

  console.log(
    "✅ Created workspaces:",
    techWorkspace.name,
    designWorkspace.name
  );

  // Add workspace members
  await db.insert(workspaceMembers).values([
    { workspaceId: techWorkspace.id, userId: alice.id, role: "owner" },
    { workspaceId: techWorkspace.id, userId: bob.id, role: "admin" },
    { workspaceId: techWorkspace.id, userId: charlie.id, role: "member" },
    { workspaceId: designWorkspace.id, userId: bob.id, role: "owner" },
    { workspaceId: designWorkspace.id, userId: alice.id, role: "member" },
  ]);

  console.log("✅ Added workspace members");

  // Create pages
  await db.insert(pages).values([
    {
      workspaceId: techWorkspace.id,
      title: "API Documentation",
      content: "# API Docs\n\nComplete API documentation for our services.",
      authorId: alice.id,
      isPublished: true,
    },
    {
      workspaceId: techWorkspace.id,
      title: "Database Schema",
      content: "# Database Schema\n\nOverview of our database structure.",
      authorId: bob.id,
      isPublished: true,
    },
    {
      workspaceId: techWorkspace.id,
      title: "Sprint Planning",
      content: "# Sprint 23\n\nGoals and tasks for this sprint.",
      authorId: charlie.id,
      isPublished: false,
    },
    {
      workspaceId: designWorkspace.id,
      title: "Brand Guidelines",
      content: "# Brand Guidelines\n\nColors, fonts, and design principles.",
      authorId: bob.id,
      isPublished: true,
    },
    {
      workspaceId: designWorkspace.id,
      title: "UI Components",
      content: "# Component Library\n\nReusable UI components.",
      authorId: alice.id,
      isPublished: true,
    },
  ]);

  console.log("✅ Created pages");
  console.log("\n🎉 Seeding complete!");
  console.log("\nTest credentials:");
  console.log("  alice@example.com / password123");
  console.log("  bob@example.com / password123");
  console.log("  charlie@example.com / password123");
}

seed()
  .catch((error) => {
    console.error("❌ Seeding failed:", error);
    process.exit(1);
  })
  .finally(() => process.exit(0));
```

```json
// package.json
{
  "scripts": {
    "db:seed": "tsx scripts/seed.ts"
  }
}
```

**Run seeding:**

```bash
npm run db:seed
```

---

## 9. Complete Example: Workspace Management System

### Full Database Setup & Configuration

Let's build a complete workspace management system with proper database setup!

**Features:**

- ✅ Local and Docker PostgreSQL setup
- ✅ Type-safe schema with Drizzle
- ✅ Database migrations
- ✅ Realistic seed data
- ✅ Environment variable management
- ✅ Server-only modules
- ✅ Complete workspace/pages CRUD

### 📁 Project Structure

```
myapp/
├── drizzle/                       # Generated migrations
│   └── 0000_initial.sql
├── scripts/
│   ├── migrate.ts                 # Run migrations
│   └── seed.ts                    # Seed data
├── src/
│   ├── lib/
│   │   └── server/
│   │       ├── env.ts             # Environment variables
│   │       └── db/
│   │           ├── index.ts       # DB instance
│   │           ├── schema.ts      # Drizzle schema
│   │           └── queries.ts     # Reusable queries
│   └── routes/
│       └── api/
│           └── health/+server.ts  # Health check
├── .env.example
├── .env                           # Local (gitignored)
├── docker-compose.yml             # Docker setup
├── drizzle.config.ts              # Drizzle config
└── package.json
```

### Environment Setup

```bash
# .env.example
DATABASE_URL="postgresql://myapp_user:myapp_password@localhost:5432/myapp_dev"
PUBLIC_APP_NAME="Workspace Manager"
```

```bash
# .env (create from .env.example)
DATABASE_URL="postgresql://myapp_user:myapp_password@localhost:5432/myapp_dev"
PUBLIC_APP_NAME="Workspace Manager"
```

### Docker Compose

```yaml
# docker-compose.yml
version: "3.8"

services:
  postgres:
    image: postgres:15-alpine
    container_name: workspace_postgres
    restart: unless-stopped
    environment:
      POSTGRES_USER: myapp_user
      POSTGRES_PASSWORD: myapp_password
      POSTGRES_DB: myapp_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U myapp_user"]
      interval: 10s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

### Database Queries

```typescript
// src/lib/server/db/queries.ts
import { db } from "./index";
import { users, workspaces, workspaceMembers, pages } from "./schema";
import { eq, and, desc } from "drizzle-orm";

export const userQueries = {
  findByEmail: async (email: string) => {
    const [user] = await db
      .select()
      .from(users)
      .where(eq(users.email, email))
      .limit(1);
    return user;
  },

  findById: async (id: number) => {
    const [user] = await db
      .select()
      .from(users)
      .where(eq(users.id, id))
      .limit(1);
    return user;
  },

  create: async (data: {
    email: string;
    name: string;
    passwordHash: string;
  }) => {
    const [user] = await db.insert(users).values(data).returning();
    return user;
  },
};

export const workspaceQueries = {
  findBySlug: async (slug: string) => {
    const [workspace] = await db
      .select()
      .from(workspaces)
      .where(eq(workspaces.slug, slug))
      .limit(1);
    return workspace;
  },

  findUserWorkspaces: async (userId: number) => {
    return db
      .select({
        workspace: workspaces,
        role: workspaceMembers.role,
      })
      .from(workspaceMembers)
      .innerJoin(workspaces, eq(workspaceMembers.workspaceId, workspaces.id))
      .where(eq(workspaceMembers.userId, userId))
      .orderBy(desc(workspaceMembers.joinedAt));
  },

  create: async (data: {
    name: string;
    slug: string;
    ownerId: number;
    description?: string;
  }) => {
    const [workspace] = await db.insert(workspaces).values(data).returning();

    // Add owner as member
    await db.insert(workspaceMembers).values({
      workspaceId: workspace.id,
      userId: data.ownerId,
      role: "owner",
    });

    return workspace;
  },
};

export const pageQueries = {
  findByWorkspace: async (workspaceId: number, publishedOnly = false) => {
    let query = db
      .select({
        page: pages,
        author: {
          id: users.id,
          name: users.name,
          avatarUrl: users.avatarUrl,
        },
      })
      .from(pages)
      .innerJoin(users, eq(pages.authorId, users.id))
      .where(eq(pages.workspaceId, workspaceId));

    if (publishedOnly) {
      query = query.where(
        and(eq(pages.workspaceId, workspaceId), eq(pages.isPublished, true))
      );
    }

    return query.orderBy(desc(pages.updatedAt));
  },

  findById: async (id: number) => {
    const [page] = await db
      .select({
        page: pages,
        author: {
          id: users.id,
          name: users.name,
          avatarUrl: users.avatarUrl,
        },
        workspace: {
          id: workspaces.id,
          name: workspaces.name,
          slug: workspaces.slug,
        },
      })
      .from(pages)
      .innerJoin(users, eq(pages.authorId, users.id))
      .innerJoin(workspaces, eq(pages.workspaceId, workspaces.id))
      .where(eq(pages.id, id))
      .limit(1);

    return page;
  },

  create: async (data: {
    workspaceId: number;
    title: string;
    content: string;
    authorId: number;
  }) => {
    const [page] = await db.insert(pages).values(data).returning();
    return page;
  },

  update: async (
    id: number,
    data: Partial<{ title: string; content: string; isPublished: boolean }>
  ) => {
    const [page] = await db
      .update(pages)
      .set({ ...data, updatedAt: new Date() })
      .where(eq(pages.id, id))
      .returning();
    return page;
  },
};
```

### Health Check API

```typescript
// src/routes/api/health/+server.ts
import { json, error } from "@sveltejs/kit";
import { db } from "$lib/server/db";
import type { RequestHandler } from "./$types";

export const GET: RequestHandler = async () => {
  try {
    // Test database connection
    await db.execute("SELECT 1");

    return json({
      status: "ok",
      timestamp: new Date().toISOString(),
      database: "connected",
    });
  } catch (err) {
    throw error(503, "Database connection failed");
  }
};
```

### Package Scripts

```json
// package.json
{
  "name": "workspace-manager",
  "scripts": {
    "dev": "vite dev",
    "build": "vite build",
    "preview": "vite preview",
    "db:generate": "drizzle-kit generate:pg",
    "db:migrate": "tsx scripts/migrate.ts",
    "db:seed": "tsx scripts/seed.ts",
    "db:studio": "drizzle-kit studio",
    "db:push": "drizzle-kit push:pg",
    "db:setup": "npm run db:migrate && npm run db:seed",
    "docker:up": "docker-compose up -d",
    "docker:down": "docker-compose down",
    "docker:reset": "docker-compose down -v && docker-compose up -d"
  },
  "dependencies": {
    "@sveltejs/kit": "^2.0.0",
    "drizzle-orm": "^0.29.0",
    "postgres": "^3.4.0"
  },
  "devDependencies": {
    "@node-rs/argon2": "^1.8.0",
    "drizzle-kit": "^0.20.0",
    "tsx": "^4.7.0",
    "vite": "^5.0.0"
  }
}
```

### Setup Instructions

**Option 1: Local PostgreSQL**

```bash
# 1. Install PostgreSQL (if not already)
brew install postgresql@15  # macOS
# or
sudo apt install postgresql  # Ubuntu

# 2. Create database
createdb myapp_dev

# 3. Set environment variable
echo 'DATABASE_URL="postgresql://localhost:5432/myapp_dev"' > .env

# 4. Install dependencies
npm install

# 5. Run migrations and seed
npm run db:setup

# 6. Start dev server
npm run dev
```

**Option 2: Docker**

```bash
# 1. Start PostgreSQL in Docker
npm run docker:up

# 2. Wait for database to be ready (check health)
docker-compose ps

# 3. Set environment variable
echo 'DATABASE_URL="postgresql://myapp_user:myapp_password@localhost:5432/myapp_dev"' > .env

# 4. Install dependencies
npm install

# 5. Run migrations and seed
npm run db:setup

# 6. Start dev server
npm run dev
```

### Verify Setup

```bash
# Check database connection
curl http://localhost:5173/api/health

# Open Drizzle Studio
npm run db:studio

# View in browser
open http://localhost:4983
```

**Key Features Demonstrated:**

- ✅ **Environment Variables**: Secure configuration management
- ✅ **Server-Only Modules**: Protected database code
- ✅ **Local Setup**: Native PostgreSQL installation
- ✅ **Docker Setup**: Containerized database
- ✅ **Drizzle Schema**: Type-safe database schema
- ✅ **Migrations**: Version-controlled schema changes
- ✅ **Seeding**: Realistic test data
- ✅ **Reusable Queries**: Clean query abstraction
- ✅ **Health Checks**: Database connection monitoring

---

## 📝 Key Takeaways

✅ Environment variables keep secrets secure  
✅ Use `$env/dynamic/private` for runtime secrets  
✅ Never put secrets in `PUBLIC_*` variables  
✅ `$lib/server/` code never goes to browser  
✅ Plan database schema before coding  
✅ Local PostgreSQL: lightweight, native  
✅ Docker: isolated, reproducible environments  
✅ Drizzle provides type-safe ORM with excellent DX  
✅ Generate migrations from TypeScript schema  
✅ Seed databases with realistic test data  
✅ Use connection pooling for performance  
✅ Always validate database connections in health checks

---

## 🚀 Next Steps

You've mastered database setup with Drizzle! Next up:

- **Section 15**: Database Queries & Data Display - Actually using the database in your app
- Building complex queries with joins and filters
- Displaying data with proper loading states and error handling
