# Svelte 5 Patterns for Tarot Card Application

Real-world examples using Svelte 5 patterns for building a tarot reading website with authentication, tiered access, and database integration.

---

## Table of Contents

1. [Data Models](#data-models)
2. [User Authentication](#user-authentication)
3. [Tiered Access Control](#tiered-access-control)
4. [Card Management](#card-management)
5. [Reading System](#reading-system)
6. [Deck Selection](#deck-selection)
7. [Complete Examples](#complete-examples)

---

## Data Models

### Card Type

```ts
interface TarotCard {
	id: string;
	name: string;
	deck: string;
	arcana: 'major' | 'minor';
	suit?: 'cups' | 'wands' | 'swords' | 'pentacles';
	number?: number;
	upright: {
		keywords: string[];
		meaning: string;
		fullMeaning: string; // Paid users only
	};
	reversed: {
		keywords: string[];
		meaning: string;
		fullMeaning: string; // Paid users only
	};
	imageUrl: string;
	isPremium: boolean; // True if card only visible to paid users
}
```

### Reading Type

```ts
interface Reading {
	id: string;
	userId: string;
	type: 'single' | 'three-card' | 'celtic-cross';
	cards: {
		cardId: string;
		position: string;
		isReversed: boolean;
	}[];
	question?: string;
	interpretation: string;
	fullInterpretation?: string; // Paid users only
	createdAt: Date;
	expiresAt: Date;
}
```

### User Type

```ts
interface User {
	id: string;
	email: string;
	subscription: 'free' | 'paid';
	subscriptionExpiresAt?: Date;
	readingHistory: string[]; // Reading IDs
}
```

---

## User Authentication

### Auth State Management

```ts
// auth.svelte.ts
class AuthManager {
	user: User | null = $state(null);
	loading = $state(true);
	error: string | undefined = $state();

	// Derived state
	isLoggedIn = $derived(this.user !== null);
	isPaidUser = $derived(
		this.user?.subscription === 'paid' &&
			(!this.user.subscriptionExpiresAt || this.user.subscriptionExpiresAt > new Date())
	);

	constructor() {
		// Check if user is already logged in
		$effect(() => {
			this.checkAuth();
		});
	}

	async checkAuth() {
		this.loading = true;

		try {
			const response = await fetch('/api/auth/me', {
				credentials: 'include'
			});

			if (response.ok) {
				this.user = await response.json();
			}
		} catch (e) {
			this.error = 'Failed to check authentication';
		}

		this.loading = false;
	}

	async login(email: string, password: string) {
		this.loading = true;
		this.error = undefined;

		try {
			const response = await fetch('/api/auth/login', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ email, password }),
				credentials: 'include'
			});

			if (!response.ok) {
				throw new Error('Invalid credentials');
			}

			this.user = await response.json();
		} catch (e) {
			this.error = e.message;
		}

		this.loading = false;
	}

	async logout() {
		await fetch('/api/auth/logout', {
			method: 'POST',
			credentials: 'include'
		});

		this.user = null;
	}

	async upgradeSubscription() {
		try {
			const response = await fetch('/api/subscription/upgrade', {
				method: 'POST',
				credentials: 'include'
			});

			if (response.ok) {
				this.user = await response.json();
			}
		} catch (e) {
			this.error = 'Failed to upgrade subscription';
		}
	}
}

export default AuthManager;
```

### Using Auth in Components

```svelte
<script lang="ts">
	import AuthManager from '$lib/auth.svelte';

	const auth = new AuthManager();
</script>

{#if auth.loading}
	<p>Loading...</p>
{:else if !auth.isLoggedIn}
	<LoginForm {auth} />
{:else}
	<div>
		<p>Welcome, {auth.user.email}!</p>

		{#if auth.isPaidUser}
			<span class="badge">Premium Member</span>
		{:else}
			<button onclick={() => auth.upgradeSubscription()}> Upgrade to Premium </button>
		{/if}

		<button onclick={() => auth.logout()}>Logout</button>
	</div>
{/if}
```

---

## Tiered Access Control

### Access Control Class

```ts
class AccessControl {
	user: User | null;

	constructor(user: User | null) {
		this.user = user;
	}

	get isPaidUser() {
		return this.user?.subscription === 'paid';
	}

	canViewCard(card: TarotCard): boolean {
		// Free users can't see premium cards
		if (card.isPremium && !this.isPaidUser) {
			return false;
		}
		return true;
	}

	canViewFullMeaning(card: TarotCard): boolean {
		// Only paid users see full meanings
		return this.isPaidUser;
	}

	canSaveReading(): boolean {
		// Must be logged in to save
		return this.user !== null;
	}

	canViewFullReading(reading: Reading): boolean {
		return this.isPaidUser;
	}

	getRemainingFreeReadings(): number {
		if (this.isPaidUser) {
			return Infinity;
		}

		// Free users get 3 readings per month
		const thisMonth = new Date().getMonth();
		const readingsThisMonth =
			this.user?.readingHistory.filter((r) => {
				// Filter by month logic here
				return true;
			}).length || 0;

		return Math.max(0, 3 - readingsThisMonth);
	}
}
```

### Conditional Card Display

```svelte
<script lang="ts">
	import AuthManager from '$lib/auth.svelte';
	import AccessControl from '$lib/access-control';

	let auth = new AuthManager();
	let access = $derived(new AccessControl(auth.user));
	let card: TarotCard = $state(/* card data */);
</script>

<div class="card">
	{#if access.canViewCard(card)}
		<img src={card.imageUrl} alt={card.name} />
		<h3>{card.name}</h3>

		<div class="meaning">
			<h4>Keywords</h4>
			<p>{card.upright.keywords.join(', ')}</p>

			<h4>Meaning</h4>
			<p>{card.upright.meaning}</p>

			{#if access.canViewFullMeaning(card)}
				<h4>Full Interpretation</h4>
				<p>{card.upright.fullMeaning}</p>
			{:else}
				<div class="premium-overlay">
					<p>Upgrade to premium to unlock full interpretation</p>
					<button>Upgrade Now</button>
				</div>
			{/if}
		</div>
	{:else}
		<div class="locked-card">
			<div class="blur">
				<img src={card.imageUrl} alt="Locked" />
			</div>
			<div class="unlock-message">
				<p>🔒 Premium Card</p>
				<button>Unlock with Premium</button>
			</div>
		</div>
	{/if}
</div>
```

---

## Card Management

### Card Database Manager

```ts
class CardDatabase {
	cards: TarotCard[] = $state([]);
	decks: string[] = $state([]);
	selectedDeck = $state('rider-waite');
	loading = $state(false);
	error: string | undefined = $state();

	// Filtered cards based on selected deck
	deckCards = $derived(this.cards.filter((card) => card.deck === this.selectedDeck));

	// Available cards for current user
	availableCards = $derived.by(() => {
		const access = new AccessControl(this.user);
		return this.deckCards.filter((card) => access.canViewCard(card));
	});

	constructor(private user: User | null) {
		$effect(() => {
			this.loadDecks();
		});

		$effect(() => {
			this.loadCards();
		});
	}

	async loadDecks() {
		try {
			this.decks = await fetch('/api/decks').then((r) => r.json());
		} catch (e) {
			this.error = 'Failed to load decks';
		}
	}

	async loadCards() {
		this.loading = true;
		this.error = undefined;

		try {
			const response = await fetch(`/api/cards?deck=${this.selectedDeck}`, {
				credentials: 'include'
			});

			this.cards = await response.json();
		} catch (e) {
			this.error = 'Failed to load cards';
		}

		this.loading = false;
	}

	async getCardById(id: string): Promise<TarotCard | null> {
		const cached = this.cards.find((c) => c.id === id);
		if (cached) return cached;

		try {
			return await fetch(`/api/cards/${id}`).then((r) => r.json());
		} catch {
			return null;
		}
	}
}
```

### Card Browser Component

```svelte
<script lang="ts">
	import CardDatabase from '$lib/card-database.svelte';
	import AuthManager from '$lib/auth.svelte';

	const auth = new AuthManager();
	const cardDb = new CardDatabase(auth.user);

	let filter = $state('all'); // 'all', 'major', 'minor'

	let filteredCards = $derived(
		filter === 'all'
			? cardDb.availableCards
			: cardDb.availableCards.filter((c) => c.arcana === filter)
	);
</script>

<div class="card-browser">
	<div class="controls">
		<select bind:value={cardDb.selectedDeck}>
			{#each cardDb.decks as deck}
				<option value={deck}>{deck}</option>
			{/each}
		</select>

		<div class="filters">
			<button class:active={filter === 'all'} onclick={() => (filter = 'all')}> All Cards </button>
			<button class:active={filter === 'major'} onclick={() => (filter = 'major')}>
				Major Arcana
			</button>
			<button class:active={filter === 'minor'} onclick={() => (filter = 'minor')}>
				Minor Arcana
			</button>
		</div>
	</div>

	{#if cardDb.loading}
		<p>Loading cards...</p>
	{:else if cardDb.error}
		<p>Error: {cardDb.error}</p>
	{:else}
		<div class="card-grid">
			{#each filteredCards as card (card.id)}
				<div class="card-item">
					<img src={card.imageUrl} alt={card.name} />
					<p>{card.name}</p>
					{#if card.isPremium}
						<span class="premium-badge">Premium</span>
					{/if}
				</div>
			{/each}
		</div>

		<p class="count">
			Showing {filteredCards.length} of {cardDb.deckCards.length} cards
		</p>
	{/if}
</div>
```

---

## Reading System

### Reading Manager

```ts
class ReadingManager {
	currentReading: Reading | null = $state(null);
	readingHistory: Reading[] = $state([]);
	loading = $state(false);
	error: string | undefined = $state();

	constructor(
		private user: User | null,
		private cardDb: CardDatabase
	) {
		if (user) {
			$effect(() => {
				this.loadHistory();
			});
		}
	}

	async performReading(type: Reading['type'], question?: string) {
		this.loading = true;
		this.error = undefined;

		try {
			const response = await fetch('/api/readings/create', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ type, question }),
				credentials: 'include'
			});

			if (!response.ok) {
				throw new Error('Failed to create reading');
			}

			this.currentReading = await response.json();
		} catch (e) {
			this.error = e.message;
		}

		this.loading = false;
	}

	async loadHistory() {
		try {
			this.readingHistory = await fetch('/api/readings/history', {
				credentials: 'include'
			}).then((r) => r.json());
		} catch (e) {
			this.error = 'Failed to load reading history';
		}
	}

	async loadReading(id: string) {
		this.loading = true;

		try {
			this.currentReading = await fetch(`/api/readings/${id}`, {
				credentials: 'include'
			}).then((r) => r.json());
		} catch (e) {
			this.error = 'Failed to load reading';
		}

		this.loading = false;
	}

	// Check if reading has expired
	isReadingExpired(reading: Reading): boolean {
		return new Date() > reading.expiresAt;
	}

	// Get cards for current reading
	async getReadingCards(): Promise<TarotCard[]> {
		if (!this.currentReading) return [];

		const cards = await Promise.all(
			this.currentReading.cards.map((c) => this.cardDb.getCardById(c.cardId))
		);

		return cards.filter((c) => c !== null) as TarotCard[];
	}
}
```

### Three-Card Reading Component

```svelte
<script lang="ts">
	import ReadingManager from '$lib/reading-manager.svelte';
	import AuthManager from '$lib/auth.svelte';
	import AccessControl from '$lib/access-control';

	const auth = new AuthManager();
	const access = $derived(new AccessControl(auth.user));
	const reading = new ReadingManager(auth.user, cardDb);

	let question = $state('');
	let readingCards = $state([]);

	async function startReading() {
		await reading.performReading('three-card', question);
		readingCards = await reading.getReadingCards();
	}
</script>

<div class="reading-container">
	<h2>Three Card Reading</h2>

	{#if !reading.currentReading}
		<div class="reading-form">
			<label>
				Your Question (optional)
				<input type="text" bind:value={question} placeholder="What guidance do you seek?" />
			</label>

			<button onclick={startReading} disabled={reading.loading}>
				{reading.loading ? 'Drawing cards...' : 'Begin Reading'}
			</button>

			{#if !auth.isLoggedIn}
				<p class="note">Sign in to save your reading</p>
			{:else if !access.isPaidUser}
				<p class="note">
					Free readings remaining: {access.getRemainingFreeReadings()}
				</p>
			{/if}
		</div>
	{:else}
		<div class="reading-result">
			{#if question}
				<p class="question">"{question}"</p>
			{/if}

			<div class="cards">
				{#each reading.currentReading.cards as cardData, i}
					{@const card = readingCards[i]}
					{@const positions = ['Past', 'Present', 'Future']}

					<div class="card-position">
						<h3>{positions[i]}</h3>

						{#if card}
							<img src={card.imageUrl} alt={card.name} class:reversed={cardData.isReversed} />
							<p class="card-name">
								{card.name}
								{#if cardData.isReversed}(Reversed){/if}
							</p>

							<div class="interpretation">
								<p>
									{cardData.isReversed ? card.reversed.meaning : card.upright.meaning}
								</p>

								{#if access.canViewFullMeaning(card)}
									<details>
										<summary>Full Interpretation</summary>
										<p>
											{cardData.isReversed ? card.reversed.fullMeaning : card.upright.fullMeaning}
										</p>
									</details>
								{:else}
									<button class="upgrade-prompt"> Unlock full interpretation </button>
								{/if}
							</div>
						{/if}
					</div>
				{/each}
			</div>

			<div class="overall-interpretation">
				<h3>Overall Guidance</h3>
				<p>{reading.currentReading.interpretation}</p>

				{#if access.canViewFullReading(reading.currentReading)}
					{#if reading.currentReading.fullInterpretation}
						<details>
							<summary>Detailed Analysis</summary>
							<p>{reading.currentReading.fullInterpretation}</p>
						</details>
					{/if}
				{:else}
					<div class="premium-unlock">
						<p>Upgrade to premium for detailed analysis</p>
						<button>Upgrade Now</button>
					</div>
				{/if}
			</div>

			<button onclick={() => (reading.currentReading = null)}> New Reading </button>
		</div>
	{/if}
</div>

<style>
	.reversed {
		transform: rotate(180deg);
	}
</style>
```

---

## Deck Selection

### Deck Selector Component

```svelte
<script lang="ts">
	import CardDatabase from '$lib/card-database.svelte';

	const cardDb = new CardDatabase(auth.user);

	let selectedDeck = $state(cardDb.selectedDeck);
	let previewCards = $state([]);

	// Load preview cards when deck changes
	$effect(() => {
		loadPreview(selectedDeck);
	});

	async function loadPreview(deck: string) {
		const response = await fetch(`/api/decks/${deck}/preview`);
		previewCards = await response.json();
	}

	function selectDeck() {
		cardDb.selectedDeck = selectedDeck;
	}
</script>

<div class="deck-selector">
	<h2>Choose Your Deck</h2>

	<div class="deck-options">
		{#each cardDb.decks as deck}
			<label class="deck-option">
				<input type="radio" name="deck" value={deck} bind:group={selectedDeck} />
				<div class="deck-card">
					<h3>{deck}</h3>

					{#if selectedDeck === deck}
						<div class="preview">
							{#each previewCards.slice(0, 3) as card}
								<img src={card.imageUrl} alt={card.name} />
							{/each}
						</div>
					{/if}
				</div>
			</label>
		{/each}
	</div>

	<button onclick={selectDeck}>
		Use {selectedDeck} Deck
	</button>
</div>
```

---

## Complete Examples

### Full Reading Page

```svelte
<script lang="ts">
	import AuthManager from '$lib/auth.svelte';
	import CardDatabase from '$lib/card-database.svelte';
	import ReadingManager from '$lib/reading-manager.svelte';
	import AccessControl from '$lib/access-control';

	const auth = new AuthManager();
	const cardDb = new CardDatabase(auth.user);
	const reading = new ReadingManager(auth.user, cardDb);
	const access = $derived(new AccessControl(auth.user));

	let readingType = $state('three-card');
	let question = $state('');
</script>

{#if auth.loading}
	<div class="loading">Loading...</div>
{:else}
	<div class="reading-page">
		<header>
			<h1>Tarot Reading</h1>

			{#if auth.isLoggedIn}
				<div class="user-info">
					<p>{auth.user.email}</p>
					{#if !access.isPaidUser}
						<p>Readings remaining: {access.getRemainingFreeReadings()}</p>
					{/if}
				</div>
			{/if}
		</header>

		{#if !auth.isLoggedIn}
			<div class="guest-notice">
				<p>Sign in to save your readings and unlock premium features</p>
				<button onclick={() => goto('/login')}>Sign In</button>
			</div>
		{/if}

		<div class="reading-setup">
			<label>
				Choose Reading Type
				<select bind:value={readingType}>
					<option value="single">Single Card</option>
					<option value="three-card">Three Card Spread</option>
					<option value="celtic-cross" disabled={!access.isPaidUser}>
						Celtic Cross {!access.isPaidUser ? '(Premium)' : ''}
					</option>
				</select>
			</label>

			<label>
				Your Question
				<textarea bind:value={question} placeholder="What guidance are you seeking?" />
			</label>

			<button
				onclick={() => reading.performReading(readingType, question)}
				disabled={reading.loading ||
					(!access.isPaidUser && access.getRemainingFreeReadings() === 0)}
			>
				{reading.loading ? 'Drawing cards...' : 'Begin Reading'}
			</button>
		</div>

		{#if reading.currentReading}
			<ReadingResult {reading} {access} {cardDb} />
		{/if}

		{#if auth.isLoggedIn && reading.readingHistory.length > 0}
			<section class="history">
				<h2>Your Past Readings</h2>
				<div class="history-list">
					{#each reading.readingHistory as pastReading}
						{@const isExpired = reading.isReadingExpired(pastReading)}

						<div class="history-item" class:expired={isExpired}>
							<p class="date">
								{pastReading.createdAt.toLocaleDateString()}
							</p>
							<p class="type">{pastReading.type}</p>

							{#if isExpired && !access.isPaidUser}
								<span class="expired-badge">Expired</span>
							{:else}
								<button onclick={() => reading.loadReading(pastReading.id)}> View Reading </button>
							{/if}
						</div>
					{/each}
				</div>
			</section>
		{/if}
	</div>
{/if}
```

### Card Detail Modal

```svelte
<script lang="ts">
	import AccessControl from '$lib/access-control';

	let { card, auth, onClose } = $props();
	const access = $derived(new AccessControl(auth.user));

	let showReversed = $state(false);
</script>

<div class="modal-backdrop" onclick={onClose}>
	<div class="modal-content" onclick={(e) => e.stopPropagation()}>
		<button class="close" onclick={onClose}>×</button>

		<div class="card-detail">
			<img src={card.imageUrl} alt={card.name} class:reversed={showReversed} />

			<h2>{card.name}</h2>

			<div class="orientation-toggle">
				<button class:active={!showReversed} onclick={() => (showReversed = false)}>
					Upright
				</button>
				<button class:active={showReversed} onclick={() => (showReversed = true)}>
					Reversed
				</button>
			</div>

			{@const meaning = showReversed ? card.reversed : card.upright}

			<div class="keywords">
				<h3>Keywords</h3>
				<ul>
					{#each meaning.keywords as keyword}
						<li>{keyword}</li>
					{/each}
				</ul>
			</div>

			<div class="meaning">
				<h3>Meaning</h3>
				<p>{meaning.meaning}</p>
			</div>

			{#if access.canViewFullMeaning(card)}
				<div class="full-meaning">
					<h3>Full Interpretation</h3>
					<p>{meaning.fullMeaning}</p>
				</div>
			{:else}
				<div class="premium-section">
					<div class="blur-overlay">
						<p>{meaning.fullMeaning.substring(0, 100)}...</p>
					</div>
					<div class="unlock-prompt">
						<p>🔒 Unlock full interpretation with Premium</p>
						<button onclick={() => goto('/upgrade')}> Upgrade Now </button>
					</div>
				</div>
			{/if}
		</div>
	</div>
</div>
```

---

## Key Patterns Summary

### For Tarot App Specifically

| Pattern                    | Use Case                                             |
| -------------------------- | ---------------------------------------------------- |
| `$state()` in Auth         | Track logged-in user and subscription status         |
| `$derived()` for Access    | Automatically check permissions based on user tier   |
| `$effect()` for Auto-fetch | Load cards when deck changes, refetch on auth change |
| Custom bind                | Deck selection, reading type selection               |
| Loading states             | Card fetching, reading generation, auth checks       |
| Conditional rendering      | Show/hide premium content based on subscription      |
| Array filtering            | Filter cards by deck, arcana, user access            |
| Date checking              | Validate subscription expiry, reading expiry         |

### Database Integration Points

```ts
// GET /api/cards?deck=rider-waite
// GET /api/cards/:id
// GET /api/decks
// GET /api/readings/history
// POST /api/readings/create
// GET /api/readings/:id
// POST /api/auth/login
// POST /api/auth/logout
// GET /api/auth/me
// POST /api/subscription/upgrade
```

### Remember

- Always check user tier before showing premium content
- Cache cards to reduce API calls
- Handle expired readings gracefully
- Show clear upgrade prompts for free users
- Save readings only for logged-in users
- Implement reading expiry for free users
- Use loading states for better UX

---

## Universal Reactivity for Tarot App

Using universal reactivity patterns to share state across the entire tarot application.

---

### Global User Session Store

**Shared state for authentication across all pages**

```typescript
// $lib/stores/session.svelte.ts
import { browser } from '$app/environment';

const storedUser = browser && localStorage.getItem('tarot_user');
const user = $state(storedUser ? JSON.parse(storedUser) : null);

// Auto-save user to localStorage
$effect.root(() => {
	$effect(() => {
		if (browser) {
			if (user) {
				localStorage.setItem('tarot_user', JSON.stringify(user));
			} else {
				localStorage.removeItem('tarot_user');
			}
		}
	});
});

export default {
	// Getters for reactive access
	get currentUser() {
		return user;
	},

	get isLoggedIn() {
		return user !== null;
	},

	get isPremium() {
		return (
			user?.subscription === 'paid' &&
			(!user.subscriptionExpiresAt || new Date(user.subscriptionExpiresAt) > new Date())
		);
	},

	// Actions
	async login(email, password) {
		const response = await fetch('/api/auth/login', {
			method: 'POST',
			headers: { 'Content-Type': 'application/json' },
			body: JSON.stringify({ email, password })
		});

		if (response.ok) {
			user = await response.json();
		}
	},

	logout() {
		user = null;
	},

	updateSubscription(subscription) {
		if (user) {
			user.subscription = subscription;
		}
	}
};
```

**Usage in Any Component:**

```svelte
<script lang="ts">
	import session from '$lib/stores/session.svelte';
</script>

<nav>
	{#if session.isLoggedIn}
		<span>Welcome, {session.currentUser.email}</span>
		{#if session.isPremium}
			<span class="badge">Premium</span>
		{/if}
		<button onclick={() => session.logout()}>Logout</button>
	{:else}
		<a href="/login">Sign In</a>
	{/if}
</nav>
```

---

### Persistent Reading History

**LocalStorage-backed reading history with classes**

```typescript
// $lib/stores/reading-history.svelte.ts
import { browser } from '$app/environment';

class ReadingHistory {
	#readings = $state([]);

	constructor() {
		// Load from localStorage
		if (browser) {
			const stored = localStorage.getItem('tarot_readings');
			if (stored) {
				this.#readings = JSON.parse(stored);
			}
		}

		// Auto-save to localStorage
		$effect.root(() => {
			$effect(() => {
				if (browser) {
					localStorage.setItem('tarot_readings', JSON.stringify(this.#readings));
				}
			});
		});
	}

	get all() {
		return this.#readings;
	}

	get recent() {
		return this.#readings.slice(0, 5);
	}

	get count() {
		return this.#readings.length;
	}

	// Get readings from this month (for free user limit)
	get thisMonth() {
		const now = new Date();
		const month = now.getMonth();
		const year = now.getFullYear();

		return this.#readings.filter((r) => {
			const date = new Date(r.createdAt);
			return date.getMonth() === month && date.getFullYear() === year;
		});
	}

	add(reading) {
		this.#readings.unshift({
			...reading,
			createdAt: new Date().toISOString()
		});

		// Keep only last 50 readings
		if (this.#readings.length > 50) {
			this.#readings = this.#readings.slice(0, 50);
		}
	}

	remove(id) {
		this.#readings = this.#readings.filter((r) => r.id !== id);
	}

	clear() {
		this.#readings = [];
	}
}

export const readingHistory = new ReadingHistory();
```

**Usage:**

```svelte
<script lang="ts">
	import { readingHistory } from '$lib/stores/reading-history.svelte';
	import session from '$lib/stores/session.svelte';

	// Check if user can do more readings
	let canRead = $derived(session.isPremium || readingHistory.thisMonth.length < 3);
</script>

<div class="reading-limit">
	{#if !session.isPremium}
		<p>Free readings this month: {readingHistory.thisMonth.length} / 3</p>

		{#if !canRead}
			<button onclick={() => goto('/upgrade')}>Upgrade for Unlimited</button>
		{/if}
	{/if}
</div>

<section class="history">
	<h2>Your Recent Readings ({readingHistory.count})</h2>

	{#each readingHistory.recent as reading}
		<div class="reading-card">
			<p>{new Date(reading.createdAt).toLocaleDateString()}</p>
			<p>{reading.type} - {reading.cards.length} cards</p>
		</div>
	{/each}
</section>
```

---

### App-Wide Settings with Getters/Setters

**User preferences that persist across sessions**

```typescript
// $lib/stores/settings.svelte.ts
import { browser } from '$app/environment';

const stored = browser && localStorage.getItem('tarot_settings');
const state = $state(
	stored
		? JSON.parse(stored)
		: {
				theme: 'dark',
				cardBack: 'celestial',
				soundEnabled: true,
				animationsEnabled: true,
				autoSave: true
			}
);

// Persist settings
$effect.root(() => {
	$effect(() => {
		if (browser) {
			localStorage.setItem('tarot_settings', JSON.stringify(state));
			// Apply theme immediately
			document.body.setAttribute('data-theme', state.theme);
		}
	});
});

export default {
	get theme() {
		return state.theme;
	},
	set theme(value) {
		state.theme = value;
	},

	get cardBack() {
		return state.cardBack;
	},
	set cardBack(value) {
		state.cardBack = value;
	},

	get soundEnabled() {
		return state.soundEnabled;
	},
	set soundEnabled(value) {
		state.soundEnabled = value;
	},

	get animationsEnabled() {
		return state.animationsEnabled;
	},
	set animationsEnabled(value) {
		state.animationsEnabled = value;
	},

	get autoSave() {
		return state.autoSave;
	},
	set autoSave(value) {
		state.autoSave = value;
	},

	reset() {
		state.theme = 'dark';
		state.cardBack = 'celestial';
		state.soundEnabled = true;
		state.animationsEnabled = true;
		state.autoSave = true;
	}
};
```

**Settings Page:**

```svelte
<script lang="ts">
	import settings from '$lib/stores/settings.svelte';
</script>

<div class="settings">
	<h2>Preferences</h2>

	<label>
		Theme:
		<select bind:value={settings.theme}>
			<option value="light">Light</option>
			<option value="dark">Dark</option>
			<option value="cosmic">Cosmic</option>
		</select>
	</label>

	<label>
		Card Back Design:
		<select bind:value={settings.cardBack}>
			<option value="celestial">Celestial</option>
			<option value="mystic">Mystic</option>
			<option value="classic">Classic</option>
		</select>
	</label>

	<label>
		<input type="checkbox" bind:checked={settings.soundEnabled} />
		Sound Effects
	</label>

	<label>
		<input type="checkbox" bind:checked={settings.animationsEnabled} />
		Card Animations
	</label>

	<label>
		<input type="checkbox" bind:checked={settings.autoSave} />
		Auto-save Readings
	</label>

	<button onclick={() => settings.reset()}>Reset to Defaults</button>
</div>
<!-- All changes automatically save! -->
```

---

### Lazy Scroll Progress Indicator

**Using createSubscriber for efficient scroll tracking**

```typescript
// $lib/utils/scroll-tracker.svelte.ts
import { createSubscriber } from 'svelte/reactivity';

const [getScrollY, subscribeY] = createSubscriber(0);
const [getScrollPercent, subscribePercent] = createSubscriber(0);

// Only subscribes when something reads scrollY
subscribeY((set) => {
	function handleScroll() {
		set(window.scrollY);
	}

	window.addEventListener('scroll', handleScroll, { passive: true });
	return () => window.removeEventListener('scroll', handleScroll);
});

// Only subscribes when something reads scrollPercent
subscribePercent((set) => {
	function handleScroll() {
		const height = document.documentElement.scrollHeight - window.innerHeight;
		const percent = height > 0 ? (window.scrollY / height) * 100 : 0;
		set(percent);
	}

	window.addEventListener('scroll', handleScroll, { passive: true });
	window.addEventListener('resize', handleScroll);

	return () => {
		window.removeEventListener('scroll', handleScroll);
		window.removeEventListener('resize', handleScroll);
	};
});

export { getScrollY, getScrollPercent };
```

**Usage - Scroll Progress Bar:**

```svelte
<script lang="ts">
	import { getScrollPercent } from '$lib/utils/scroll-tracker.svelte';

	// Only subscribes to scroll events when this component is mounted
	let progress = $derived(getScrollPercent());
</script>

<div class="scroll-progress" style="width: {progress}%"></div>

<style>
	.scroll-progress {
		position: fixed;
		top: 0;
		left: 0;
		height: 3px;
		background: linear-gradient(90deg, #6366f1, #a855f7);
		transition: width 0.2s ease;
		z-index: 1000;
	}
</style>
```

---

### Selected Deck State with Proxy Validation

**Ensure only valid decks can be selected**

```typescript
// $lib/stores/selected-deck.svelte.ts
const VALID_DECKS = ['rider-waite', 'thoth', 'marseille', 'modern'];

const stored = browser && localStorage.getItem('tarot_deck');
const state = $state({ deck: stored || 'rider-waite' });

const deckProxy = new Proxy(state, {
	get(target, prop) {
		return target[prop];
	},

	set(target, prop, value) {
		if (prop === 'deck') {
			// Validate deck
			if (!VALID_DECKS.includes(value)) {
				console.error(`Invalid deck: ${value}. Using default.`);
				target[prop] = 'rider-waite';
			} else {
				target[prop] = value;
			}

			// Save to localStorage
			if (browser) {
				localStorage.setItem('tarot_deck', target[prop]);
			}
		}

		return true;
	}
});

export const selectedDeck = deckProxy;
export { VALID_DECKS };
```

**Usage:**

```svelte
<script lang="ts">
	import { selectedDeck, VALID_DECKS } from '$lib/stores/selected-deck.svelte';
</script>

<select bind:value={selectedDeck.deck}>
	{#each VALID_DECKS as deck}
		<option value={deck}>{deck}</option>
	{/each}
</select>

<!-- This would show error and fallback to rider-waite -->
<button onclick={() => (selectedDeck.deck = 'invalid-deck')}>Break It (Try)</button>
```

---

### Current Reading State Manager

**Class-based store for active reading session**

```typescript
// $lib/stores/current-reading.svelte.ts
import { createSubscriber } from 'svelte/reactivity';

class CurrentReading {
	#reading = $state(null);
	#loading = $state(false);

	get reading() {
		return this.#reading;
	}

	get isActive() {
		return this.#reading !== null;
	}

	get loading() {
		return this.#loading;
	}

	get cards() {
		return this.#reading?.cards || [];
	}

	get type() {
		return this.#reading?.type;
	}

	async create(type, question) {
		this.#loading = true;

		try {
			const response = await fetch('/api/readings/create', {
				method: 'POST',
				headers: { 'Content-Type': 'application/json' },
				body: JSON.stringify({ type, question })
			});

			this.#reading = await response.json();
		} catch (error) {
			console.error('Failed to create reading:', error);
		} finally {
			this.#loading = false;
		}
	}

	clear() {
		this.#reading = null;
	}

	saveToHistory(historyStore) {
		if (this.#reading) {
			historyStore.add(this.#reading);
		}
	}
}

export const currentReading = new CurrentReading();
```

**Usage:**

```svelte
<script lang="ts">
	import { currentReading } from '$lib/stores/current-reading.svelte';
	import { readingHistory } from '$lib/stores/reading-history.svelte';

	let question = $state('');

	async function startReading() {
		await currentReading.create('three-card', question);
	}

	function saveAndClear() {
		currentReading.saveToHistory(readingHistory);
		currentReading.clear();
		question = '';
	}
</script>

{#if !currentReading.isActive}
	<input bind:value={question} placeholder="Your question..." />
	<button onclick={startReading} disabled={currentReading.loading}>
		{currentReading.loading ? 'Drawing cards...' : 'Begin Reading'}
	</button>
{:else}
	<div class="reading-result">
		{#each currentReading.cards as card}
			<CardDisplay {card} />
		{/each}

		<button onclick={saveAndClear}>Save & New Reading</button>
	</div>
{/if}
```

---

## Universal Reactivity Patterns Summary for Tarot App

### Store Structure

```
$lib/stores/
  session.svelte.ts          // User auth & subscription
  settings.svelte.ts         // App preferences
  selected-deck.svelte.ts    // Current deck selection
  reading-history.svelte.ts  // Past readings
  current-reading.svelte.ts  // Active reading session

$lib/utils/
  scroll-tracker.svelte.ts   // Lazy scroll tracking
```

### Key Patterns Used

| Store           | Pattern Used     | Why                                   |
| --------------- | ---------------- | ------------------------------------- |
| session         | Object + getters | Property-like access, computed values |
| settings        | Getters/setters  | Clean syntax for reading/writing      |
| selected-deck   | Proxy            | Validate deck selection               |
| reading-history | Class            | Complex logic, multiple methods       |
| current-reading | Class            | Encapsulate reading workflow          |
| scroll-tracker  | createSubscriber | Lazy, efficient scroll tracking       |
| All stores      | $effect.root()   | Persist to localStorage automatically |

### Best Practices for Tarot App

1. **Always check `browser`** before localStorage/DOM access
2. **Use getters for reactive values** - `session.isPremium` not `session.isPremium()`
3. **Persist user data** with `$effect.root()` and localStorage
4. **Validate inputs** with Proxy for deck selection
5. **Lazy subscribe** to expensive operations (scroll, resize) with `createSubscriber`
6. **Single instances** - Export one instance of each store for the whole app
