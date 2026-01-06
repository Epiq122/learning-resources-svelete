# Final Project: Mystic Readings - Tarot Card SaaS

> **Note:** This is a comprehensive capstone project that integrates all concepts from Sections 1-19. It's designed as a real-world freelance project worth $5,000-$15,000 that demonstrates your mastery of Svelte 5 and SvelteKit.

## 📚 Learning Objectives

By the end of this project, you will have built a complete production-ready SaaS application that combines everything you've learned:

- Full-stack authentication with Better Auth
- Subscription payments with Stripe
- Role-based authorization (free vs paid tiers)
- Database design with Drizzle ORM
- Time-based features (reading expiration)
- Form handling with SuperForms
- Complex business logic
- Admin dashboard
- Real monetization strategy

---

## Table of Contents

- [Final Project: Mystic Readings - Tarot Card SaaS](#final-project-mystic-readings---tarot-card-saas)
  - [📚 Learning Objectives](#-learning-objectives)
  - [Table of Contents](#table-of-contents)
  - [1. Project Overview \& Features](#1-project-overview--features)
    - [What We're Building](#what-were-building)
    - [Free Account Features](#free-account-features)
    - [Paid Account Features ($9.99/month)](#paid-account-features-999month)
    - [Reading Expiration System](#reading-expiration-system)
  - [2. Database Schema \& Setup](#2-database-schema--setup)
    - [Complete Database Design](#complete-database-design)
    - [Database Schema](#database-schema)
    - [Drizzle Configuration](#drizzle-configuration)
    - [Run Migrations](#run-migrations)
  - [3. Authentication Setup](#3-authentication-setup)
    - [Better Auth with Subscription Tiers](#better-auth-with-subscription-tiers)
    - [Hooks for Authentication](#hooks-for-authentication)
    - [Type Definitions](#type-definitions)
    - [Auth Helper Functions](#auth-helper-functions)
  - [4. Card Data \& Deck Management](#4-card-data--deck-management)
    - [Seeding Tarot Cards](#seeding-tarot-cards)
    - [Run Seed Script](#run-seed-script)
    - [Card Helper Functions](#card-helper-functions)
  - [5. Stripe Subscription Integration](#5-stripe-subscription-integration)
    - [Payment Setup with Stripe](#payment-setup-with-stripe)
    - [Create Checkout Session](#create-checkout-session)
    - [Stripe Webhook Handler](#stripe-webhook-handler)
    - [Subscribe Page UI](#subscribe-page-ui)
    - [Success \& Cancel Pages](#success--cancel-pages)
  - [6. Reading Generation Logic](#6-reading-generation-logic)
    - [Card Shuffling \& Selection](#card-shuffling--selection)
    - [Reading API Endpoint](#reading-api-endpoint)
  - [7. User Interface - Reading Experience](#7-user-interface---reading-experience)
    - [New Reading Page](#new-reading-page)
    - [Page Server Load](#page-server-load)
  - [8. Reading History \& Background Jobs](#8-reading-history--background-jobs)
    - [Reading History Page](#reading-history-page)
    - [Background Job: Expire Old Readings](#background-job-expire-old-readings)
    - [Configure Cron Job (Vercel)](#configure-cron-job-vercel)
  - [📝 Final Summary](#-final-summary)
    - [✅ What You've Accomplished](#-what-youve-accomplished)
    - [💰 Real-World Value](#-real-world-value)
    - [🚀 Next Steps](#-next-steps)
  - [🏆 Congratulations!](#-congratulations)

---

## 1. Project Overview & Features

### What We're Building

**Mystic Readings** is a tarot card reading SaaS application with a freemium model.

**Tech Stack:**

- SvelteKit + TypeScript
- PostgreSQL + Drizzle ORM
- Better Auth (authentication)
- Stripe (subscriptions)
- SuperForms + Zod (forms)
- DaisyUI (UI components)

**Business Model:**

- Free tier: Limited cards and readings
- Paid tier ($9.99/month): Full access + saved readings

---

### Free Account Features

✅ **Limited Card Access**

- Access to 22 Major Arcana cards only
- Cannot see Minor Arcana (56 cards)

✅ **Partial Readings**

- 3-card spread only (Past, Present, Future)
- Truncated interpretations (first 100 characters)
- See paywall for full reading

✅ **Basic Experience**

- One deck option (Rider-Waite)
- No reading history
- Cannot save readings

---

### Paid Account Features ($9.99/month)

✅ **Full Card Access**

- All 78 cards (Major + Minor Arcana)
- Multiple decks (Rider-Waite, Modern, Mystical)

✅ **Complete Readings**

- Multiple spread types:
  - 3-Card Spread (Past, Present, Future)
  - Celtic Cross (10 cards)
  - Relationship Spread (7 cards)
  - Career Path (5 cards)
- Full interpretations with guidance
- Reversed card meanings

✅ **Advanced Features**

- Save readings to account
- Reading history (last 30 days)
- Export readings as PDF
- Unlimited daily readings
- No ads

---

### Reading Expiration System

**For paid users:**

- Readings saved to database
- Automatically deleted after 30 days
- Background job runs daily
- Email reminder 3 days before expiration

**For free users:**

- Readings not saved
- Session-only (lost on refresh)
- Must upgrade to save

---

## 2. Database Schema & Setup

### Complete Database Design

Let's design the entire database schema for our tarot SaaS.

**Schema Relationships:**

```
users (Better Auth)
  ↓ 1:many
readings ← user_id
  ↓ 1:many
reading_cards ← reading_id
  ↓ many:1
cards → card_id

cards
  ↓ many:1
decks → deck_id
```

---

### Database Schema

```typescript
// src/lib/server/db/schema.ts
import {
	pgTable,
	text,
	timestamp,
	integer,
	boolean,
	jsonb,
	uuid,
	varchar,
	decimal
} from 'drizzle-orm/pg-core';
import { relations } from 'drizzle-orm';

// ============================================
// USERS (Better Auth integration)
// ============================================

export const users = pgTable('users', {
	id: uuid('id').primaryKey().defaultRandom(),
	email: varchar('email', { length: 255 }).notNull().unique(),
	emailVerified: boolean('email_verified').notNull().default(false),
	name: varchar('name', { length: 255 }),
	image: text('image'),
	createdAt: timestamp('created_at').notNull().defaultNow(),
	updatedAt: timestamp('updated_at').notNull().defaultNow(),

	// Subscription fields
	accountTier: varchar('account_tier', { length: 20 }).notNull().default('free'), // 'free' | 'paid'
	stripeCustomerId: varchar('stripe_customer_id', { length: 255 }),
	stripeSubscriptionId: varchar('stripe_subscription_id', { length: 255 }),
	subscriptionStatus: varchar('subscription_status', { length: 50 }), // 'active' | 'canceled' | 'past_due'
	subscriptionEndsAt: timestamp('subscription_ends_at'),
	trialEndsAt: timestamp('trial_ends_at')
});

export const sessions = pgTable('sessions', {
	id: uuid('id').primaryKey().defaultRandom(),
	userId: uuid('user_id')
		.notNull()
		.references(() => users.id, { onDelete: 'cascade' }),
	expiresAt: timestamp('expires_at').notNull(),
	token: text('token').notNull().unique(),
	ipAddress: varchar('ip_address', { length: 45 }),
	userAgent: text('user_agent'),
	createdAt: timestamp('created_at').notNull().defaultNow()
});

// ============================================
// DECKS
// ============================================

export const decks = pgTable('decks', {
	id: uuid('id').primaryKey().defaultRandom(),
	name: varchar('name', { length: 100 }).notNull(),
	slug: varchar('slug', { length: 100 }).notNull().unique(),
	description: text('description'),
	imageUrl: text('image_url'),
	tierRequired: varchar('tier_required', { length: 20 }).notNull().default('free'), // 'free' | 'paid'
	isActive: boolean('is_active').notNull().default(true),
	createdAt: timestamp('created_at').notNull().defaultNow(),
	updatedAt: timestamp('updated_at').notNull().defaultNow()
});

// ============================================
// CARDS
// ============================================

export const cards = pgTable('cards', {
	id: uuid('id').primaryKey().defaultRandom(),
	deckId: uuid('deck_id')
		.notNull()
		.references(() => decks.id, { onDelete: 'cascade' }),

	name: varchar('name', { length: 100 }).notNull(),
	number: integer('number'), // 0-21 for Major, 1-14 for Minor
	arcana: varchar('arcana', { length: 20 }).notNull(), // 'major' | 'minor'
	suit: varchar('suit', { length: 20 }), // 'wands' | 'cups' | 'swords' | 'pentacles' | null

	// Interpretations
	uprightKeywords: jsonb('upright_keywords').$type<string[]>(),
	reversedKeywords: jsonb('reversed_keywords').$type<string[]>(),
	uprightMeaning: text('upright_meaning').notNull(),
	reversedMeaning: text('reversed_meaning').notNull(),

	// Access control
	tierRequired: varchar('tier_required', { length: 20 }).notNull().default('free'), // 'free' | 'paid'

	// Media
	imageUrl: text('image_url').notNull(),

	createdAt: timestamp('created_at').notNull().defaultNow(),
	updatedAt: timestamp('updated_at').notNull().defaultNow()
});

// ============================================
// SPREADS (Reading Types)
// ============================================

export const spreads = pgTable('spreads', {
	id: uuid('id').primaryKey().defaultRandom(),
	name: varchar('name', { length: 100 }).notNull(),
	slug: varchar('slug', { length: 100 }).notNull().unique(),
	description: text('description'),
	cardCount: integer('card_count').notNull(),
	positions: jsonb('positions').$type<
		{
			position: number;
			name: string;
			meaning: string;
		}[]
	>(),
	tierRequired: varchar('tier_required', { length: 20 }).notNull().default('free'),
	imageUrl: text('image_url'),
	isActive: boolean('is_active').notNull().default(true),
	createdAt: timestamp('created_at').notNull().defaultNow()
});

// ============================================
// READINGS
// ============================================

export const readings = pgTable('readings', {
	id: uuid('id').primaryKey().defaultRandom(),
	userId: uuid('user_id')
		.notNull()
		.references(() => users.id, { onDelete: 'cascade' }),
	deckId: uuid('deck_id')
		.notNull()
		.references(() => decks.id),
	spreadId: uuid('spread_id')
		.notNull()
		.references(() => spreads.id),

	title: varchar('title', { length: 255 }),
	question: text('question'),

	// Reading data
	cards: jsonb('cards').$type<
		{
			cardId: string;
			position: number;
			isReversed: boolean;
		}[]
	>(),

	interpretation: text('interpretation'),

	// Metadata
	createdAt: timestamp('created_at').notNull().defaultNow(),
	expiresAt: timestamp('expires_at'), // 30 days from creation for paid users
	deletedAt: timestamp('deleted_at') // Soft delete
});

// ============================================
// RELATIONS
// ============================================

export const usersRelations = relations(users, ({ many }) => ({
	readings: many(readings),
	sessions: many(sessions)
}));

export const decksRelations = relations(decks, ({ many }) => ({
	cards: many(cards),
	readings: many(readings)
}));

export const cardsRelations = relations(cards, ({ one }) => ({
	deck: one(decks, {
		fields: [cards.deckId],
		references: [decks.id]
	})
}));

export const spreadsRelations = relations(spreads, ({ many }) => ({
	readings: many(readings)
}));

export const readingsRelations = relations(readings, ({ one }) => ({
	user: one(users, {
		fields: [readings.userId],
		references: [users.id]
	}),
	deck: one(decks, {
		fields: [readings.deckId],
		references: [decks.id]
	}),
	spread: one(spreads, {
		fields: [readings.spreadId],
		references: [spreads.id]
	})
}));
```

---

### Drizzle Configuration

```typescript
// drizzle.config.ts
import type { Config } from 'drizzle-kit';
import { config } from 'dotenv';

config();

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

---

### Run Migrations

```bash
# Generate migration
npm run db:generate

# Push to database
npm run db:push

# Or run migration
npm run db:migrate
```

---

## 3. Authentication Setup

### Better Auth with Subscription Tiers

We'll extend Better Auth to include subscription information.

```typescript
// src/lib/server/auth.ts
import { betterAuth } from 'better-auth';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { db } from './db';
import * as schema from './db/schema';

export const auth = betterAuth({
	database: drizzleAdapter(db, {
		provider: 'pg',
		schema
	}),
	emailAndPassword: {
		enabled: true,
		requireEmailVerification: true
	},
	socialProviders: {
		github: {
			clientId: process.env.GITHUB_CLIENT_ID!,
			clientSecret: process.env.GITHUB_CLIENT_SECRET!
		},
		google: {
			clientId: process.env.GOOGLE_CLIENT_ID!,
			clientSecret: process.env.GOOGLE_CLIENT_SECRET!
		}
	},
	session: {
		expiresIn: 60 * 60 * 24 * 7, // 7 days
		updateAge: 60 * 60 * 24 // Update session every day
	}
});
```

---

### Hooks for Authentication

```typescript
// src/hooks.server.ts
import { sequence } from '@sveltejs/kit/hooks';
import { auth } from '$lib/server/auth';
import type { Handle } from '@sveltejs/kit';

const handleAuth: Handle = async ({ event, resolve }) => {
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

---

### Type Definitions

```typescript
// src/app.d.ts
import type { User, Session } from 'better-auth/types';

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

---

### Auth Helper Functions

```typescript
// src/lib/server/auth-helpers.ts
import { redirect } from '@sveltejs/kit';
import type { RequestEvent } from '@sveltejs/kit';

export function requireAuth(event: RequestEvent) {
	if (!event.locals.user) {
		throw redirect(303, '/login');
	}
	return event.locals.user;
}

export function requirePaidAccount(event: RequestEvent) {
	const user = requireAuth(event);

	if (user.accountTier !== 'paid') {
		throw redirect(303, '/upgrade');
	}

	return user;
}

export function isPaidUser(user: { accountTier: string } | null): boolean {
	return user?.accountTier === 'paid';
}

export function hasActiveSubscription(
	user: {
		accountTier: string;
		subscriptionStatus?: string | null;
		subscriptionEndsAt?: Date | null;
	} | null
): boolean {
	if (!user || user.accountTier !== 'paid') return false;

	if (user.subscriptionStatus === 'canceled' && user.subscriptionEndsAt) {
		return new Date() < new Date(user.subscriptionEndsAt);
	}

	return user.subscriptionStatus === 'active';
}
```

---

## 4. Card Data & Deck Management

### Seeding Tarot Cards

Let's create a seed script to populate our database with tarot cards.

```typescript
// src/lib/server/db/seed-tarot.ts
import { db } from './index';
import { decks, cards, spreads } from './schema';

// Major Arcana Data
const majorArcana = [
	{
		number: 0,
		name: 'The Fool',
		uprightKeywords: ['beginnings', 'innocence', 'spontaneity', 'free spirit'],
		reversedKeywords: ['recklessness', 'risk-taking', 'naivety'],
		uprightMeaning:
			"The Fool represents new beginnings, having faith in the future, being inexperienced, not knowing what to expect, having beginner's luck, improvisation and believing in the universe.",
		reversedMeaning:
			'When reversed, The Fool can indicate recklessness, carelessness, and taking unnecessary risks without thinking about the consequences.',
		imageUrl: '/cards/rider-waite/major/00-fool.jpg',
		tierRequired: 'free'
	},
	{
		number: 1,
		name: 'The Magician',
		uprightKeywords: ['manifestation', 'resourcefulness', 'power', 'inspired action'],
		reversedKeywords: ['manipulation', 'poor planning', 'untapped talents'],
		uprightMeaning:
			'The Magician represents manifestation, resourcefulness, inspired action, and having the power to create your desired reality through focus and determination.',
		reversedMeaning:
			'Reversed, The Magician suggests manipulation, poor planning, or untapped potential. You may be using your skills for selfish or negative purposes.',
		imageUrl: '/cards/rider-waite/major/01-magician.jpg',
		tierRequired: 'free'
	},
	{
		number: 2,
		name: 'The High Priestess',
		uprightKeywords: ['intuition', 'sacred knowledge', 'divine feminine', 'subconscious mind'],
		reversedKeywords: ['secrets', 'disconnected from intuition', 'withdrawal'],
		uprightMeaning:
			'The High Priestess signifies spiritual enlightenment, inner illumination, divine knowledge and wisdom. She represents intuition and the subconscious mind.',
		reversedMeaning:
			'When reversed, she indicates secrets, disconnection from intuition, and withdrawal from the world. Hidden agendas may be at play.',
		imageUrl: '/cards/rider-waite/major/02-high-priestess.jpg',
		tierRequired: 'free'
	}
	// ... Add all 22 Major Arcana cards
];

// Minor Arcana - Wands (example)
const wandsCards = [
	{
		number: 1,
		name: 'Ace of Wands',
		suit: 'wands',
		uprightKeywords: ['inspiration', 'new opportunities', 'growth', 'potential'],
		reversedKeywords: ['emerging ideas', 'lack of direction', 'delays'],
		uprightMeaning:
			'The Ace of Wands represents inspiration, new opportunities, and creative potential. A spark of passion and enthusiasm for a new project or idea.',
		reversedMeaning:
			"Reversed, it suggests delays, lack of direction, or emerging ideas that aren't quite ready. You may be experiencing creative blocks.",
		imageUrl: '/cards/rider-waite/wands/01-ace.jpg',
		tierRequired: 'paid'
	}
	// ... Add all 14 Wands cards (Ace through King)
];

// Spreads
const spreadTypes = [
	{
		name: '3-Card Spread',
		slug: '3-card',
		description: 'Past, Present, Future - A simple yet powerful reading for guidance',
		cardCount: 3,
		positions: [
			{ position: 1, name: 'Past', meaning: 'Past influences affecting the current situation' },
			{ position: 2, name: 'Present', meaning: 'The current situation or challenge' },
			{ position: 3, name: 'Future', meaning: 'The likely outcome or future path' }
		],
		tierRequired: 'free',
		imageUrl: '/spreads/3-card.jpg'
	},
	{
		name: 'Celtic Cross',
		slug: 'celtic-cross',
		description: 'The most detailed and comprehensive spread',
		cardCount: 10,
		positions: [
			{ position: 1, name: 'Present', meaning: 'The current situation' },
			{ position: 2, name: 'Challenge', meaning: 'What crosses or challenges you' },
			{ position: 3, name: 'Foundation', meaning: 'The root cause or foundation' },
			{ position: 4, name: 'Past', meaning: 'Recent past influences' },
			{ position: 5, name: 'Crown', meaning: 'What is above or conscious goals' },
			{ position: 6, name: 'Future', meaning: 'Near future influences' },
			{ position: 7, name: 'Self', meaning: 'Your attitude and approach' },
			{ position: 8, name: 'Environment', meaning: 'External influences' },
			{ position: 9, name: 'Hopes/Fears', meaning: 'Your hopes and fears' },
			{ position: 10, name: 'Outcome', meaning: 'The final outcome' }
		],
		tierRequired: 'paid',
		imageUrl: '/spreads/celtic-cross.jpg'
	},
	{
		name: 'Relationship Spread',
		slug: 'relationship',
		description: 'Explore the dynamics and future of a relationship',
		cardCount: 7,
		positions: [
			{ position: 1, name: 'You', meaning: 'Your position in the relationship' },
			{ position: 2, name: 'Them', meaning: 'Their position in the relationship' },
			{ position: 3, name: 'Connection', meaning: 'What connects you' },
			{ position: 4, name: 'Challenge', meaning: 'Current challenges' },
			{ position: 5, name: 'Past', meaning: 'Past influences' },
			{ position: 6, name: 'Future', meaning: 'Potential future' },
			{ position: 7, name: 'Advice', meaning: 'Guidance for the relationship' }
		],
		tierRequired: 'paid',
		imageUrl: '/spreads/relationship.jpg'
	}
];

export async function seedTarotData() {
	console.log('🌱 Seeding tarot data...');

	// 1. Create Decks
	const [riderWaiteDeck] = await db
		.insert(decks)
		.values({
			name: 'Rider-Waite Tarot',
			slug: 'rider-waite',
			description: 'The classic and most widely recognized tarot deck',
			imageUrl: '/decks/rider-waite.jpg',
			tierRequired: 'free',
			isActive: true
		})
		.returning();

	const [modernDeck] = await db
		.insert(decks)
		.values({
			name: 'Modern Mystic Tarot',
			slug: 'modern-mystic',
			description: 'A contemporary interpretation with vibrant artwork',
			imageUrl: '/decks/modern-mystic.jpg',
			tierRequired: 'paid',
			isActive: true
		})
		.returning();

	console.log(`✅ Created ${2} decks`);

	// 2. Create Spreads
	await db.insert(spreads).values(spreadTypes);
	console.log(`✅ Created ${spreadTypes.length} spread types`);

	// 3. Create Cards for Rider-Waite Deck
	const cardsToInsert = [
		// Major Arcana
		...majorArcana.map((card) => ({
			...card,
			deckId: riderWaiteDeck.id,
			arcana: 'major' as const,
			suit: null
		})),
		// Minor Arcana - Wands
		...wandsCards.map((card) => ({
			...card,
			deckId: riderWaiteDeck.id,
			arcana: 'minor' as const
		}))
		// ... Add Cups, Swords, Pentacles
	];

	await db.insert(cards).values(cardsToInsert);
	console.log(`✅ Created ${cardsToInsert.length} cards`);

	console.log('🎉 Tarot data seeding complete!');
}

// Run if called directly
if (import.meta.url === `file://${process.argv[1]}`) {
	seedTarotData()
		.then(() => process.exit(0))
		.catch((error) => {
			console.error('❌ Seeding failed:', error);
			process.exit(1);
		});
}
```

---

### Run Seed Script

```bash
# Add to package.json
{
	"scripts": {
		"db:seed:tarot": "tsx src/lib/server/db/seed-tarot.ts"
	}
}

# Run it
npm run db:seed:tarot
```

---

### Card Helper Functions

```typescript
// src/lib/server/db/queries/cards.ts
import { db } from '../index';
import { cards, decks } from '../schema';
import { eq, and } from 'drizzle-orm';

export async function getCardsForUser(deckId: string, userTier: 'free' | 'paid') {
	const availableCards = await db
		.select()
		.from(cards)
		.where(
			and(
				eq(cards.deckId, deckId),
				// Free users only see free cards (Major Arcana)
				userTier === 'free' ? eq(cards.tierRequired, 'free') : undefined // Paid users see all
			)
		)
		.orderBy(cards.arcana, cards.number);

	return availableCards;
}

export async function getAvailableDecks(userTier: 'free' | 'paid') {
	const availableDecks = await db
		.select()
		.from(decks)
		.where(
			and(
				eq(decks.isActive, true),
				userTier === 'free' ? eq(decks.tierRequired, 'free') : undefined
			)
		);

	return availableDecks;
}

export async function getCardById(cardId: string) {
	const [card] = await db.select().from(cards).where(eq(cards.id, cardId)).limit(1);

	return card;
}
```

---

## 5. Stripe Subscription Integration

### Payment Setup with Stripe

Implement subscription payments for paid tier access.

**Install Stripe:**

```bash
npm install stripe @stripe/stripe-js
```

**Environment variables:**

```env
# .env
STRIPE_SECRET_KEY=sk_test_...
PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_test_...
STRIPE_WEBHOOK_SECRET=whsec_...
STRIPE_PRICE_ID=price_... # Monthly subscription price
```

**Stripe configuration:**

```typescript
// src/lib/server/stripe.ts
import Stripe from 'stripe';
import { STRIPE_SECRET_KEY } from '$env/static/private';

export const stripe = new Stripe(STRIPE_SECRET_KEY, {
	apiVersion: '2024-12-18.acacia'
});

// Product configuration
export const SUBSCRIPTION_PRICE_ID = 'price_1234567890'; // Your Stripe price ID
export const SUBSCRIPTION_AMOUNT = 999; // $9.99 in cents
```

### Create Checkout Session

```typescript
// src/routes/(app)/subscribe/+page.server.ts
import { redirect, fail } from '@sveltejs/kit';
import { stripe, SUBSCRIPTION_PRICE_ID } from '$lib/server/stripe';
import { PUBLIC_APP_URL } from '$env/static/public';
import type { PageServerLoad, Actions } from './$types';

export const load: PageServerLoad = async ({ locals }) => {
	if (!locals.user) {
		throw redirect(303, '/login');
	}

	if (locals.user.accountTier === 'paid') {
		throw redirect(303, '/dashboard');
	}

	return {};
};

export const actions = {
	default: async ({ locals, url }) => {
		if (!locals.user) {
			return fail(401, { error: 'Unauthorized' });
		}

		try {
			// Create Stripe checkout session
			const session = await stripe.checkout.sessions.create({
				customer_email: locals.user.email,
				client_reference_id: locals.user.id,
				mode: 'subscription',
				payment_method_types: ['card'],
				line_items: [
					{
						price: SUBSCRIPTION_PRICE_ID,
						quantity: 1
					}
				],
				success_url: `${PUBLIC_APP_URL}/subscribe/success?session_id={CHECKOUT_SESSION_ID}`,
				cancel_url: `${PUBLIC_APP_URL}/subscribe/cancel`,
				metadata: {
					userId: locals.user.id
				}
			});

			throw redirect(303, session.url!);
		} catch (error) {
			console.error('Stripe checkout error:', error);
			return fail(500, { error: 'Failed to create checkout session' });
		}
	}
} satisfies Actions;
```

### Stripe Webhook Handler

```typescript
// src/routes/api/webhooks/stripe/+server.ts
import { json } from '@sveltejs/kit';
import { stripe } from '$lib/server/stripe';
import { db } from '$lib/server/db';
import { users } from '$lib/server/db/schema';
import { eq } from 'drizzle-orm';
import { STRIPE_WEBHOOK_SECRET } from '$env/static/private';
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ request }) => {
	const body = await request.text();
	const signature = request.headers.get('stripe-signature');

	if (!signature) {
		return json({ error: 'No signature' }, { status: 400 });
	}

	let event;

	try {
		event = stripe.webhooks.constructEvent(body, signature, STRIPE_WEBHOOK_SECRET);
	} catch (err) {
		console.error('Webhook signature verification failed:', err);
		return json({ error: 'Invalid signature' }, { status: 400 });
	}

	// Handle the event
	switch (event.type) {
		case 'checkout.session.completed': {
			const session = event.data.object as Stripe.Checkout.Session;
			const userId = session.metadata?.userId || session.client_reference_id;

			if (userId && session.subscription) {
				// Upgrade user to paid
				await db
					.update(users)
					.set({
						accountTier: 'paid',
						stripeCustomerId: session.customer as string,
						stripeSubscriptionId: session.subscription as string,
						subscriptionStatus: 'active'
					})
					.where(eq(users.id, userId));

				console.log(`User ${userId} upgraded to paid`);
			}
			break;
		}

		case 'customer.subscription.updated': {
			const subscription = event.data.object as Stripe.Subscription;

			await db
				.update(users)
				.set({
					subscriptionStatus: subscription.status
				})
				.where(eq(users.stripeSubscriptionId, subscription.id));

			break;
		}

		case 'customer.subscription.deleted': {
			const subscription = event.data.object as Stripe.Subscription;

			// Downgrade user to free
			await db
				.update(users)
				.set({
					accountTier: 'free',
					subscriptionStatus: 'canceled'
				})
				.where(eq(users.stripeSubscriptionId, subscription.id));

			console.log(`Subscription ${subscription.id} canceled`);
			break;
		}
	}

	return json({ received: true });
};
```

### Subscribe Page UI

```svelte
<!-- src/routes/(app)/subscribe/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';

	let loading = $state(false);
</script>

<div class="min-h-screen bg-gradient-to-br from-purple-900 via-indigo-900 to-black text-white">
	<div class="container mx-auto px-4 py-16">
		<div class="max-w-4xl mx-auto">
			<!-- Header -->
			<div class="text-center mb-12">
				<h1 class="text-5xl font-bold mb-4">Unlock Your Mystical Journey</h1>
				<p class="text-xl text-purple-200">Upgrade to Premium and access the full power of tarot</p>
			</div>

			<!-- Pricing Comparison -->
			<div class="grid md:grid-cols-2 gap-8 mb-12">
				<!-- Free Plan -->
				<div class="card bg-gray-800 border-2 border-gray-700">
					<div class="card-body">
						<h2 class="card-title text-2xl">Free Account</h2>
						<p class="text-3xl font-bold my-4">$0<span class="text-lg font-normal">/month</span></p>

						<div class="space-y-3">
							<div class="flex items-center gap-2">
								<span class="text-green-400">✓</span>
								<span>22 Major Arcana cards</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-green-400">✓</span>
								<span>3-Card spread only</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-green-400">✓</span>
								<span>Partial interpretations</span>
							</div>
							<div class="flex items-center gap-2 opacity-50">
								<span class="text-red-400">✗</span>
								<span>No reading history</span>
							</div>
							<div class="flex items-center gap-2 opacity-50">
								<span class="text-red-400">✗</span>
								<span>Limited deck access</span>
							</div>
						</div>
					</div>
				</div>

				<!-- Paid Plan -->
				<div
					class="card bg-gradient-to-br from-purple-600 to-indigo-600 border-4 border-yellow-400"
				>
					<div class="badge badge-warning absolute top-4 right-4">Most Popular</div>
					<div class="card-body">
						<h2 class="card-title text-2xl">Premium Account</h2>
						<p class="text-3xl font-bold my-4">
							$9.99<span class="text-lg font-normal">/month</span>
						</p>

						<div class="space-y-3">
							<div class="flex items-center gap-2">
								<span class="text-yellow-300">✓</span>
								<span class="font-semibold">All 78 tarot cards</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-yellow-300">✓</span>
								<span class="font-semibold">All spread types</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-yellow-300">✓</span>
								<span class="font-semibold">Full interpretations</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-yellow-300">✓</span>
								<span class="font-semibold">Save reading history (30 days)</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-yellow-300">✓</span>
								<span class="font-semibold">Multiple premium decks</span>
							</div>
							<div class="flex items-center gap-2">
								<span class="text-yellow-300">✓</span>
								<span class="font-semibold">Export readings as PDF</span>
							</div>
						</div>

						<form
							method="POST"
							use:enhance={() => {
								loading = true;
								return async ({ update }) => {
									await update();
									loading = false;
								};
							}}
							class="mt-6"
						>
							<button type="submit" disabled={loading} class="btn btn-warning btn-block btn-lg">
								{loading ? 'Processing...' : 'Upgrade Now'}
							</button>
						</form>

						<p class="text-sm text-center mt-4 opacity-80">
							Cancel anytime. No long-term commitment.
						</p>
					</div>
				</div>
			</div>

			<!-- FAQ -->
			<div class="text-center text-purple-200">
				<p class="mb-2">Questions? <a href="/faq" class="underline">Visit our FAQ</a></p>
				<p>Or <a href="/contact" class="underline">contact support</a></p>
			</div>
		</div>
	</div>
</div>
```

### Success & Cancel Pages

```svelte
<!-- src/routes/(app)/subscribe/success/+page.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import { goto } from '$app/navigation';

	let countdown = $state(5);

	onMount(() => {
		const interval = setInterval(() => {
			countdown--;
			if (countdown === 0) {
				goto('/dashboard');
			}
		}, 1000);

		return () => clearInterval(interval);
	});
</script>

<div class="hero min-h-screen bg-gradient-to-br from-purple-900 to-black">
	<div class="hero-content text-center">
		<div class="max-w-md">
			<div class="text-6xl mb-4">🎉</div>
			<h1 class="text-5xl font-bold text-white mb-4">Welcome to Premium!</h1>
			<p class="text-xl text-purple-200 mb-8">
				Your account has been upgraded. You now have access to all premium features!
			</p>

			<div class="space-y-4">
				<p class="text-purple-300">Redirecting to dashboard in {countdown} seconds...</p>
				<a href="/dashboard" class="btn btn-primary btn-lg"> Go to Dashboard Now </a>
			</div>
		</div>
	</div>
</div>
```

---

## 6. Reading Generation Logic

### Card Shuffling & Selection

```typescript
// src/lib/server/reading-engine.ts
import { db } from './db';
import { cards, spreads, readings } from './db/schema';
import { eq, and, inArray } from 'drizzle-orm';
import type { User } from './db/schema';

export interface SelectedCard {
	id: string;
	name: string;
	arcana: string;
	suit: string | null;
	position: number;
	reversed: boolean;
	uprightMeaning: string;
	reversedMeaning: string;
	imageUrl: string;
}

export interface Reading {
	id: string;
	spreadName: string;
	cards: SelectedCard[];
	interpretation: string;
	createdAt: Date;
}

/**
 * Shuffle an array using Fisher-Yates algorithm
 */
function shuffleArray<T>(array: T[]): T[] {
	const shuffled = [...array];
	for (let i = shuffled.length - 1; i > 0; i--) {
		const j = Math.floor(Math.random() * (i + 1));
		[shuffled[i], shuffled[j]] = [shuffled[j], shuffled[i]];
	}
	return shuffled;
}

/**
 * Get available cards based on user tier and deck
 */
export async function getAvailableCards(deckId: string, userTier: 'free' | 'paid') {
	const availableCards = await db
		.select()
		.from(cards)
		.where(
			and(
				eq(cards.deckId, deckId),
				userTier === 'free' ? eq(cards.tierRequired, 'free') : undefined
			)
		);

	return availableCards;
}

/**
 * Draw cards for a reading
 */
export async function drawCards(
	deckId: string,
	spreadId: string,
	userTier: 'free' | 'paid'
): Promise<SelectedCard[]> {
	// Get available cards
	const availableCards = await getAvailableCards(deckId, userTier);

	if (availableCards.length === 0) {
		throw new Error('No cards available for this deck');
	}

	// Get spread to know how many cards to draw
	const [spread] = await db.select().from(spreads).where(eq(spreads.id, spreadId)).limit(1);

	if (!spread) {
		throw new Error('Spread not found');
	}

	// Shuffle and select cards
	const shuffled = shuffleArray(availableCards);
	const selectedCards = shuffled.slice(0, spread.cardCount);

	// Randomly reverse some cards (30% chance)
	const result: SelectedCard[] = selectedCards.map((card, index) => ({
		id: card.id,
		name: card.name,
		arcana: card.arcana,
		suit: card.suit,
		position: index + 1,
		reversed: Math.random() < 0.3, // 30% chance of reversed
		uprightMeaning: card.uprightMeaning,
		reversedMeaning: card.reversedMeaning,
		imageUrl: card.imageUrl
	}));

	return result;
}

/**
 * Generate interpretation based on cards and spread
 */
export async function generateInterpretation(
	cards: SelectedCard[],
	spreadId: string,
	userTier: 'free' | 'paid'
): Promise<string> {
	const [spread] = await db.select().from(spreads).where(eq(spreads.id, spreadId)).limit(1);

	if (!spread) {
		throw new Error('Spread not found');
	}

	const positions = JSON.parse(spread.positions as string) as string[];

	// Build interpretation
	let interpretation = `**${spread.name} Reading**\n\n`;
	interpretation += `${spread.description}\n\n`;

	cards.forEach((card, index) => {
		const meaning = card.reversed ? card.reversedMeaning : card.uprightMeaning;
		const position = positions[index] || `Position ${index + 1}`;

		interpretation += `**${position}: ${card.name}${card.reversed ? ' (Reversed)' : ''}**\n`;
		interpretation += `${meaning}\n\n`;
	});

	// Add summary
	interpretation += `**Summary**\n`;
	interpretation += `This reading reveals ${cards.length} cards that together tell a story about your current path. `;

	// Tier-specific content
	if (userTier === 'free') {
		interpretation += `\n\n*🔒 Upgrade to Premium for a complete in-depth interpretation with personalized insights and guidance.*`;
	} else {
		// Add deeper interpretation for paid users
		interpretation += `The combination of these cards suggests a journey of transformation and growth. `;
		interpretation += `Pay special attention to ${cards[0].name}, which sets the foundation for this reading. `;

		if (cards.length >= 3) {
			interpretation += `The interplay between ${cards[1].name} and ${cards[2].name} indicates important dynamics at work in your situation.`;
		}
	}

	return interpretation;
}

/**
 * Create and save a reading
 */
export async function createReading(
	userId: string,
	deckId: string,
	spreadId: string,
	userTier: 'free' | 'paid'
) {
	// Draw cards
	const selectedCards = await drawCards(deckId, spreadId, userTier);

	// Generate interpretation
	const interpretation = await generateInterpretation(selectedCards, spreadId, userTier);

	// Calculate expiration (30 days for paid users, null for free)
	const expiresAt = userTier === 'paid' ? new Date(Date.now() + 30 * 24 * 60 * 60 * 1000) : null;

	// Save reading (only for paid users)
	if (userTier === 'paid') {
		const [newReading] = await db
			.insert(readings)
			.values({
				userId,
				deckId,
				spreadId,
				cards: JSON.stringify(selectedCards),
				interpretation,
				expiresAt
			})
			.returning();

		return {
			id: newReading.id,
			cards: selectedCards,
			interpretation,
			createdAt: newReading.createdAt,
			canSave: true
		};
	}

	// Return without saving for free users
	return {
		id: null,
		cards: selectedCards,
		interpretation,
		createdAt: new Date(),
		canSave: false
	};
}
```

### Reading API Endpoint

```typescript
// src/routes/api/readings/+server.ts
import { json, error } from '@sveltejs/kit';
import { createReading } from '$lib/server/reading-engine';
import { requireAuth } from '$lib/server/auth-helpers';
import type { RequestHandler } from './$types';

export const POST: RequestHandler = async ({ request, locals }) => {
	const user = requireAuth(locals);

	const { deckId, spreadId } = await request.json();

	if (!deckId || !spreadId) {
		throw error(400, 'Deck ID and Spread ID are required');
	}

	try {
		const reading = await createReading(user.id, deckId, spreadId, user.accountTier);

		return json(reading);
	} catch (err) {
		console.error('Reading creation error:', err);
		throw error(500, 'Failed to create reading');
	}
};
```

---

## 7. User Interface - Reading Experience

### New Reading Page

```svelte
<!-- src/routes/(app)/reading/new/+page.svelte -->
<script lang="ts">
	import { onMount } from 'svelte';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	let selectedDeck = $state(data.decks[0]?.id || '');
	let selectedSpread = $state(data.spreads[0]?.id || '');
	let loading = $state(false);
	let reading = $state<any>(null);
	let flippedCards = $state<Set<number>>(new Set());

	async function generateReading() {
		loading = true;
		reading = null;
		flippedCards = new Set();

		try {
			const response = await fetch('/api/readings', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({
					deckId: selectedDeck,
					spreadId: selectedSpread
				})
			});

			if (!response.ok) throw new Error('Failed to generate reading');

			reading = await response.json();

			// Auto-flip cards one by one
			reading.cards.forEach((_: any, index: number) => {
				setTimeout(
					() => {
						flippedCards.add(index);
						flippedCards = new Set(flippedCards);
					},
					500 * (index + 1)
				);
			});
		} catch (error) {
			console.error('Error generating reading:', error);
			alert('Failed to generate reading. Please try again.');
		} finally {
			loading = false;
		}
	}

	function toggleCard(index: number) {
		if (flippedCards.has(index)) {
			flippedCards.delete(index);
		} else {
			flippedCards.add(index);
		}
		flippedCards = new Set(flippedCards);
	}
</script>

<div class="min-h-screen bg-gradient-to-br from-purple-900 via-indigo-900 to-black text-white">
	<div class="container mx-auto px-4 py-12">
		{#if !reading}
			<!-- Selection Phase -->
			<div class="max-w-2xl mx-auto">
				<h1 class="text-4xl font-bold text-center mb-8">Start a New Reading</h1>

				<!-- Deck Selection -->
				<div class="card bg-gray-800 mb-6">
					<div class="card-body">
						<h2 class="card-title">Choose Your Deck</h2>
						<div class="grid grid-cols-2 gap-4">
							{#each data.decks as deck}
								<label class="cursor-pointer">
									<input
										type="radio"
										name="deck"
										value={deck.id}
										bind:group={selectedDeck}
										class="radio radio-primary"
									/>
									<div class="ml-3 {selectedDeck === deck.id ? 'text-primary font-bold' : ''}">
										{deck.name}
										{#if deck.tierRequired === 'paid' && data.user.accountTier === 'free'}
											<span class="badge badge-warning badge-sm">Premium</span>
										{/if}
									</div>
									<p class="text-sm text-gray-400 ml-7">{deck.description}</p>
								</label>
							{/each}
						</div>
					</div>
				</div>

				<!-- Spread Selection -->
				<div class="card bg-gray-800 mb-6">
					<div class="card-body">
						<h2 class="card-title">Choose Your Spread</h2>
						<div class="space-y-3">
							{#each data.spreads as spread}
								<label class="cursor-pointer flex items-start">
									<input
										type="radio"
										name="spread"
										value={spread.id}
										bind:group={selectedSpread}
										class="radio radio-primary mt-1"
									/>
									<div class="ml-3 flex-1">
										<div class={selectedSpread === spread.id ? 'text-primary font-bold' : ''}>
											{spread.name}
											{#if spread.tierRequired === 'paid' && data.user.accountTier === 'free'}
												<span class="badge badge-warning badge-sm">Premium</span>
											{/if}
										</div>
										<p class="text-sm text-gray-400">{spread.description}</p>
									</div>
								</label>
							{/each}
						</div>
					</div>
				</div>

				<!-- Generate Button -->
				<button
					onclick={generateReading}
					disabled={loading}
					class="btn btn-primary btn-lg btn-block"
				>
					{loading ? 'Drawing Cards...' : 'Generate Reading'}
				</button>
			</div>
		{:else}
			<!-- Reading Display -->
			<div class="max-w-6xl mx-auto">
				<div class="text-center mb-8">
					<h1 class="text-4xl font-bold mb-4">Your Reading</h1>
					<p class="text-purple-200">Click any card to flip it</p>
				</div>

				<!-- Card Display -->
				<div class="flex flex-wrap justify-center gap-6 mb-12">
					{#each reading.cards as card, index}
						<button
							onclick={() => toggleCard(index)}
							class="card-flip-container {flippedCards.has(index) ? 'flipped' : ''}"
						>
							<div class="card-flip-inner">
								<!-- Card Back -->
								<div class="card-flip-face card-flip-back">
									<div
										class="w-40 h-64 bg-gradient-to-br from-purple-600 to-indigo-600 rounded-lg flex items-center justify-center border-2 border-purple-400"
									>
										<span class="text-6xl">🔮</span>
									</div>
								</div>

								<!-- Card Front -->
								<div class="card-flip-face card-flip-front">
									<div class="w-40 h-64 bg-gray-800 rounded-lg p-4 border-2 border-purple-400">
										<div class="text-center">
											<div class="text-4xl mb-2">
												{card.reversed ? '🔄' : '✨'}
											</div>
											<h3 class="font-bold text-sm mb-2">
												{card.name}
											</h3>
											{#if card.reversed}
												<span class="badge badge-warning badge-sm">Reversed</span>
											{/if}
										</div>
									</div>
								</div>
							</div>
						</button>
					{/each}
				</div>

				<!-- Interpretation -->
				<div class="card bg-gray-800">
					<div class="card-body">
						<h2 class="card-title">Interpretation</h2>
						<div class="prose prose-invert max-w-none">
							{@html reading.interpretation.replace(/\n/g, '<br>')}
						</div>

						{#if !reading.canSave && data.user.accountTier === 'free'}
							<div class="alert alert-warning mt-6">
								<span
									>🔒 Upgrade to Premium to save your readings and get full interpretations!</span
								>
								<a href="/subscribe" class="btn btn-warning btn-sm">Upgrade Now</a>
							</div>
						{/if}

						<div class="card-actions justify-end mt-6">
							<a href="/reading/new" class="btn btn-secondary">New Reading</a>
							{#if reading.canSave}
								<a href="/readings" class="btn btn-primary">View Saved Readings</a>
							{/if}
						</div>
					</div>
				</div>
			</div>
		{/if}
	</div>
</div>

<style>
	.card-flip-container {
		perspective: 1000px;
		cursor: pointer;
	}

	.card-flip-inner {
		position: relative;
		width: 160px;
		height: 256px;
		transition: transform 0.6s;
		transform-style: preserve-3d;
	}

	.card-flip-container.flipped .card-flip-inner {
		transform: rotateY(180deg);
	}

	.card-flip-face {
		position: absolute;
		width: 100%;
		height: 100%;
		backface-visibility: hidden;
	}

	.card-flip-front {
		transform: rotateY(180deg);
	}
</style>
```

### Page Server Load

```typescript
// src/routes/(app)/reading/new/+page.server.ts
import { redirect } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import { decks, spreads } from '$lib/server/db/schema';
import { eq, or } from 'drizzle-orm';
import type { PageServerLoad } from './$types';

export const load: PageServerLoad = async ({ locals }) => {
	if (!locals.user) {
		throw redirect(303, '/login');
	}

	const userTier = locals.user.accountTier;

	// Get available decks
	const availableDecks = await db
		.select()
		.from(decks)
		.where(userTier === 'free' ? eq(decks.tierRequired, 'free') : undefined);

	// Get available spreads
	const availableSpreads = await db
		.select()
		.from(spreads)
		.where(userTier === 'free' ? eq(spreads.tierRequired, 'free') : undefined);

	return {
		user: locals.user,
		decks: availableDecks,
		spreads: availableSpreads
	};
};
```

---

## 8. Reading History & Background Jobs

### Reading History Page

```typescript
// src/routes/(app)/readings/+page.server.ts
import { redirect } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import { readings } from '$lib/server/db/schema';
import { eq, desc, isNull } from 'drizzle-orm';
import type { PageServerLoad, Actions } from './$types';

export const load: PageServerLoad = async ({ locals }) => {
	if (!locals.user) {
		throw redirect(303, '/login');
	}

	if (locals.user.accountTier === 'free') {
		throw redirect(303, '/subscribe');
	}

	// Get user's saved readings
	const savedReadings = await db
		.select()
		.from(readings)
		.where(eq(readings.userId, locals.user.id))
		.orderBy(desc(readings.createdAt));

	return {
		readings: savedReadings
	};
};

export const actions = {
	delete: async ({ request, locals }) => {
		if (!locals.user) {
			throw redirect(303, '/login');
		}

		const formData = await request.formData();
		const readingId = formData.get('readingId') as string;

		// Soft delete
		await db.update(readings).set({ deletedAt: new Date() }).where(eq(readings.id, readingId));

		return { success: true };
	}
} satisfies Actions;
```

```svelte
<!-- src/routes/(app)/readings/+page.svelte -->
<script lang="ts">
	import { enhance } from '$app/forms';
	import type { PageData } from './$types';

	let { data }: { data: PageData } = $props();

	function getDaysUntilExpiration(expiresAt: Date | null) {
		if (!expiresAt) return null;
		const now = new Date();
		const expires = new Date(expiresAt);
		const days = Math.ceil((expires.getTime() - now.getTime()) / (1000 * 60 * 60 * 24));
		return days;
	}
</script>

<div class="container mx-auto px-4 py-12">
	<div class="flex justify-between items-center mb-8">
		<h1 class="text-4xl font-bold">Your Saved Readings</h1>
		<a href="/reading/new" class="btn btn-primary">New Reading</a>
	</div>

	{#if data.readings.length === 0}
		<div class="card bg-base-200">
			<div class="card-body text-center">
				<p class="text-lg">You haven't saved any readings yet.</p>
				<div>
					<a href="/reading/new" class="btn btn-primary mt-4">Create Your First Reading</a>
				</div>
			</div>
		</div>
	{:else}
		<div class="grid gap-6">
			{#each data.readings as reading}
				{@const cards = JSON.parse(reading.cards)}
				{@const daysLeft = getDaysUntilExpiration(reading.expiresAt)}

				<div class="card bg-base-100 shadow-xl">
					<div class="card-body">
						<div class="flex justify-between items-start">
							<div class="flex-1">
								<h2 class="card-title">
									Reading from {new Date(reading.createdAt).toLocaleDateString()}
								</h2>
								<div class="flex gap-2 mt-2">
									{#each cards as card}
										<div class="badge badge-primary">{card.name}</div>
									{/each}
								</div>
							</div>

							{#if daysLeft !== null}
								<div class="badge {daysLeft <= 7 ? 'badge-warning' : 'badge-info'}">
									Expires in {daysLeft} days
								</div>
							{/if}
						</div>

						<div class="prose prose-sm mt-4">
							{reading.interpretation.substring(0, 200)}...
						</div>

						<div class="card-actions justify-end mt-4">
							<a href="/readings/{reading.id}" class="btn btn-sm btn-primary">
								View Full Reading
							</a>
							<form method="POST" action="?/delete" use:enhance>
								<input type="hidden" name="readingId" value={reading.id} />
								<button type="submit" class="btn btn-sm btn-error">Delete</button>
							</form>
						</div>
					</div>
				</div>
			{/each}
		</div>
	{/if}
</div>
```

### Background Job: Expire Old Readings

```typescript
// src/routes/api/cron/expire-readings/+server.ts
import { json } from '@sveltejs/kit';
import { db } from '$lib/server/db';
import { readings } from '$lib/server/db/schema';
import { lt, isNull } from 'drizzle-orm';
import type { RequestHandler } from './$types';

export const GET: RequestHandler = async ({ request }) => {
	// Verify cron secret (for production)
	const authHeader = request.headers.get('authorization');
	if (authHeader !== `Bearer ${process.env.CRON_SECRET}`) {
		return json({ error: 'Unauthorized' }, { status: 401 });
	}

	// Find expired readings
	const now = new Date();
	const expiredReadings = await db.select().from(readings).where(lt(readings.expiresAt, now));

	// Soft delete them
	for (const reading of expiredReadings) {
		await db.update(readings).set({ deletedAt: now }).where(eq(readings.id, reading.id));
	}

	return json({
		success: true,
		expired: expiredReadings.length,
		timestamp: now.toISOString()
	});
};
```

### Configure Cron Job (Vercel)

```json
// vercel.json
{
	"crons": [
		{
			"path": "/api/cron/expire-readings",
			"schedule": "0 0 * * *"
		}
	]
}
```

---

## 📝 Final Summary

Congratulations! You've built a complete production-ready Tarot SaaS application! 🔮

### ✅ What You've Accomplished

**Core Features:**

- ✅ Full authentication with Better Auth
- ✅ Subscription payments with Stripe
- ✅ Role-based authorization (free vs paid tiers)
- ✅ Card shuffling and reading generation
- ✅ Beautiful card flip animations
- ✅ Reading history with expiration
- ✅ Background jobs for cleanup

**Technical Skills:**

- ✅ Database design with Drizzle ORM
- ✅ Webhook handling (Stripe)
- ✅ Complex business logic (card selection, interpretations)
- ✅ Time-based features (30-day expiration)
- ✅ Form handling with progressive enhancement
- ✅ CSS animations and interactions
- ✅ API endpoint design
- ✅ Cron job implementation

**Business Skills:**

- ✅ SaaS monetization strategy
- ✅ Freemium tier implementation
- ✅ Subscription management
- ✅ Feature gating
- ✅ User retention (saved readings)

### 💰 Real-World Value

This project demonstrates:

- **Subscription SaaS** - $5k-$15k freelance project
- **Payment integration** - High-value skill
- **Complex authorization** - Enterprise pattern
- **Background jobs** - Production requirement
- **Professional UI** - Portfolio-worthy design

### 🚀 Next Steps

**Enhance Your Project:**

1. Add email notifications (reading reminders)
2. Implement PDF export for readings
3. Add social sharing features
4. Create admin dashboard for card management
5. Add user testimonials/reviews
6. Implement analytics tracking

**Deploy Your Project:**

1. Set up Stripe production keys
2. Configure production database (Neon)
3. Deploy to Vercel
4. Set up cron jobs
5. Configure webhook endpoints
6. Add custom domain

**Monetize:**

1. Launch on Product Hunt
2. Create landing page with SEO
3. Run targeted ads (spirituality niche)
4. Partner with tarot influencers
5. Offer affiliate program
6. Create tiered pricing ($9.99, $19.99, $49.99)

---

## 🏆 Congratulations!

You've completed a **production-ready SaaS application** that combines:

- Modern web development (SvelteKit, TypeScript)
- Payment processing (Stripe)
- Database design (PostgreSQL, Drizzle)
- Authentication & authorization
- Background jobs
- Beautiful UX

**This project alone could land you freelance contracts worth $10k-$20k!** 🎉

You now have the skills to build any SaaS application. The patterns you've learned here apply to countless other projects.

**Ready to launch your tarot business?** 🔮✨
