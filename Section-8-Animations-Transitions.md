# Section 8: Animations & Transitions Study Guide

**Complete lesson plan for learning Svelte 5 animations, transitions, and motion design**

---

## 📚 Learning Objectives

By the end of this section, you will:

- ✅ Add smooth enter/leave transitions to elements
- ✅ Animate list reordering with FLIP animations
- ✅ Create custom CSS and JavaScript transitions
- ✅ Use key blocks for content transitions
- ✅ Implement crossfade for seamless element movement
- ✅ Build professional, polished user interfaces
- ✅ Know when and how to use each animation technique

---

## Table of Contents

1. [Transition Directive Basics](#1-transition-directive-basics)
2. [Real-World Tier Animations](#2-real-world-tier-animations)
3. [Key Block Transitions](#3-key-block-transitions)
4. [FLIP Animations](#4-flip-animations)
5. [Crossfade Transitions](#5-crossfade-transitions)
6. [Custom CSS Transitions](#6-custom-css-transitions)
7. [Custom JS Transitions](#7-custom-js-transitions)
8. [Custom Animations](#8-custom-animations)

---

## 1. Transition Directive Basics

### What are Transitions?

Transitions animate elements when they appear (`in:`) or disappear (`out:`) from the DOM. They make your app feel smooth and professional.

**Real-World Scenario:** You're building a notification system. When a new notification appears, it should slide in smoothly, not just pop into existence!

### 📁 Files to Create

**Reusable Notification Component:**

- `src/lib/components/Toast.svelte` - Single toast notification
- `src/lib/components/ToastContainer.svelte` - Container for all toasts

**Notification Manager** (optional, advanced):

- `src/lib/stores/notifications.svelte.ts` - Global notification state

**Demo Page:**

- `src/routes/transitions-demo/+page.svelte` - Toast demo

**Best Practices:**

- Create reusable notification components
- Use transition directives for enter/exit animations
- Consider a global store for app-wide notifications
- Keep animations under 300ms for snappy feel

### Built-in Transitions

Svelte provides 5 built-in transitions:

- **`fade`**: Opacity 0 → 1
- **`fly`**: Slides from a direction
- **`slide`**: Height-based slide
- **`scale`**: Grows/shrinks
- **`blur`**: Blurs in/out

### Real-World Example: Toast Notifications

**What it does:** Notification system with slide-in animations. Common in every modern app!

```svelte
<script lang="ts">
	import { fly, fade } from 'svelte/transition';
	import { cubicOut } from 'svelte/easing';

	type NotificationType = 'success' | 'error' | 'warning' | 'info';

	interface Notification {
		id: number;
		type: NotificationType;
		title: string;
		message: string;
	}

	let notifications = $state<Notification[]>([]);
	let nextId = 0;

	function addNotification(type: NotificationType, title: string, message: string) {
		const notification: Notification = {
			id: nextId++,
			type,
			title,
			message
		};

		notifications = [...notifications, notification];

		// Auto-remove after 5 seconds
		setTimeout(() => {
			removeNotification(notification.id);
		}, 5000);
	}

	function removeNotification(id: number) {
		notifications = notifications.filter((n) => n.id !== id);
	}

	// Demo buttons
	function showSuccess() {
		addNotification('success', 'Order Placed!', 'Your order has been confirmed.');
	}

	function showError() {
		addNotification('error', 'Payment Failed', 'Please check your card details.');
	}

	function showWarning() {
		addNotification('warning', 'Low Stock', 'Only 3 items remaining.');
	}

	function showInfo() {
		addNotification('info', 'Update Available', 'A new version is ready to install.');
	}
</script>

<div class="bg-gray-800 py-8 px-8 rounded-xl border border-gray-700 max-w-2xl mx-auto my-10">
	<h2 class="m-0 mb-2 text-blue-400 text-3xl">Toast Notifications</h2>
	<p class="m-0 mb-6 text-gray-500 text-sm">Click buttons to see transition animations</p>

	<div class="grid grid-cols-2 gap-3">
		<button
			onclick={showSuccess}
			class="border-none px-5 py-3.5 rounded-lg text-base font-semibold cursor-pointer transition-all bg-green-400 text-black hover:-translate-y-0.5 hover:shadow-lg"
		>
			✓ Success
		</button>
		<button
			onclick={showError}
			class="border-none px-5 py-3.5 rounded-lg text-base font-semibold cursor-pointer transition-all bg-red-400 text-white hover:-translate-y-0.5 hover:shadow-lg"
		>
			✕ Error
		</button>
		<button
			onclick={showWarning}
			class="border-none px-5 py-3.5 rounded-lg text-base font-semibold cursor-pointer transition-all bg-orange-500 text-black hover:-translate-y-0.5 hover:shadow-lg"
		>
			⚠ Warning
		</button>
		<button
			onclick={showInfo}
			class="border-none px-5 py-3.5 rounded-lg text-base font-semibold cursor-pointer transition-all bg-blue-400 text-black hover:-translate-y-0.5 hover:shadow-lg"
		>
			ℹ Info
		</button>
	</div>
</div>

<!-- Notification container (fixed position) -->
<div class="fixed top-5 right-5 flex flex-col gap-3 z-50 max-w-sm">
	{#each notifications as notification (notification.id)}
		<div
			class="bg-gray-800 border-l-4 rounded-lg px-5 py-4 shadow-lg min-w-72"
			class:border-green-400={notification.type === 'success'}
			class:border-red-400={notification.type === 'error'}
			class:border-orange-500={notification.type === 'warning'}
			class:border-blue-400={notification.type === 'info'}
			in:fly={{ x: 300, duration: 300, easing: cubicOut }}
			out:fade={{ duration: 200 }}
		>
			<div class="flex items-center gap-2.5 mb-2">
				<span
					class="text-lg font-bold"
					class:text-green-400={notification.type === 'success'}
					class:text-red-400={notification.type === 'error'}
					class:text-orange-500={notification.type === 'warning'}
					class:text-blue-400={notification.type === 'info'}
				>
					{#if notification.type === 'success'}✓{/if}
					{#if notification.type === 'error'}✕{/if}
					{#if notification.type === 'warning'}⚠{/if}
					{#if notification.type === 'info'}ℹ{/if}
				</span>
				<strong class="flex-1 text-white text-base">{notification.title}</strong>
				<button
					class="bg-transparent border-none text-gray-500 text-lg cursor-pointer p-0 w-5 h-5 transition-colors hover:text-white"
					onclick={() => removeNotification(notification.id)}
				>
					✕
				</button>
			</div>
			<p class="m-0 text-gray-300 text-sm leading-normal">{notification.message}</p>
		</div>
	{/each}
</div>
```

**Key Concepts:**

- **`in:`** - Transition when element enters DOM
- **`out:`** - Transition when element leaves DOM
- **`transition:`** - Same transition for both in/out
- **Duration**: How long animation takes (milliseconds)
- **Easing**: Animation curve (`cubicOut`, `linear`, etc.)
- **`{#each}` with key**: Required for list transitions

---

## 2. Real-World Tier Animations

### Real-World Scenario: Pricing Page

Animating pricing tiers when they appear makes your page feel premium and professional.

**What it does:** Pricing cards fade and fly in with staggered timing.

```svelte
<script lang="ts">
	import { fly, scale } from 'svelte/transition';
	import { backOut } from 'svelte/easing';

	interface PricingTier {
		id: string;
		name: string;
		price: number;
		period: string;
		features: string[];
		popular: boolean;
		color: string;
	}

	const tiers: PricingTier[] = [
		{
			id: 'starter',
			name: 'Starter',
			price: 9,
			period: 'month',
			features: ['Up to 100 projects', '5GB storage', 'Email support', 'Basic analytics'],
			popular: false,
			color: '#4a9eff'
		},
		{
			id: 'professional',
			name: 'Professional',
			price: 29,
			period: 'month',
			features: [
				'Unlimited projects',
				'50GB storage',
				'Priority support',
				'Advanced analytics',
				'Custom domains',
				'Team collaboration'
			],
			popular: true,
			color: '#4ade80'
		},
		{
			id: 'enterprise',
			name: 'Enterprise',
			price: 99,
			period: 'month',
			features: [
				'Everything in Pro',
				'Unlimited storage',
				'24/7 phone support',
				'Custom integrations',
				'Dedicated account manager',
				'SLA guarantee'
			],
			popular: false,
			color: '#ffa500'
		}
	];

	let show = $state(false);

	// Show cards after component mounts
	import { onMount } from 'svelte';
	onMount(() => {
		setTimeout(() => (show = true), 100);
	});
</script>

<div class="bg-gray-900 min-h-screen px-5 py-16">
	<div class="text-center mb-16">
		<h1 class="m-0 mb-3 text-white text-5xl font-bold">Choose Your Plan</h1>
		<p class="m-0 text-gray-500 text-xl">Start free, upgrade when you need more</p>
	</div>

	{#if show}
		<div class="grid grid-cols-[repeat(auto-fit,minmax(300px,1fr))] gap-8 max-w-7xl mx-auto mb-10">
			{#each tiers as tier, i (tier.id)}
				<div
					class="bg-gray-800 border-2 border-gray-700 rounded-2xl p-8 relative transition-all hover:-translate-y-2 hover:shadow-2xl"
					class:border-[var(--tier-color)]={tier.popular}
					class:bg-gradient-to-br={tier.popular}
					class:from-gray-800={tier.popular}
					class:to-gray-900={tier.popular}
					style:border-color={tier.popular ? `var(--tier-color)` : ''}
					style:box-shadow={tier.popular ? '' : ''}
					in:fly={{
						y: 50,
						duration: 500,
						delay: i * 150,
						easing: backOut
					}}
					style="--tier-color: {tier.color}"
				>
					{#if tier.popular}
						<div
							class="absolute -top-3 left-1/2 -translate-x-1/2 bg-[var(--tier-color)] text-black px-5 py-1.5 rounded-full text-xs font-bold uppercase"
						>
							Most Popular
						</div>
					{/if}

					<h2 class="m-0 mb-5 text-[var(--tier-color)] text-3xl">{tier.name}</h2>
					<div class="mb-8">
						<span class="text-white text-6xl font-bold">${tier.price}</span>
						<span class="text-gray-500 text-lg">/{tier.period}</span>
					</div>

					<ul class="list-none p-0 m-0 mb-8">
						{#each tier.features as feature}
							<li class="text-gray-300 py-3 border-b border-gray-700 text-base last:border-b-0">
								✓ {feature}
							</li>
						{/each}
					</ul>

					<button
						class="w-full bg-[var(--tier-color)] text-black border-none p-4 rounded-lg text-base font-bold cursor-pointer transition-all hover:scale-105 hover:shadow-xl"
					>
						Get Started
					</button>
				</div>
			{/each}
		</div>
	{/if}

	<button
		class="block mx-auto bg-blue-400 text-black border-none py-3.5 px-8 rounded-lg text-base font-semibold cursor-pointer"
		onclick={() => (show = !show)}
	>
		{show ? 'Hide' : 'Show'} Plans
	</button>
</div>
```

**Key Concepts:**

- **Staggered delay**: `delay: i * 150` creates wave effect
- **Easing functions**: `backOut` creates bounce effect
- **CSS custom properties**: `style="--tier-color: {tier.color}"`
- **Conditional transitions**: Only animate when `show` is true
- **Real-world use**: Pricing pages, feature showcases, product galleries

---

## 3. Key Block Transitions

### What are Key Blocks?

Key blocks (`{#key}`) destroy and recreate elements when a value changes, allowing transitions between different states.

**Real-World Scenario:** Image gallery where photos slide in/out when navigating.

**What it does:** Creates a smooth image carousel with slide transitions.

```svelte
<script lang="ts">
	import { fly } from 'svelte/transition';
	import { cubicOut } from 'svelte/easing';

	interface GalleryImage {
		id: number;
		url: string;
		title: string;
		description: string;
	}

	const images: GalleryImage[] = [
		{
			id: 1,
			url: 'https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=800',
			title: 'Mountain Lake',
			description: 'Peaceful morning at the mountain lake'
		},
		{
			id: 2,
			url: 'https://images.unsplash.com/photo-1469474968028-56623f02e42e?w=800',
			title: 'Forest Trail',
			description: 'Sunlight filtering through ancient trees'
		},
		{
			id: 3,
			url: 'https://images.unsplash.com/photo-1507525428034-b723cf961d3e?w=800',
			title: 'Ocean Sunset',
			description: 'Golden hour by the sea'
		},
		{
			id: 4,
			url: 'https://images.unsplash.com/photo-1511593358241-7eea1f3c84e5?w=800',
			title: 'Desert Dunes',
			description: 'Endless sand formations'
		}
	];

	let currentIndex = $state(0);
	let direction = $state(1); // 1 = forward, -1 = backward

	const currentImage = $derived(images[currentIndex]);

	function next() {
		direction = 1;
		currentIndex = (currentIndex + 1) % images.length;
	}

	function prev() {
		direction = -1;
		currentIndex = (currentIndex - 1 + images.length) % images.length;
	}

	function goTo(index: number) {
		direction = index > currentIndex ? 1 : -1;
		currentIndex = index;
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10">
	<h2 class="text-center m-0 mb-10 text-blue-400 text-4xl">Photo Gallery</h2>

	<div class="relative max-w-4xl mx-auto mb-8 flex items-center gap-5">
		<button
			class="bg-gray-800/90 border-2 border-blue-400 text-white w-16 h-16 rounded-full text-2xl cursor-pointer transition-all shrink-0 backdrop-blur-md hover:bg-blue-400 hover:text-black hover:scale-110"
			onclick={prev}
		>
			←
		</button>

		<div class="flex-1 relative h-96 overflow-hidden rounded-2xl bg-gray-800">
			{#key currentIndex}
				<div
					class="absolute w-full h-full"
					in:fly={{ x: direction * 100, duration: 400, easing: cubicOut }}
					out:fly={{ x: direction * -100, duration: 400, easing: cubicOut }}
				>
					<img
						src={currentImage.url}
						alt={currentImage.title}
						class="w-full h-full object-cover block"
					/>
					<div
						class="absolute bottom-0 left-0 right-0 bg-gradient-to-t from-black/90 to-transparent pt-10 px-8 pb-6 text-white"
					>
						<h3 class="m-0 mb-2 text-3xl">{currentImage.title}</h3>
						<p class="m-0 text-base text-gray-300">{currentImage.description}</p>
					</div>
				</div>
			{/key}
		</div>

		<button
			class="bg-gray-800/90 border-2 border-blue-400 text-white w-16 h-16 rounded-full text-2xl cursor-pointer transition-all shrink-0 backdrop-blur-md hover:bg-blue-400 hover:text-black hover:scale-110"
			onclick={next}
		>
			→
		</button>
	</div>

	<div class="flex justify-center gap-3 mb-5">
		{#each images as image, i}
			<button
				class="w-24 h-20 rounded-lg overflow-hidden border-2 border-transparent cursor-pointer transition-all p-0 bg-gray-800 hover:border-gray-500 hover:scale-105"
				class:!border-blue-400={i === currentIndex}
				onclick={() => goTo(i)}
			>
				<img src={image.url} alt={image.title} class="w-full h-full object-cover" />
			</button>
		{/each}
	</div>

	<div class="text-center text-gray-500 text-lg font-semibold">
		{currentIndex + 1} / {images.length}
	</div>
</div>
```

**Key Concepts:**

- **`{#key value}`**: Destroys and recreates when value changes
- **Direction awareness**: Track `direction` for correct slide animation
- **Modulo arithmetic**: `(index + 1) % length` for wrapping
- **Real-world use**: Image galleries, step-by-step wizards, content sliders

---

## 4. FLIP Animations

### What is FLIP?

FLIP = **F**irst, **L**ast, **I**nvert, **P**lay. Smoothly animates elements when they change position in a list.

**Real-World Scenario:** Todo list where items can be reordered, filtered, or moved between lists.

**What it does:** Drag-and-drop todo list with smooth animations when items reorder.

```svelte
<script lang="ts">
	import { flip } from 'svelte/animate';
	import { fly, scale } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';

	interface Todo {
		id: number;
		text: string;
		completed: boolean;
		priority: 'low' | 'medium' | 'high';
	}

	let todos = $state<Todo[]>([
		{ id: 1, text: 'Review pull requests', completed: false, priority: 'high' },
		{ id: 2, text: 'Update documentation', completed: false, priority: 'medium' },
		{ id: 3, text: 'Fix responsive layout bug', completed: false, priority: 'high' },
		{ id: 4, text: 'Plan next sprint', completed: false, priority: 'low' },
		{ id: 5, text: 'Write unit tests', completed: false, priority: 'medium' }
	]);

	let filter = $state<'all' | 'active' | 'completed'>('all');
	let sortBy = $state<'none' | 'priority' | 'alphabetical'>('none');

	const filteredTodos = $derived(() => {
		let result = todos;

		// Filter
		if (filter === 'active') {
			result = result.filter((t) => !t.completed);
		} else if (filter === 'completed') {
			result = result.filter((t) => t.completed);
		}

		// Sort
		if (sortBy === 'priority') {
			const priorityOrder = { high: 0, medium: 1, low: 2 };
			result = [...result].sort((a, b) => priorityOrder[a.priority] - priorityOrder[b.priority]);
		} else if (sortBy === 'alphabetical') {
			result = [...result].sort((a, b) => a.text.localeCompare(b.text));
		}

		return result;
	});

	function toggleComplete(id: number) {
		const todo = todos.find((t) => t.id === id);
		if (todo) {
			todo.completed = !todo.completed;
		}
	}

	function removeTodo(id: number) {
		todos = todos.filter((t) => t.id !== id);
	}

	function shuffle() {
		todos = todos.sort(() => Math.random() - 0.5);
	}
</script>

<div class="bg-gray-800 p-8 rounded-2xl border border-gray-700 max-w-3xl mx-auto my-10">
	<h2 class="m-0 mb-6 text-blue-400 text-3xl">Project Tasks</h2>

	<div class="flex gap-4 mb-6 flex-wrap">
		<div class="flex items-center gap-2">
			<label class="text-gray-500 text-sm font-semibold">Filter:</label>
			<select
				bind:value={filter}
				class="bg-gray-900 text-white border-2 border-gray-700 px-3 py-2 rounded-md text-sm cursor-pointer"
			>
				<option value="all">All Tasks</option>
				<option value="active">Active</option>
				<option value="completed">Completed</option>
			</select>
		</div>

		<div class="flex items-center gap-2">
			<label class="text-gray-500 text-sm font-semibold">Sort:</label>
			<select
				bind:value={sortBy}
				class="bg-gray-900 text-white border-2 border-gray-700 px-3 py-2 rounded-md text-sm cursor-pointer"
			>
				<option value="none">None</option>
				<option value="priority">Priority</option>
				<option value="alphabetical">A-Z</option>
			</select>
		</div>

		<button
			onclick={shuffle}
			class="bg-blue-400 text-black border-none px-4 py-2 rounded-md text-sm font-semibold cursor-pointer ml-auto"
		>
			🔀 Shuffle
		</button>
	</div>

	<ul class="list-none p-0 m-0 mb-5">
		{#each filteredTodos as todo (todo.id)}
			<li
				class="bg-gray-900 p-4 mb-3 rounded-lg border-l-4 flex items-center gap-3 transition-all hover:bg-gray-800"
				class:border-l-red-400={todo.priority === 'high'}
				class:border-l-orange-500={todo.priority === 'medium'}
				class:border-l-green-400={todo.priority === 'low'}
				class:opacity-60={todo.completed}
				animate:flip={{ duration: 400, easing: quintOut }}
				in:scale={{ duration: 300 }}
				out:scale={{ duration: 200 }}
			>
				<input
					type="checkbox"
					checked={todo.completed}
					onchange={() => toggleComplete(todo.id)}
					class="w-6 h-6 cursor-pointer accent-blue-400"
				/>

				<div class="flex-1 flex items-center gap-3">
					<span
						class="text-white text-base"
						class:line-through={todo.completed}
						class:!text-gray-500={todo.completed}>{todo.text}</span
					>
					<span
						class="text-xs font-bold uppercase px-2 py-1 rounded text-black"
						class:bg-red-400={todo.priority === 'high'}
						class:bg-orange-500={todo.priority === 'medium'}
						class:bg-green-400={todo.priority === 'low'}>{todo.priority}</span
					>
				</div>

				<button
					onclick={() => removeTodo(todo.id)}
					class="bg-transparent border-none text-gray-500 text-xl cursor-pointer px-2 py-1 transition-colors hover:text-red-400"
				>
					🗑️
				</button>
			</li>
		{/each}
	</ul>

	<div class="text-center text-gray-500 text-sm">
		{filteredTodos.length}
		{filteredTodos.length === 1 ? 'task' : 'tasks'}
	</div>
</div>
```

**Key Concepts:**

- **`animate:flip`**: Smoothly animates position changes
- **Must use with key**: `{#each items as item (item.id)}`
- **Combine with transitions**: `in:` and `out:` for add/remove
- **Duration and easing**: Control speed and feel
- **Real-world use**: Sortable lists, drag-and-drop, filters

---

## 5. Crossfade Transitions

### What is Crossfade?

Crossfade creates paired transitions where an element appears to move from one position to another, like drag-and-drop!

**Real-World Scenario:** You're building a Kanban board where tasks move between columns (To Do → In Progress → Done). You want smooth animations when dragging items!

**What it does:** Creates smooth transitions when items move between different lists.

```svelte
<script lang="ts">
	import { crossfade } from 'svelte/transition';
	import { quintOut } from 'svelte/easing';

	interface Task {
		id: number;
		title: string;
		description: string;
	}

	interface Column {
		id: string;
		title: string;
		color: string;
		tasks: Task[];
	}

	// Create paired transitions
	const [send, receive] = crossfade({
		duration: 400,
		easing: quintOut,
		fallback: (node) => {
			// Fallback for items that don't have a pair
			return {
				duration: 300,
				css: (t: number) => `opacity: ${t}`
			};
		}
	});

	let columns = $state<Column[]>([
		{
			id: 'todo',
			title: 'To Do',
			color: '#4a9eff',
			tasks: [
				{ id: 1, title: 'Design homepage', description: 'Create mockups' },
				{ id: 2, title: 'Setup database', description: 'PostgreSQL config' }
			]
		},
		{
			id: 'progress',
			title: 'In Progress',
			color: '#ffa500',
			tasks: [{ id: 3, title: 'Build API', description: 'REST endpoints' }]
		},
		{
			id: 'done',
			title: 'Done',
			color: '#4ade80',
			tasks: [{ id: 4, title: 'Project setup', description: 'Init repository' }]
		}
	]);

	let draggedTask: Task | null = null;
	let draggedFrom: string | null = null;

	function handleDragStart(task: Task, columnId: string) {
		draggedTask = task;
		draggedFrom = columnId;
	}

	function handleDragOver(event: DragEvent) {
		event.preventDefault();
	}

	function handleDrop(event: DragEvent, targetColumnId: string) {
		event.preventDefault();

		if (!draggedTask || !draggedFrom || draggedFrom === targetColumnId) {
			return;
		}

		// Remove from source column
		const sourceColumn = columns.find((c) => c.id === draggedFrom);
		if (sourceColumn) {
			sourceColumn.tasks = sourceColumn.tasks.filter((t) => t.id !== draggedTask!.id);
		}

		// Add to target column
		const targetColumn = columns.find((c) => c.id === targetColumnId);
		if (targetColumn) {
			targetColumn.tasks = [...targetColumn.tasks, draggedTask];
		}

		// Trigger reactivity
		columns = columns;

		draggedTask = null;
		draggedFrom = null;
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-10 text-4xl">🚀 Project Kanban Board</h1>

	<div class="grid grid-cols-[repeat(auto-fit,minmax(300px,1fr))] gap-6 max-w-7xl mx-auto">
		{#each columns as column}
			<div
				class="bg-gray-800 rounded-xl overflow-hidden border-2 border-gray-700 min-h-96"
				ondragover={handleDragOver}
				ondrop={(e) => handleDrop(e, column.id)}
			>
				<div
					class="p-5 text-black flex justify-between items-center"
					style="background: {column.color};"
				>
					<h2 class="m-0 text-xl font-extrabold">{column.title}</h2>
					<span class="bg-black/20 px-3 py-1.5 rounded-full font-bold text-sm"
						>{column.tasks.length}</span
					>
				</div>

				<div class="p-4 flex flex-col gap-3">
					{#each column.tasks as task (task.id)}
						<div
							class="bg-gray-900 border-2 border-gray-700 rounded-lg p-4 cursor-grab transition-all active:cursor-grabbing hover:border-blue-500 hover:-translate-y-0.5 hover:shadow-lg hover:shadow-blue-500/30"
							draggable="true"
							ondragstart={() => handleDragStart(task, column.id)}
							in:receive={{ key: task.id }}
							out:send={{ key: task.id }}
						>
							<h3 class="m-0 mb-2 text-white text-base">{task.title}</h3>
							<p class="m-0 mb-3 text-gray-400 text-sm leading-relaxed">{task.description}</p>
							<div class="flex justify-between items-center">
								<span class="text-gray-600 text-xs font-semibold">#{task.id}</span>
							</div>
						</div>
					{/each}

					{#if column.tasks.length === 0}
						<div
							class="text-center py-10 px-5 text-gray-600 italic border-2 border-dashed border-gray-700 rounded-lg"
						>
							Drop tasks here
						</div>
					{/if}
				</div>
			</div>
		{/each}
	</div>
</div>
```

**Key Concepts:**

- **`crossfade()`**: Creates paired `send` and `receive` transitions
- **`in:receive`** and **`out:send`**: Apply paired transitions
- **Key parameter**: `{ key: item.id }` links the transitions
- **Fallback**: Handles items without a pair
- **Real-world use**: Drag-and-drop, shopping carts, task boards

---

## 6. Custom CSS Transitions

### What are Custom CSS Transitions?

Create your own transitions by returning custom CSS! Full control over animation timing and properties.

**Real-World Scenario:** You want unique entrance animations for different content types - a bounce for successes, shake for errors, slide-rotate for info cards.

**What it does:** Creates custom CSS-based transitions with complete control.

```svelte
<script lang="ts">
	import { cubicOut } from 'svelte/easing';
	import type { TransitionConfig } from 'svelte/transition';

	// Custom bounce transition
	function bounce(node: HTMLElement, { delay = 0, duration = 600 }): TransitionConfig {
		return {
			delay,
			duration,
			css: (t: number) => {
				const eased = cubicOut(t);
				// Create bounce effect with scale and translateY
				const scale = 0.7 + 0.3 * eased;
				const translateY = (1 - eased) * -50;
				const opacity = eased;

				return `
					transform: scale(${scale}) translateY(${translateY}px);
					opacity: ${opacity};
				`;
			}
		};
	}

	// Custom slide-rotate transition
	function slideRotate(node: HTMLElement, { delay = 0, duration = 500 }): TransitionConfig {
		return {
			delay,
			duration,
			css: (t: number) => {
				const eased = cubicOut(t);
				const translateX = (1 - eased) * 200;
				const rotate = (1 - eased) * 180;
				const opacity = eased;

				return `
					transform: translateX(${translateX}px) rotate(${rotate}deg);
					opacity: ${opacity};
				`;
			}
		};
	}

	// Custom shake transition
	function shake(node: HTMLElement, { delay = 0, duration = 400 }): TransitionConfig {
		return {
			delay,
			duration,
			css: (t: number) => {
				// Shake decreases over time
				const intensity = (1 - t) * 20;
				const x = Math.sin(t * 10) * intensity;

				return `
					transform: translateX(${x}px);
					opacity: ${t};
				`;
			}
		};
	}

	interface Notification {
		id: number;
		type: 'success' | 'error' | 'info';
		title: string;
		message: string;
	}

	let notifications = $state<Notification[]>([]);
	let nextId = 1;

	function addNotification(type: 'success' | 'error' | 'info') {
		const notification: Notification = {
			id: nextId++,
			type,
			title: type === 'success' ? '✅ Success!' : type === 'error' ? '❌ Error!' : 'ℹ️ Info',
			message:
				type === 'success'
					? 'Operation completed successfully!'
					: type === 'error'
						? 'Something went wrong!'
						: 'Here is some information for you.'
		};

		notifications = [...notifications, notification];

		// Auto-remove after 4 seconds
		setTimeout(() => {
			removeNotification(notification.id);
		}, 4000);
	}

	function removeNotification(id: number) {
		notifications = notifications.filter((n) => n.id !== id);
	}

	function getTransition(type: string) {
		switch (type) {
			case 'success':
				return bounce;
			case 'error':
				return shake;
			case 'info':
				return slideRotate;
			default:
				return bounce;
		}
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-10 text-4xl">🎨 Custom CSS Transitions</h1>

	<div class="flex justify-center gap-4 mb-10">
		<button
			onclick={() => addNotification('success')}
			class="border-none px-7 py-3.5 rounded-lg text-base font-bold cursor-pointer transition-all bg-green-400 text-black hover:-translate-y-0.5 hover:shadow-lg"
		>
			Show Success
		</button>
		<button
			onclick={() => addNotification('error')}
			class="border-none px-7 py-3.5 rounded-lg text-base font-bold cursor-pointer transition-all bg-red-400 text-white hover:-translate-y-0.5 hover:shadow-lg"
		>
			Show Error
		</button>
		<button
			onclick={() => addNotification('info')}
			class="border-none px-7 py-3.5 rounded-lg text-base font-bold cursor-pointer transition-all bg-blue-400 text-black hover:-translate-y-0.5 hover:shadow-lg"
		>
			Show Info
		</button>
	</div>

	<div class="max-w-lg mx-auto flex flex-col gap-4">
		{#each notifications as notification (notification.id)}
			<div
				class="relative p-5 pr-12 rounded-xl border-2"
				class:bg-green-400/10={notification.type === 'success'}
				class:border-green-400={notification.type === 'success'}
				class:bg-red-400/10={notification.type === 'error'}
				class:border-red-400={notification.type === 'error'}
				class:bg-blue-400/10={notification.type === 'info'}
				class:border-blue-400={notification.type === 'info'}
				in:transition={getTransition(notification.type)}
			>
				<h3 class="m-0 mb-2 text-lg">{notification.title}</h3>
				<p class="m-0 text-gray-400 text-sm leading-relaxed">{notification.message}</p>
				<button
					onclick={() => removeNotification(notification.id)}
					class="absolute top-2.5 right-2.5 bg-transparent border-none text-gray-500 text-2xl cursor-pointer px-2.5 py-1.5 leading-none hover:text-white"
				>
					✕
				</button>
			</div>
		{/each}
	</div>
</div>
```

**Key Concepts:**

- **Return CSS string**: Control transform, opacity, and more
- **Easing functions**: Apply easing with `cubicOut()`, etc.
- **`t` parameter**: Goes from 0 to 1 during transition
- **Math for effects**: Use `Math.sin()`, etc., for complex animations
- **Real-world use**: Unique branding animations, attention-grabbing effects

---

## 7. Custom JS Transitions

### What are Custom JS Transitions?

Use the `tick` function to run JavaScript on every frame! Perfect for canvas animations, physics, or complex effects.

**Real-World Scenario:** You want to animate a progress circle or create a counter that smoothly increments with easing.

**What it does:** Creates JavaScript-driven transitions with frame-by-frame control.

```svelte
<script lang="ts">
	import { cubicOut } from 'svelte/easing';
	import type { TransitionConfig } from 'svelte/transition';

	// Custom counter animation
	function countUp(node: HTMLElement, { from = 0, to = 100, duration = 1000 }): TransitionConfig {
		return {
			duration,
			tick: (t: number) => {
				const eased = cubicOut(t);
				const value = Math.floor(from + (to - from) * eased);
				node.textContent = value.toString();
			}
		};
	}

	// Custom progress circle
	function progressCircle(node: HTMLElement, { duration = 1500 }): TransitionConfig {
		const svg = node.querySelector('svg');
		const circle = node.querySelector('circle.progress');

		if (!svg || !circle) return { duration: 0 };

		const radius = 90;
		const circumference = radius * 2 * Math.PI;

		return {
			duration,
			tick: (t: number) => {
				const eased = cubicOut(t);
				const offset = circumference - eased * circumference;
				circle.setAttribute('stroke-dashoffset', offset.toString());

				// Update text
				const textEl = node.querySelector('.percentage');
				if (textEl) {
					textEl.textContent = `${Math.floor(eased * 100)}%`;
				}
			}
		};
	}

	// Custom wave animation
	function wave(node: HTMLElement, { duration = 800 }): TransitionConfig {
		const letters = node.querySelectorAll('.letter');

		return {
			duration,
			tick: (t: number) => {
				letters.forEach((letter, i) => {
					const delay = i * 0.05;
					const letterT = Math.max(0, Math.min(1, (t - delay) / (1 - delay)));
					const eased = cubicOut(letterT);
					const translateY = (1 - eased) * -30;
					const opacity = eased;

					(letter as HTMLElement).style.transform = `translateY(${translateY}px)`;
					(letter as HTMLElement).style.opacity = opacity.toString();
				});
			}
		};
	}

	interface Stat {
		id: number;
		label: string;
		value: number;
		icon: string;
		color: string;
	}

	let showStats = $state(false);

	const stats: Stat[] = [
		{ id: 1, label: 'Users', value: 45823, icon: '👥', color: '#4a9eff' },
		{ id: 2, label: 'Revenue', value: 98750, icon: '💰', color: '#4ade80' },
		{ id: 3, label: 'Projects', value: 1234, icon: '📁', color: '#ffa500' }
	];
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-10 text-4xl">⚡ Custom JS Transitions</h1>

	<button
		onclick={() => (showStats = !showStats)}
		class="block mx-auto mb-10 bg-blue-500 text-black border-none px-8 py-4 rounded-lg text-lg font-bold cursor-pointer transition-all hover:bg-blue-400 hover:-translate-y-0.5"
	>
		{showStats ? 'Hide' : 'Show'} Dashboard
	</button>

	{#if showStats}
		<div class="max-w-5xl mx-auto">
			<!-- Animated heading -->
			<div class="text-center text-5xl font-extrabold mb-10 tracking-wider" in:wave>
				{#each 'Dashboard Stats' as char, i}
					<span class="inline-block" style="color: hsl({i * 20}, 70%, 60%)">
						{char === ' ' ? '\u00A0' : char}
					</span>
				{/each}
			</div>

			<!-- Progress circle -->
			<div class="relative w-52 h-52 mx-auto mb-16" in:progressCircle>
				<svg width="200" height="200">
					<circle cx="100" cy="100" r="90" fill="none" class="stroke-gray-800" stroke-width="12" />
					<circle
						class="progress stroke-blue-400"
						cx="100"
						cy="100"
						r="90"
						fill="none"
						stroke-width="12"
						stroke-linecap="round"
						stroke-dasharray="565.48"
						stroke-dashoffset="565.48"
						transform="rotate(-90 100 100)"
					/>
				</svg>
				<div class="absolute top-1/2 left-1/2 -translate-x-1/2 -translate-y-1/2 text-center">
					<div class="percentage text-4xl font-extrabold text-blue-400 mb-1">0%</div>
					<div class="label text-gray-500 text-sm font-semibold">Complete</div>
				</div>
			</div>

			<!-- Stat cards with counters -->
			<div class="grid grid-cols-[repeat(auto-fit,minmax(250px,1fr))] gap-6">
				{#each stats as stat, i}
					<div
						class="bg-gray-800 border-2 rounded-2xl p-8 text-center"
						style="border-color: {stat.color}"
					>
						<div class="text-5xl mb-4">{stat.icon}</div>
						<div
							class="text-4xl font-extrabold text-white mb-2 font-mono"
							in:countUp={{ from: 0, to: stat.value, duration: 1500 + i * 200 }}
						>
							0
						</div>
						<div class="text-gray-500 text-base font-semibold uppercase tracking-wide">
							{stat.label}
						</div>
					</div>
				{/each}
			</div>
		</div>
	{/if}
</div>
```

**Key Concepts:**

- **`tick` function**: Called every frame with `t` from 0 to 1
- **Direct DOM manipulation**: Update textContent, attributes, styles
- **Complex animations**: Counters, SVG progress, wave effects
- **Easing in tick**: Apply easing functions within tick
- **Real-world use**: Dashboards, data visualization, animated statistics

---

## 8. Custom Animations with animate:

### What are Custom Animations?

Combine `animate:` directive with custom animation functions for full control over FLIP animations!

**Real-World Scenario:** You're building a photo gallery where images shuffle into a masonry grid with custom easing and stagger effects.

**What it does:** Creates custom FLIP animations with unique timing and effects.

```svelte
<script lang="ts">
	import { quintOut, elasticOut } from 'svelte/easing';

	interface Photo {
		id: number;
		title: string;
		image: string;
		color: string;
		size: 'small' | 'medium' | 'large';
	}

	// Custom FLIP animation
	function customFlip(node: HTMLElement, { from, to }: any) {
		const dx = from.left - to.left;
		const dy = from.top - to.top;
		const dw = from.width / to.width;
		const dh = from.height / to.height;

		const duration = 600;
		const delay = Math.random() * 200; // Random stagger

		return {
			delay,
			duration,
			css: (t: number, u: number) => {
				const eased = elasticOut(t);
				const x = u * dx;
				const y = u * dy;
				const scaleX = 1 + u * (dw - 1);
				const scaleY = 1 + u * (dh - 1);

				return `
					transform: translate(${x}px, ${y}px) scale(${scaleX}, ${scaleY});
					transform-origin: top left;
				`;
			}
		};
	}

	let photos = $state<Photo[]>([
		{ id: 1, title: 'Mountain', image: '🏔️', color: '#4a9eff', size: 'large' },
		{ id: 2, title: 'Beach', image: '🏖️', color: '#4ade80', size: 'small' },
		{ id: 3, title: 'Forest', image: '🌲', color: '#4ade80', size: 'medium' },
		{ id: 4, title: 'Desert', image: '🏜️', color: '#ffa500', size: 'small' },
		{ id: 5, title: 'City', image: '🌆', color: '#4a9eff', size: 'medium' },
		{ id: 6, title: 'Lake', image: '🏞️', color: '#4a9eff', size: 'small' },
		{ id: 7, title: 'Sunset', image: '🌅', color: '#ff6b6b', size: 'large' },
		{ id: 8, title: 'Snow', image: '⛷️', color: '#4a9eff', size: 'medium' }
	]);

	let layout: 'grid' | 'masonry' | 'list' = $state('grid');

	function shuffle() {
		photos = photos.sort(() => Math.random() - 0.5);
	}

	function sortBySize() {
		photos = photos.sort((a, b) => {
			const sizeOrder = { small: 1, medium: 2, large: 3 };
			return sizeOrder[b.size] - sizeOrder[a.size];
		});
	}

	function getGridClass() {
		return layout === 'grid' ? 'photo-grid' : layout === 'masonry' ? 'photo-masonry' : 'photo-list';
	}

	function getSizeClass(size: string) {
		return layout === 'masonry' ? `size-${size}` : '';
	}
</script>

<div class="bg-gray-900 min-h-screen px-5 py-10 text-gray-200">
	<h1 class="text-center text-blue-400 m-0 mb-10 text-4xl">📸 Animated Photo Gallery</h1>

	<div class="max-w-5xl mx-auto mb-10 flex justify-between items-center flex-wrap gap-5">
		<div class="flex items-center gap-3">
			<span class="text-gray-500 font-semibold text-sm">Layout:</span>
			<button class="bg-gray-800 border-2 border-gray-700 text-gray-300 px-5 py-2.5 rounded-md font-semibold cursor-pointer transition-all hover:border-blue-500" class:!bg-blue-500={layout === 'grid'} class:!border-blue-500={layout === 'grid'} class:!text-black={layout === 'grid'} onclick={() => (layout = 'grid')}>
				Grid
			</button>
			<button
				class="bg-gray-800 border-2 border-gray-700 text-gray-300 px-5 py-2.5 rounded-md font-semibold cursor-pointer transition-all hover:border-blue-500"
				class:!bg-blue-500={layout === 'masonry'}
				class:!border-blue-500={layout === 'masonry'}
				class:!text-black={layout === 'masonry'}
				onclick={() => (layout = 'masonry')}
			>
				Masonry
			</button>
			<button class="bg-gray-800 border-2 border-gray-700 text-gray-300 px-5 py-2.5 rounded-md font-semibold cursor-pointer transition-all hover:border-blue-500" class:!bg-blue-500={layout === 'list'} class:!border-blue-500={layout === 'list'} class:!text-black={layout === 'list'} onclick={() => (layout = 'list')}>
				List
			</button>
		</div>

		<div class="flex items-center gap-3">
			<button onclick={shuffle} class="bg-green-500 border-none text-black px-6 py-3 rounded-md font-bold cursor-pointer transition-all hover:bg-green-400 hover:-translate-y-0.5"> 🔀 Shuffle </button>
			<button onclick={sortBySize} class="bg-green-500 border-none text-black px-6 py-3 rounded-md font-bold cursor-pointer transition-all hover:bg-green-400 hover:-translate-y-0.5"> 📏 Sort by Size </button>
		</div>
	</div>

	<div class={layout === 'grid' ? 'max-w-5xl mx-auto grid grid-cols-[repeat(auto-fill,minmax(200px,1fr))] gap-5' : layout === 'masonry' ? 'max-w-5xl mx-auto grid grid-cols-[repeat(auto-fill,minmax(150px,1fr))] auto-rows-fr gap-5' : 'max-w-xl mx-auto flex flex-col gap-4'}>
		{#each photos as photo (photo.id)}
			<div
				class="rounded-2xl p-8 flex items-center justify-center text-black cursor-pointer transition-transform min-h-48 hover:scale-105"
				class:flex-col={layout !== 'list'}
				class:flex-row={layout === 'list'}
				class:!justify-start={layout === 'list'}
				class:gap-6={layout === 'list'}
				class:!min-h-0={layout === 'list'}
				class:row-span-1={layout === 'masonry' && photo.size === 'small'}
				class:col-span-1={layout === 'masonry' && photo.size === 'small'}
				class:row-span-2={layout === 'masonry' && (photo.size === 'medium' || photo.size === 'large')}
				class:col-span-1={layout === 'masonry' && photo.size === 'medium'}
				class:col-span-2={layout === 'masonry' && photo.size === 'large'}
				style="background: {photo.color};"
				animate:customFlip
			>
				<div class="text-6xl mb-4" class:!mb-0={layout === 'list'} class:!text-5xl={layout === 'list'}>{photo.image}</div>
				<div class="text-2xl font-extrabold text-center" class:!text-xl={layout === 'list'}>{photo.title}</div>
			</div>
		{/each}
	</div>
</div>
```

**Key Concepts:**

- **Custom flip function**: Full control over FLIP animation
- **`from` and `to`**: Contain position/size before and after
- **Random delays**: Create staggered animations
- **Elastic easing**: Bouncy, fun animations
- **Real-world use**: Photo galleries, dashboard widgets, sortable grids

---

## 📝 Key Takeaways

✅ Transitions animate elements entering/leaving the DOM
✅ Built-in transitions (fade, fly, slide) cover most use cases
✅ Key blocks reset components and trigger transitions
✅ FLIP animations (`animate:flip`) smoothly animate position changes
✅ Crossfade creates paired transitions for moving elements
✅ Custom CSS transitions give full control over animation styles
✅ Custom JS transitions use `tick` for frame-by-frame control
✅ Custom animations combine with `animate:` for advanced FLIP effects

### 💡 Best Practices for Animations

**Choosing the Right Animation:**

```svelte
<!-- ✅ Use transitions for enter/exit -->
{#if show}
  <div transition:fade>Content</div>
{/if}

<!-- ✅ Use animate:flip for reordering -->
{#each items as item (item.id)}
  <div animate:flip>{ item.name }</div>
{/each}

<!-- ✅ Use key blocks to reset and transition -->
{#key selected}
  <Component data={selected} />
{/key}

<!-- ❌ Don't use both transition and animate -->
<div transition:fade animate:flip> <!-- Conflict! -->
```

**Performance Patterns:**

```typescript
// ✅ Animate transform and opacity (GPU-accelerated)
const transition = (node) => ({
  css: (t) => `
    transform: translateY(${(1 - t) * 100}px);
    opacity: ${t};
  `,
});

// ❌ Avoid animating layout properties
const badTransition = (node) => ({
  css: (t) => `
    height: ${t * 100}%;  // Forces reflow
    margin-top: ${(1 - t) * 50}px;  // Forces reflow
  `,
});
```

**Animation Organization:**

```
src/lib/
├── transitions/
│   ├── custom.ts           # Custom transitions
│   ├── presets.ts          # Transition presets
│   └── easing.ts           # Custom easing functions
└── animations/
    ├── flip.ts             # Custom FLIP animations
    └── spring.ts           # Spring physics
```

### ⚠️ Common Mistakes

1. **Animating expensive properties**:

```typescript
// ❌ Bad - animates width/height (causes reflow)
transition:slide

// ✅ Good - animates transform (GPU-accelerated)
transition:fly={{ y: -20 }}
```

2. **Forgetting `|global` modifier**:

```svelte
<!-- ❌ Bad - transition removed if parent transitions -->
{#if show}
	<div transition:fade>Content</div>
{/if}

<!-- ✅ Good - always runs regardless of parent -->
{#if show}
	<div transition:fade|global>Content</div>
{/if}
```

3. **Over-animating** - too much motion is distracting:

```svelte
<!-- ❌ Bad - every tiny change animates -->
{#key count}
	<div transition:fly={{ duration: 1000 }}>{count}</div>
{/key}

<!-- ✅ Good - only animate significant changes -->
{#if userChanged}
	<UserProfile transition:fade user={currentUser} />
{/if}
```

4. **Missing unique keys** for `animate:`:

```svelte
<!-- ❌ Bad - animations broken without unique keys -->
{#each items as item}
	<div animate:flip>{item.name}</div>
{/each}

<!-- ✅ Good - unique key enables proper animation -->
{#each items as item (item.id)}
	<div animate:flip>{item.name}</div>
{/each}
```

### ⚡ Performance Tips

- **GPU-accelerated properties**: Animate `transform` and `opacity` only
- **Use `will-change`** for complex animations: `style="will-change: transform"`
- **Reduce motion** for accessibility:

```typescript
const prefersReducedMotion = window.matchMedia(
  "(prefers-reduced-motion: reduce)"
).matches;
const duration = prefersReducedMotion ? 0 : 300;
```

- **Debounce re-orders**: Don't animate every keystroke in search filters
- **Lazy load animations**: Import heavy animation libraries only when needed

### 🎨 Easing Reference

```typescript
// Built-in easings
import { cubicOut, elasticOut, bounceOut } from 'svelte/easing';

// Usage
transition:fly={{ duration: 400, easing: cubicOut }}
animate:flip={{ duration: 400, easing: elasticOut }}

// Common patterns:
// - Enter: cubicOut (smooth entrance)
// - Exit: cubicIn (smooth exit)
// - Playful: elasticOut, bounceOut
// - Professional: quintOut, expoOut
```

### 🧪 Testing Animations

```typescript
import { render } from "@testing-library/svelte";
import { tick } from "svelte";

test("element transitions in", async () => {
  const { container } = render(Component);

  // Trigger show
  await tick();

  // Check element exists (animation started)
  expect(container.querySelector(".animated")).toBeTruthy();

  // For testing, disable animations:
  // Add `duration: 0` in test environment
});
```

---

## 🚀 End-of-Section Project: Animated Photo Gallery with Shopping Cart

### Project Overview

Build a **professional photo gallery e-commerce site** that demonstrates all Section 8 animation concepts: transitions, FLIP animations, crossfade, custom transitions, and key blocks. This combines smooth animations with real-world e-commerce functionality.

**What You'll Build:**

- **Masonry gallery layout** with stagger entrance transitions
- **Lightbox modal** with custom slide transitions
- **Shopping cart** with crossfade animations
- **Filter animations** using FLIP for smooth reordering
- **Add-to-cart animation** with custom JS transition
- **Checkout flow** with key block transitions between steps
- **Toast notifications** with slide and fade transitions

### 📁 Files to Create

**1. Photo Store** - `src/lib/stores/gallery.svelte.ts`

```typescript
export interface Photo {
  id: string;
  title: string;
  photographer: string;
  price: number;
  category: string;
  url: string;
  thumbnail: string;
  width: number;
  height: number;
}

export interface CartItem {
  photo: Photo;
  quantity: number;
  size: "small" | "medium" | "large";
  priceMultiplier: number;
}

class GalleryStore {
  photos = $state<Photo[]>([]);
  cart = $state<CartItem[]>([]);
  selectedCategory = $state<string>("all");
  lightboxIndex = $state<number | null>(null);

  // Derived values
  filteredPhotos = $derived(
    this.selectedCategory === "all"
      ? this.photos
      : this.photos.filter((p) => p.category === this.selectedCategory)
  );

  cartTotal = $derived(
    this.cart.reduce(
      (sum, item) =>
        sum + item.photo.price * item.priceMultiplier * item.quantity,
      0
    )
  );

  cartCount = $derived(this.cart.reduce((sum, item) => sum + item.quantity, 0));

  categories = $derived([
    "all",
    ...new Set(this.photos.map((p) => p.category)),
  ]);

  constructor() {
    // Load sample photos
    this.photos = [
      {
        id: "1",
        title: "Mountain Sunset",
        photographer: "John Doe",
        price: 29.99,
        category: "nature",
        url: "https://images.unsplash.com/photo-1506905925346-21bda4d32df4",
        thumbnail:
          "https://images.unsplash.com/photo-1506905925346-21bda4d32df4?w=400",
        width: 1600,
        height: 1200,
      },
      {
        id: "2",
        title: "City Lights",
        photographer: "Jane Smith",
        price: 39.99,
        category: "urban",
        url: "https://images.unsplash.com/photo-1514565131-fce0801e5785",
        thumbnail:
          "https://images.unsplash.com/photo-1514565131-fce0801e5785?w=400",
        width: 1600,
        height: 1200,
      },
      {
        id: "3",
        title: "Ocean Waves",
        photographer: "Mike Johnson",
        price: 34.99,
        category: "nature",
        url: "https://images.unsplash.com/photo-1505142468610-359e7d316be0",
        thumbnail:
          "https://images.unsplash.com/photo-1505142468610-359e7d316be0?w=400",
        width: 1600,
        height: 1200,
      },
      {
        id: "4",
        title: "Street Art",
        photographer: "Sarah Lee",
        price: 44.99,
        category: "urban",
        url: "https://images.unsplash.com/photo-1499781350541-7783f6c6a0c8",
        thumbnail:
          "https://images.unsplash.com/photo-1499781350541-7783f6c6a0c8?w=400",
        width: 1600,
        height: 1200,
      },
      {
        id: "5",
        title: "Forest Path",
        photographer: "Tom Wilson",
        price: 32.99,
        category: "nature",
        url: "https://images.unsplash.com/photo-1441974231531-c6227db76b6e",
        thumbnail:
          "https://images.unsplash.com/photo-1441974231531-c6227db76b6e?w=400",
        width: 1600,
        height: 1200,
      },
      {
        id: "6",
        title: "Neon Nights",
        photographer: "Emma Davis",
        price: 49.99,
        category: "urban",
        url: "https://images.unsplash.com/photo-1485847791529-83c593e11d2c",
        thumbnail:
          "https://images.unsplash.com/photo-1485847791529-83c593e11d2c?w=400",
        width: 1600,
        height: 1200,
      },
    ];
  }

  addToCart(photo: Photo, size: "small" | "medium" | "large") {
    const multipliers = { small: 1, medium: 1.5, large: 2 };
    const existing = this.cart.find(
      (item) => item.photo.id === photo.id && item.size === size
    );

    if (existing) {
      existing.quantity++;
    } else {
      this.cart.push({
        photo,
        quantity: 1,
        size,
        priceMultiplier: multipliers[size],
      });
    }
  }

  removeFromCart(photoId: string, size: string) {
    this.cart = this.cart.filter(
      (item) => !(item.photo.id === photoId && item.size === size)
    );
  }

  updateQuantity(photoId: string, size: string, quantity: number) {
    const item = this.cart.find(
      (i) => i.photo.id === photoId && i.size === size
    );
    if (item) {
      item.quantity = Math.max(1, quantity);
    }
  }

  clearCart() {
    this.cart = [];
  }

  openLightbox(index: number) {
    this.lightboxIndex = index;
  }

  closeLightbox() {
    this.lightboxIndex = null;
  }

  nextPhoto() {
    if (this.lightboxIndex !== null) {
      this.lightboxIndex =
        (this.lightboxIndex + 1) % this.filteredPhotos.length;
    }
  }

  prevPhoto() {
    if (this.lightboxIndex !== null) {
      this.lightboxIndex =
        (this.lightboxIndex - 1 + this.filteredPhotos.length) %
        this.filteredPhotos.length;
    }
  }
}

export const gallery = new GalleryStore();
```

**2. Custom Zoom Transition** - `src/lib/transitions/zoom.ts`

```typescript
import { cubicOut } from "svelte/easing";
import type { TransitionConfig } from "svelte/transition";

export function zoom(
  node: Element,
  { delay = 0, duration = 300, easing = cubicOut, start = 0.5 }
): TransitionConfig {
  const style = getComputedStyle(node);
  const opacity = +style.opacity;
  const transform = style.transform === "none" ? "" : style.transform;

  return {
    delay,
    duration,
    easing,
    css: (t, u) => `
			transform: ${transform} scale(${start + (1 - start) * t});
			opacity: ${t * opacity};
		`,
  };
}
```

**3. Add to Cart Animation** - `src/lib/transitions/flyToCart.ts`

```typescript
import type { TransitionConfig } from "svelte/transition";

export function flyToCart(
  node: Element,
  { targetSelector }: { targetSelector: string }
): TransitionConfig {
  const rect = node.getBoundingClientRect();
  const target = document.querySelector(targetSelector);

  if (!target) {
    return { duration: 0 };
  }

  const targetRect = target.getBoundingClientRect();
  const dx = targetRect.left - rect.left;
  const dy = targetRect.top - rect.top;

  return {
    duration: 600,
    css: (t, u) => {
      const eased = 1 - Math.pow(1 - t, 3); // easeOut cubic
      return `
				transform: translate(${dx * eased}px, ${dy * eased}px) scale(${1 - eased * 0.8});
				opacity: ${1 - eased};
			`;
    },
  };
}
```

**4. Gallery Grid Component** - `src/lib/components/PhotoGrid.svelte`

```svelte
<script lang="ts">
	import { fly, scale } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import { gallery } from '$lib/stores/gallery.svelte';
	import { zoom } from '$lib/transitions/zoom';

	let showAddToCart = $state<string | null>(null);
	let addingToCart = $state<string | null>(null);

	async function handleAddToCart(photoId: string, size: 'small' | 'medium' | 'large') {
		const photo = gallery.filteredPhotos.find(p => p.id === photoId);
		if (!photo) return;

		addingToCart = photoId;
		await new Promise(r => setTimeout(r, 100));

		gallery.addToCart(photo, size);

		await new Promise(r => setTimeout(r, 600));
		addingToCart = null;
		showAddToCart = null;
	}
</script>

<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
	{#each gallery.filteredPhotos as photo, i (photo.id)}
		<div
			in:fly={{ y: 50, delay: i * 50, duration: 400 }}
			animate:flip={{ duration: 400 }}
			class="group relative bg-gray-900 rounded-xl overflow-hidden border border-gray-800 hover:border-blue-500 transition-colors"
		>
			<button
				class="w-full"
				onclick={() => gallery.openLightbox(i)}
			>
				<img
					src={photo.thumbnail}
					alt={photo.title}
					class="w-full aspect-[4/3] object-cover group-hover:scale-105 transition-transform duration-300"
				/>
			</button>

			<div class="p-4">
				<h3 class="text-white font-semibold mb-1">{photo.title}</h3>
				<p class="text-gray-500 text-sm mb-3">by {photo.photographer}</p>

				<div class="flex items-center justify-between">
					<span class="text-blue-400 font-bold text-lg">${photo.price}</span>
					<button
						class="px-4 py-2 bg-blue-600 hover:bg-blue-500 text-white rounded-lg text-sm transition-colors"
						onclick={(e) => {
							e.stopPropagation();
							showAddToCart = photo.id;
						}}
					>
						Add to Cart
					</button>
				</div>
			</div>

			<!-- Size Selection Popup -->
			{#if showAddToCart === photo.id}
				<div
					in:scale={{ duration: 200 }}
					out:scale={{ duration: 200 }}
					class="absolute inset-0 bg-black/90 flex items-center justify-center p-6"
					onclick={(e) => e.stopPropagation()}
				>
					<div class="text-center">
						<h4 class="text-white font-bold mb-4">Select Size</h4>
						<div class="space-y-2">
							<button
								class="w-full px-6 py-3 bg-gray-800 hover:bg-gray-700 text-white rounded-lg transition-colors"
								onclick={() => handleAddToCart(photo.id, 'small')}
							>
								Small - ${photo.price}
							</button>
							<button
								class="w-full px-6 py-3 bg-gray-800 hover:bg-gray-700 text-white rounded-lg transition-colors"
								onclick={() => handleAddToCart(photo.id, 'medium')}
							>
								Medium - ${(photo.price * 1.5).toFixed(2)}
							</button>
							<button
								class="w-full px-6 py-3 bg-gray-800 hover:bg-gray-700 text-white rounded-lg transition-colors"
								onclick={() => handleAddToCart(photo.id, 'large')}
							>
								Large - ${(photo.price * 2).toFixed(2)}
							</button>
						</div>
						<button
							class="mt-4 text-gray-400 hover:text-white"
							onclick={() => (showAddToCart = null)}
						>
							Cancel
						</button>
					</div>
				</div>
			{/if}

			<!-- Flying to cart animation -->
			{#if addingToCart === photo.id}
				<div
					class="absolute inset-0 pointer-events-none"
					out:zoom={{ duration: 600 }}
				>
					<img
						src={photo.thumbnail}
						alt=""
						class="w-full h-full object-cover"
					/>
				</div>
			{/if}
		</div>
	{/each}
</div>
```

**5. Lightbox Modal** - `src/lib/components/Lightbox.svelte`

```svelte
<script lang="ts">
	import { fade, fly } from 'svelte/transition';
	import { gallery } from '$lib/stores/gallery.svelte';
	import { cubicOut } from 'svelte/easing';

	const currentPhoto = $derived(
		gallery.lightboxIndex !== null
			? gallery.filteredPhotos[gallery.lightboxIndex]
			: null
	);
</script>

{#if currentPhoto}
	<div
		class="fixed inset-0 z-50 flex items-center justify-center bg-black/95"
		transition:fade={{ duration: 200 }}
		onclick={gallery.closeLightbox}
	>
		<!-- Close button -->
		<button
			class="absolute top-4 right-4 text-white hover:text-gray-300 text-3xl z-10"
			onclick={gallery.closeLightbox}
		>
			✕
		</button>

		<!-- Navigation buttons -->
		<button
			class="absolute left-4 text-white hover:text-gray-300 text-4xl z-10"
			onclick={(e) => {
				e.stopPropagation();
				gallery.prevPhoto();
			}}
		>
			‹
		</button>
		<button
			class="absolute right-4 text-white hover:text-gray-300 text-4xl z-10"
			onclick={(e) => {
				e.stopPropagation();
				gallery.nextPhoto();
			}}
		>
			›
		</button>

		<!-- Photo with key block for smooth transitions -->
		{#key currentPhoto.id}
			<div
				class="max-w-5xl max-h-[90vh] mx-auto"
				in:fly={{ x: 100, duration: 300, easing: cubicOut }}
				out:fly={{ x: -100, duration: 300, easing: cubicOut }}
				onclick={(e) => e.stopPropagation()}
			>
				<img
					src={currentPhoto.url}
					alt={currentPhoto.title}
					class="max-w-full max-h-[80vh] object-contain rounded-lg"
				/>
				<div class="mt-4 text-center">
					<h2 class="text-white text-2xl font-bold">{currentPhoto.title}</h2>
					<p class="text-gray-400 mt-2">by {currentPhoto.photographer}</p>
				</div>
			</div>
		{/key}
	</div>
{/if}

<svelte:window
	onkeydown={(e) => {
		if (gallery.lightboxIndex === null) return;
		if (e.key === 'Escape') gallery.closeLightbox();
		if (e.key === 'ArrowLeft') gallery.prevPhoto();
		if (e.key === 'ArrowRight') gallery.nextPhoto();
	}}
/>
```

**6. Shopping Cart** - `src/lib/components/ShoppingCart.svelte`

```svelte
<script lang="ts">
	import { slide, fade } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import { gallery } from '$lib/stores/gallery.svelte';

	let isOpen = $state(false);
</script>

<!-- Cart Button -->
<button
	class="fixed top-6 right-6 bg-blue-600 hover:bg-blue-500 text-white px-6 py-3 rounded-full shadow-lg transition-colors flex items-center gap-2 z-40"
	onclick={() => (isOpen = !isOpen)}
>
	<span class="text-xl">🛒</span>
	<span class="font-bold">{gallery.cartCount}</span>
	{#if gallery.cartCount > 0}
		<span class="ml-2">${gallery.cartTotal.toFixed(2)}</span>
	{/if}
</button>

<!-- Cart Sidebar -->
{#if isOpen}
	<div
		class="fixed inset-0 z-50"
		transition:fade={{ duration: 200 }}
	>
		<!-- Backdrop -->
		<div
			class="absolute inset-0 bg-black/50"
			onclick={() => (isOpen = false)}
		></div>

		<!-- Sidebar -->
		<div
			class="absolute right-0 top-0 h-full w-full max-w-md bg-gray-900 shadow-2xl"
			transition:slide={{ duration: 300, axis: 'x' }}
		>
			<div class="flex flex-col h-full">
				<!-- Header -->
				<div class="p-6 border-b border-gray-800 flex items-center justify-between">
					<h2 class="text-xl font-bold text-white">Shopping Cart ({gallery.cartCount})</h2>
					<button
						class="text-gray-400 hover:text-white"
						onclick={() => (isOpen = false)}
					>
						✕
					</button>
				</div>

				<!-- Cart Items -->
				<div class="flex-1 overflow-y-auto p-6 space-y-4">
					{#if gallery.cart.length === 0}
						<div class="text-center py-12 text-gray-500">
							Your cart is empty
						</div>
					{:else}
						{#each gallery.cart as item (item.photo.id + item.size)}
							<div
								animate:flip={{ duration: 300 }}
								in:slide={{ duration: 200 }}
								out:slide={{ duration: 200 }}
								class="flex gap-4 bg-gray-800 rounded-lg p-4"
							>
								<img
									src={item.photo.thumbnail}
									alt={item.photo.title}
									class="w-20 h-20 object-cover rounded"
								/>
								<div class="flex-1 min-w-0">
									<h3 class="text-white font-semibold truncate">{item.photo.title}</h3>
									<p class="text-gray-400 text-sm">Size: {item.size}</p>
									<p class="text-blue-400 font-bold">
										${(item.photo.price * item.priceMultiplier).toFixed(2)}
									</p>
									<div class="flex items-center gap-2 mt-2">
										<button
											class="w-6 h-6 bg-gray-700 hover:bg-gray-600 rounded text-white"
											onclick={() =>
												gallery.updateQuantity(item.photo.id, item.size, item.quantity - 1)}
										>
											-
										</button>
										<span class="text-white">{item.quantity}</span>
										<button
											class="w-6 h-6 bg-gray-700 hover:bg-gray-600 rounded text-white"
											onclick={() =>
												gallery.updateQuantity(item.photo.id, item.size, item.quantity + 1)}
										>
											+
										</button>
									</div>
								</div>
								<button
									class="text-red-400 hover:text-red-300"
									onclick={() => gallery.removeFromCart(item.photo.id, item.size)}
								>
									🗑️
								</button>
							</div>
						{/each}
					{/if}
				</div>

				<!-- Footer -->
				{#if gallery.cart.length > 0}
					<div class="p-6 border-t border-gray-800 space-y-4">
						<div class="flex justify-between text-white text-xl font-bold">
							<span>Total:</span>
							<span>${gallery.cartTotal.toFixed(2)}</span>
						</div>
						<button
							class="w-full py-3 bg-blue-600 hover:bg-blue-500 text-white rounded-lg font-bold transition-colors"
							onclick={() => alert('Checkout coming soon!')}
						>
							Checkout
						</button>
						<button
							class="w-full py-2 text-gray-400 hover:text-white"
							onclick={() => gallery.clearCart()}
						>
							Clear Cart
						</button>
					</div>
				{/if}
			</div>
		</div>
	</div>
{/if}
```

**7. Main Gallery Page** - `src/routes/gallery/+page.svelte`

```svelte
<script lang="ts">
	import { fade, fly } from 'svelte/transition';
	import { flip } from 'svelte/animate';
	import { gallery } from '$lib/stores/gallery.svelte';
	import PhotoGrid from '$lib/components/PhotoGrid.svelte';
	import Lightbox from '$lib/components/Lightbox.svelte';
	import ShoppingCart from '$lib/components/ShoppingCart.svelte';
</script>

<div class="min-h-screen bg-gray-950 pb-20">
	<!-- Header -->
	<header class="bg-gray-900 border-b border-gray-800 sticky top-0 z-30">
		<div class="max-w-7xl mx-auto px-6 py-6">
			<h1 class="text-3xl font-bold text-white mb-6">📸 Photo Gallery</h1>

			<!-- Category Filters -->
			<div class="flex flex-wrap gap-2">
				{#each gallery.categories as category (category)}
					<button
						animate:flip={{ duration: 300 }}
						class="px-4 py-2 rounded-lg font-semibold transition-all {gallery.selectedCategory ===
						category
							? 'bg-blue-600 text-white scale-105'
							: 'bg-gray-800 text-gray-400 hover:bg-gray-700 hover:text-white'}"
						onclick={() => (gallery.selectedCategory = category)}
					>
						{category.charAt(0).toUpperCase() + category.slice(1)}
						{#if category !== 'all'}
							({gallery.photos.filter((p) => p.category === category).length})
						{:else}
							({gallery.photos.length})
						{/if}
					</button>
				{/each}
			</div>
		</div>
	</header>

	<!-- Gallery Grid -->
	<main class="max-w-7xl mx-auto px-6 py-8">
		<PhotoGrid />
	</main>

	<!-- Lightbox -->
	<Lightbox />

	<!-- Shopping Cart -->
	<ShoppingCart />
</div>
```

### 🎯 Project Checklist

- [ ] Staggered entrance animations with `in:fly`
- [ ] FLIP animations for category filter reordering
- [ ] Custom zoom transition for add-to-cart effect
- [ ] Key block transitions in lightbox for photo switching
- [ ] Slide transitions for cart sidebar
- [ ] Scale transitions for size selection popup
- [ ] Animate directive for smooth list reordering
- [ ] Fade transitions for modal overlays

### 💡 Extension Ideas

1. **Add image lazy loading** with intersection observer
2. **Implement favorites** with heart animation using crossfade
3. **Add photo comparison** slider with custom transition
4. **Implement infinite scroll** with loading animations
5. **Add zoom on hover** using transform animations
6. **Create image editor** with transition between edit modes
7. **Add download animation** with progress circle

---

## 🚀 Next Steps

After mastering animations and transitions, you're ready to:

- Build production-ready Svelte 5 applications
- Create polished, professional UIs with smooth interactions
- Implement complex animation patterns
- Combine all Svelte 5 features into complete projects

🎉 **Congratulations!** You've completed the Svelte 5 comprehensive study guide!
