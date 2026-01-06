# Tailwind CSS 4 + DaisyUI Reference Guide

## 📚 Complete Reference for All Classes Used in Svelte Study Guides

This guide explains every Tailwind CSS 4 class and DaisyUI component used throughout the study materials. Use this as a quick reference while learning.

### Learning Progression

The study materials follow a deliberate progression:

| Sections          | Styling Approach   | Purpose                                    |
| ----------------- | ------------------ | ------------------------------------------ |
| **1-8**           | Raw Tailwind CSS   | Learn utility-first CSS fundamentals       |
| **9-21**          | Tailwind + DaisyUI | Production patterns with component library |
| **Final Project** | Tailwind + DaisyUI | Real-world SaaS application                |

This progression teaches you to understand the fundamentals (Tailwind utilities) before leveraging productivity tools (DaisyUI components) - the same approach used by professional development teams.

---

## Table of Contents

1. [DaisyUI Components](#daisyui-components)
2. [Layout & Display](#layout--display)
3. [Spacing (Padding & Margin)](#spacing-padding--margin)
4. [Sizing (Width & Height)](#sizing-width--height)
5. [Typography](#typography)
6. [Colors & Backgrounds](#colors--backgrounds)
7. [Borders & Radius](#borders--radius)
8. [Flexbox](#flexbox)
9. [Grid](#grid)
10. [Effects & Filters](#effects--filters)
11. [Transitions & Animations](#transitions--animations)
12. [Interactivity](#interactivity)
13. [Positioning](#positioning)
14. [Responsive Design](#responsive-design)
15. [States (hover, focus, etc.)](#states-hover-focus-etc)

---

## DaisyUI Components

DaisyUI provides pre-built components on top of Tailwind CSS. These are CSS-only components (no JavaScript) that work perfectly with Svelte's reactivity.

### Buttons

```svelte
<!-- Basic buttons -->
<button class="btn">Default</button>
<button class="btn btn-primary">Primary</button>
<button class="btn btn-secondary">Secondary</button>
<button class="btn btn-accent">Accent</button>
<button class="btn btn-info">Info</button>
<button class="btn btn-success">Success</button>
<button class="btn btn-warning">Warning</button>
<button class="btn btn-error">Error</button>

<!-- Button sizes -->
<button class="btn btn-lg">Large</button>
<button class="btn btn-md">Medium (default)</button>
<button class="btn btn-sm">Small</button>
<button class="btn btn-xs">Extra Small</button>

<!-- Button variants -->
<button class="btn btn-outline btn-primary">Outline</button>
<button class="btn btn-ghost">Ghost</button>
<button class="btn btn-link">Link</button>
<button class="btn btn-active">Active</button>
<button class="btn btn-disabled">Disabled</button>

<!-- Button shapes -->
<button class="btn btn-circle">○</button>
<button class="btn btn-square">□</button>
<button class="btn btn-wide">Wide Button</button>
<button class="btn btn-block">Full Width</button>

<!-- Loading state (common in Svelte) -->
<button class="btn btn-primary" disabled={loading}>
  {#if loading}
    <span class="loading loading-spinner loading-sm"></span>
  {/if}
  {loading ? 'Loading...' : 'Submit'}
</button>
```

### Cards

```svelte
<!-- Basic card -->
<div class="card bg-base-100 shadow-xl">
  <figure>
    <img src="/image.jpg" alt="Cover" />
  </figure>
  <div class="card-body">
    <h2 class="card-title">Card Title</h2>
    <p>Card description goes here.</p>
    <div class="card-actions justify-end">
      <button class="btn btn-primary">Action</button>
    </div>
  </div>
</div>

<!-- Card variants -->
<div class="card card-compact"><!-- Smaller padding --></div>
<div class="card card-bordered"><!-- With border --></div>
<div class="card card-side"><!-- Horizontal layout --></div>
<div class="card image-full"><!-- Image as background --></div>

<!-- Glass effect card -->
<div class="card glass">
  <div class="card-body">
    <h2 class="card-title">Glass Card</h2>
  </div>
</div>
```

### Forms & Inputs

```svelte
<!-- Text input -->
<input type="text" class="input input-bordered w-full" placeholder="Type here" />

<!-- Input variants -->
<input class="input input-primary" />
<input class="input input-secondary" />
<input class="input input-accent" />
<input class="input input-info" />
<input class="input input-success" />
<input class="input input-warning" />
<input class="input input-error" />

<!-- Input sizes -->
<input class="input input-lg" />
<input class="input input-md" />
<input class="input input-sm" />
<input class="input input-xs" />

<!-- With label (form-control) -->
<div class="form-control w-full">
  <label class="label">
    <span class="label-text">Email</span>
    <span class="label-text-alt">Required</span>
  </label>
  <input type="email" class="input input-bordered" />
  <label class="label">
    <span class="label-text-alt text-error">Error message</span>
  </label>
</div>

<!-- Textarea -->
<textarea class="textarea textarea-bordered" placeholder="Message"></textarea>

<!-- Select -->
<select class="select select-bordered w-full">
  <option disabled selected>Pick one</option>
  <option>Option 1</option>
  <option>Option 2</option>
</select>

<!-- Checkbox -->
<div class="form-control">
  <label class="label cursor-pointer">
    <span class="label-text">Remember me</span>
    <input type="checkbox" class="checkbox checkbox-primary" />
  </label>
</div>

<!-- Toggle -->
<input type="checkbox" class="toggle toggle-primary" />

<!-- Radio -->
<input type="radio" name="radio-group" class="radio radio-primary" />

<!-- Range slider -->
<input type="range" class="range range-primary" min="0" max="100" />

<!-- Rating -->
<div class="rating">
  <input type="radio" name="rating" class="mask mask-star-2 bg-orange-400" />
  <input type="radio" name="rating" class="mask mask-star-2 bg-orange-400" checked />
  <input type="radio" name="rating" class="mask mask-star-2 bg-orange-400" />
</div>

<!-- File input -->
<input type="file" class="file-input file-input-bordered w-full" />
```

### Modals

```svelte
<!-- Modal with checkbox toggle -->
<label for="my-modal" class="btn">Open Modal</label>
<input type="checkbox" id="my-modal" class="modal-toggle" />
<div class="modal">
  <div class="modal-box">
    <h3 class="font-bold text-lg">Modal Title</h3>
    <p class="py-4">Modal content goes here.</p>
    <div class="modal-action">
      <label for="my-modal" class="btn">Close</label>
    </div>
  </div>
  <label class="modal-backdrop" for="my-modal"></label>
</div>

<!-- Modal with Svelte state (recommended) -->
<script>
  let isOpen = $state(false);
</script>

<button class="btn" onclick={() => isOpen = true}>Open Modal</button>

{#if isOpen}
  <div class="modal modal-open">
    <div class="modal-box">
      <button
        class="btn btn-sm btn-circle btn-ghost absolute right-2 top-2"
        onclick={() => isOpen = false}
      >✕</button>
      <h3 class="font-bold text-lg">Title</h3>
      <p class="py-4">Content</p>
      <div class="modal-action">
        <button class="btn" onclick={() => isOpen = false}>Close</button>
      </div>
    </div>
    <div class="modal-backdrop" onclick={() => isOpen = false}></div>
  </div>
{/if}
```

### Alerts

```svelte
<!-- Alert variants -->
<div class="alert">
  <span>Default alert message</span>
</div>
<div class="alert alert-info">
  <span>Info alert message</span>
</div>
<div class="alert alert-success">
  <span>Success alert message</span>
</div>
<div class="alert alert-warning">
  <span>Warning alert message</span>
</div>
<div class="alert alert-error">
  <span>Error alert message</span>
</div>

<!-- Alert with icon and action -->
<div class="alert alert-success">
  <svg xmlns="http://www.w3.org/2000/svg" class="stroke-current shrink-0 h-6 w-6" fill="none" viewBox="0 0 24 24">
    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9 12l2 2 4-4m6 2a9 9 0 11-18 0 9 9 0 0118 0z" />
  </svg>
  <span>Your purchase has been confirmed!</span>
  <button class="btn btn-sm">View</button>
</div>
```

### Badges

```svelte
<!-- Badge variants -->
<span class="badge">default</span>
<span class="badge badge-primary">primary</span>
<span class="badge badge-secondary">secondary</span>
<span class="badge badge-accent">accent</span>
<span class="badge badge-info">info</span>
<span class="badge badge-success">success</span>
<span class="badge badge-warning">warning</span>
<span class="badge badge-error">error</span>

<!-- Badge sizes -->
<span class="badge badge-lg">large</span>
<span class="badge badge-md">medium</span>
<span class="badge badge-sm">small</span>
<span class="badge badge-xs">tiny</span>

<!-- Outline badges -->
<span class="badge badge-outline badge-primary">outline</span>

<!-- Badge in button -->
<button class="btn gap-2">
  Inbox
  <span class="badge badge-sm">+99</span>
</button>
```

### Navigation

```svelte
<!-- Navbar -->
<div class="navbar bg-base-100">
  <div class="flex-1">
    <a class="btn btn-ghost text-xl">daisyUI</a>
  </div>
  <div class="flex-none">
    <ul class="menu menu-horizontal px-1">
      <li><a>Link</a></li>
      <li>
        <details>
          <summary>Parent</summary>
          <ul class="bg-base-100 rounded-t-none p-2">
            <li><a>Link 1</a></li>
            <li><a>Link 2</a></li>
          </ul>
        </details>
      </li>
    </ul>
  </div>
</div>

<!-- Menu -->
<ul class="menu bg-base-200 rounded-box w-56">
  <li><a>Item 1</a></li>
  <li><a>Item 2</a></li>
  <li><a class="active">Active Item</a></li>
  <li>
    <details open>
      <summary>Parent</summary>
      <ul>
        <li><a>Submenu 1</a></li>
        <li><a>Submenu 2</a></li>
      </ul>
    </details>
  </li>
</ul>

<!-- Tabs -->
<div class="tabs tabs-boxed">
  <a class="tab">Tab 1</a>
  <a class="tab tab-active">Tab 2</a>
  <a class="tab">Tab 3</a>
</div>

<!-- Breadcrumbs -->
<div class="breadcrumbs text-sm">
  <ul>
    <li><a>Home</a></li>
    <li><a>Documents</a></li>
    <li>Current Page</li>
  </ul>
</div>

<!-- Pagination -->
<div class="join">
  <button class="join-item btn">«</button>
  <button class="join-item btn">Page 1</button>
  <button class="join-item btn btn-active">Page 2</button>
  <button class="join-item btn">Page 3</button>
  <button class="join-item btn">»</button>
</div>
```

### Loading States

```svelte
<!-- Loading spinners -->
<span class="loading loading-spinner loading-xs"></span>
<span class="loading loading-spinner loading-sm"></span>
<span class="loading loading-spinner loading-md"></span>
<span class="loading loading-spinner loading-lg"></span>

<!-- Loading types -->
<span class="loading loading-dots"></span>
<span class="loading loading-ring"></span>
<span class="loading loading-ball"></span>
<span class="loading loading-bars"></span>
<span class="loading loading-infinity"></span>

<!-- Colored loading -->
<span class="loading loading-spinner text-primary"></span>
<span class="loading loading-spinner text-secondary"></span>
<span class="loading loading-spinner text-accent"></span>

<!-- Skeleton placeholders -->
<div class="flex flex-col gap-4 w-52">
  <div class="skeleton h-32 w-full"></div>
  <div class="skeleton h-4 w-28"></div>
  <div class="skeleton h-4 w-full"></div>
  <div class="skeleton h-4 w-full"></div>
</div>
```

### Tooltips & Dropdowns

```svelte
<!-- Tooltip -->
<div class="tooltip" data-tip="Hello!">
  <button class="btn">Hover me</button>
</div>

<!-- Tooltip positions -->
<div class="tooltip tooltip-top" data-tip="Top">...</div>
<div class="tooltip tooltip-bottom" data-tip="Bottom">...</div>
<div class="tooltip tooltip-left" data-tip="Left">...</div>
<div class="tooltip tooltip-right" data-tip="Right">...</div>

<!-- Dropdown -->
<div class="dropdown">
  <div tabindex="0" role="button" class="btn m-1">Click</div>
  <ul tabindex="0" class="dropdown-content menu bg-base-100 rounded-box z-[1] w-52 p-2 shadow">
    <li><a>Item 1</a></li>
    <li><a>Item 2</a></li>
  </ul>
</div>

<!-- Dropdown positions -->
<div class="dropdown dropdown-end">...</div>
<div class="dropdown dropdown-top">...</div>
<div class="dropdown dropdown-left">...</div>
<div class="dropdown dropdown-right">...</div>
```

### Tables

```svelte
<div class="overflow-x-auto">
  <table class="table">
    <!-- head -->
    <thead>
      <tr>
        <th></th>
        <th>Name</th>
        <th>Job</th>
        <th>Company</th>
      </tr>
    </thead>
    <tbody>
      <!-- row 1 -->
      <tr>
        <th>1</th>
        <td>John Doe</td>
        <td>Developer</td>
        <td>Acme Inc</td>
      </tr>
      <!-- row 2 with hover -->
      <tr class="hover">
        <th>2</th>
        <td>Jane Smith</td>
        <td>Designer</td>
        <td>Widget Co</td>
      </tr>
    </tbody>
  </table>
</div>

<!-- Table variants -->
<table class="table table-zebra"><!-- Striped rows --></table>
<table class="table table-pin-rows"><!-- Sticky header --></table>
<table class="table table-pin-cols"><!-- Sticky first column --></table>
<table class="table table-xs"><!-- Extra small --></table>
<table class="table table-sm"><!-- Small --></table>
<table class="table table-md"><!-- Medium --></table>
<table class="table table-lg"><!-- Large --></table>
```

### Toast Notifications

```svelte
<!-- Toast positioning (use with Svelte transitions) -->
<div class="toast toast-end">
  <div class="alert alert-success">
    <span>Message sent successfully.</span>
  </div>
</div>

<!-- Multiple toasts -->
<div class="toast toast-top toast-center">
  <div class="alert alert-info">
    <span>New mail arrived.</span>
  </div>
  <div class="alert alert-success">
    <span>Message sent.</span>
  </div>
</div>

<!-- Svelte toast pattern -->
<script>
  import { fly } from 'svelte/transition';

  let toasts = $state([]);

  function addToast(message, type = 'info') {
    const id = Date.now();
    toasts = [...toasts, { id, message, type }];
    setTimeout(() => removeToast(id), 3000);
  }

  function removeToast(id) {
    toasts = toasts.filter(t => t.id !== id);
  }
</script>

<div class="toast toast-end">
  {#each toasts as toast (toast.id)}
    <div
      class="alert alert-{toast.type}"
      transition:fly={{ x: 300, duration: 300 }}
    >
      <span>{toast.message}</span>
      <button class="btn btn-sm btn-ghost" onclick={() => removeToast(toast.id)}>✕</button>
    </div>
  {/each}
</div>
```

### Progress & Stats

```svelte
<!-- Progress bars -->
<progress class="progress w-56" value="0" max="100"></progress>
<progress class="progress progress-primary w-56" value="25" max="100"></progress>
<progress class="progress progress-secondary w-56" value="50" max="100"></progress>
<progress class="progress progress-accent w-56" value="75" max="100"></progress>
<progress class="progress progress-info w-56" value="100" max="100"></progress>

<!-- Radial progress -->
<div class="radial-progress" style="--value:70;">70%</div>
<div class="radial-progress text-primary" style="--value:70; --size:12rem; --thickness:2px;">70%</div>

<!-- Stats display -->
<div class="stats shadow">
  <div class="stat">
    <div class="stat-figure text-primary">
      <svg xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" class="inline-block w-8 h-8 stroke-current">
        <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M4.318 6.318a4.5 4.5 0 000 6.364L12 20.364l7.682-7.682a4.5 4.5 0 00-6.364-6.364L12 7.636l-1.318-1.318a4.5 4.5 0 00-6.364 0z"></path>
      </svg>
    </div>
    <div class="stat-title">Total Likes</div>
    <div class="stat-value text-primary">25.6K</div>
    <div class="stat-desc">21% more than last month</div>
  </div>
</div>
```

### Collapse & Accordion

```svelte
<!-- Single collapse -->
<div class="collapse collapse-arrow bg-base-200">
  <input type="checkbox" />
  <div class="collapse-title text-xl font-medium">Click to open</div>
  <div class="collapse-content">
    <p>Hidden content here</p>
  </div>
</div>

<!-- Accordion (radio for single open) -->
<div class="join join-vertical w-full">
  <div class="collapse collapse-arrow join-item border border-base-300">
    <input type="radio" name="accordion" checked />
    <div class="collapse-title text-xl font-medium">Item 1</div>
    <div class="collapse-content">
      <p>Content for item 1</p>
    </div>
  </div>
  <div class="collapse collapse-arrow join-item border border-base-300">
    <input type="radio" name="accordion" />
    <div class="collapse-title text-xl font-medium">Item 2</div>
    <div class="collapse-content">
      <p>Content for item 2</p>
    </div>
  </div>
</div>
```

### Avatars

```svelte
<!-- Basic avatar -->
<div class="avatar">
  <div class="w-24 rounded">
    <img src="/avatar.jpg" alt="User" />
  </div>
</div>

<!-- Avatar shapes -->
<div class="avatar">
  <div class="w-24 rounded-full"><!-- Circle --></div>
</div>
<div class="avatar">
  <div class="w-24 rounded-xl"><!-- Rounded square --></div>
</div>
<div class="avatar">
  <div class="w-24 mask mask-hexagon"><!-- Hexagon --></div>
</div>

<!-- Avatar with status -->
<div class="avatar online">
  <div class="w-24 rounded-full">
    <img src="/avatar.jpg" />
  </div>
</div>
<div class="avatar offline">...</div>

<!-- Avatar placeholder -->
<div class="avatar placeholder">
  <div class="bg-neutral text-neutral-content w-24 rounded-full">
    <span class="text-3xl">JD</span>
  </div>
</div>

<!-- Avatar group -->
<div class="avatar-group -space-x-6 rtl:space-x-reverse">
  <div class="avatar">
    <div class="w-12">
      <img src="/avatar1.jpg" />
    </div>
  </div>
  <div class="avatar">
    <div class="w-12">
      <img src="/avatar2.jpg" />
    </div>
  </div>
  <div class="avatar placeholder">
    <div class="bg-neutral text-neutral-content w-12">
      <span>+99</span>
    </div>
  </div>
</div>
```

### Themes

DaisyUI supports multiple themes. Configure in `tailwind.config.js`:

```javascript
// tailwind.config.js
module.exports = {
  plugins: [require("daisyui")],
  daisyui: {
    themes: [
      "light",
      "dark",
      "cupcake",
      "emerald",
      "corporate",
      "synthwave",
      "retro",
      "cyberpunk",
      "valentine",
      "halloween",
      "garden",
      "forest",
      "aqua",
      "lofi",
      "pastel",
      "fantasy",
      "wireframe",
      "black",
      "luxury",
      "dracula",
      "cmyk",
      "autumn",
      "business",
      "acid",
      "lemonade",
      "night",
      "coffee",
      "winter",
      "dim",
      "nord",
      "sunset",
    ],
  },
};
```

Apply theme in HTML:

```svelte
<!-- In app.html or layout -->
<html data-theme="dark">

<!-- Theme switcher in Svelte -->
<script>
  let theme = $state('dark');

  $effect(() => {
    document.documentElement.setAttribute('data-theme', theme);
  });
</script>

<select class="select select-bordered" bind:value={theme}>
  <option value="light">Light</option>
  <option value="dark">Dark</option>
  <option value="cupcake">Cupcake</option>
  <option value="synthwave">Synthwave</option>
</select>
```

### DaisyUI Best Practices

1. **Use semantic colors**: `btn-primary`, `btn-error` instead of `bg-blue-500`
2. **Responsive components**: DaisyUI components work with Tailwind breakpoints
3. **Combine with Tailwind**: Add custom Tailwind utilities alongside DaisyUI
4. **Keep state in Svelte**: Use `$state()` instead of DaisyUI's checkbox toggles when possible
5. **Accessibility**: DaisyUI components have good defaults, but verify ARIA labels
6. **Theme consistency**: Stick to one or two themes per project

```svelte
<!-- ✅ Good: Semantic colors + Svelte state -->
<button
  class="btn btn-primary"
  disabled={loading}
  onclick={handleSubmit}
>
  {loading ? 'Saving...' : 'Save'}
</button>

<!-- ❌ Avoid: Raw colors + DaisyUI toggle for state -->
<button class="bg-blue-500 text-white px-4 py-2">Save</button>
```

---

## Layout & Display

### Basic Display

- `block` - Display as block element (takes full width)
- `inline-block` - Display inline but can have width/height
- `inline` - Display inline (only takes content width)
- `flex` - Enable flexbox layout
- `inline-flex` - Flexbox that's inline
- `grid` - Enable CSS grid layout
- `hidden` - Hide element (display: none)

### Overflow

- `overflow-auto` - Add scrollbar when content overflows
- `overflow-hidden` - Hide overflowing content
- `overflow-x-auto` - Horizontal scroll only
- `overflow-y-auto` - Vertical scroll only
- `overflow-scroll` - Always show scrollbars

---

## Spacing (Padding & Margin)

### Padding (inside spacing)

- `p-0` to `p-96` - All sides padding (0 = 0, 4 = 1rem, 8 = 2rem, etc.)
- `px-4` - Horizontal padding (left + right)
- `py-4` - Vertical padding (top + bottom)
- `pt-4` - Padding top only
- `pr-4` - Padding right only
- `pb-4` - Padding bottom only
- `pl-4` - Padding left only

**Common Values:**

- `p-2` = 0.5rem (8px)
- `p-4` = 1rem (16px)
- `p-6` = 1.5rem (24px)
- `p-8` = 2rem (32px)
- `p-10` = 2.5rem (40px)
- `p-12` = 3rem (48px)

### Margin (outside spacing)

- `m-0` to `m-96` - All sides margin
- `mx-4` - Horizontal margin (left + right)
- `my-4` - Vertical margin (top + bottom)
- `mt-4` - Margin top
- `mr-4` - Margin right
- `mb-4` - Margin bottom
- `ml-4` - Margin left
- `mx-auto` - Center horizontally
- `-mt-4` - Negative margin top

### Gap (for flex/grid children)

- `gap-2` to `gap-96` - Gap between children
- `gap-x-4` - Horizontal gap only
- `gap-y-4` - Vertical gap only

---

## Sizing (Width & Height)

### Width

- `w-full` - 100% width
- `w-screen` - 100vw (viewport width)
- `w-auto` - Auto width
- `w-fit` - Fit content width
- `w-1/2` - 50% width
- `w-1/3` - 33.33% width
- `w-2/3` - 66.66% width
- `w-1/4` - 25% width
- `w-3/4` - 75% width
- `w-64` - 16rem (256px)
- `w-96` - 24rem (384px)

**Specific Widths:**

- `w-4` = 1rem (16px)
- `w-8` = 2rem (32px)
- `w-20` = 5rem (80px)
- `w-32` = 8rem (128px)

### Max Width

- `max-w-xs` - 20rem (320px)
- `max-w-sm` - 24rem (384px)
- `max-w-md` - 28rem (448px)
- `max-w-lg` - 32rem (512px)
- `max-w-xl` - 36rem (576px)
- `max-w-2xl` - 42rem (672px)
- `max-w-4xl` - 56rem (896px)
- `max-w-6xl` - 72rem (1152px)
- `max-w-7xl` - 80rem (1280px)

### Min Width

- `min-w-0` - No minimum width
- `min-w-full` - 100% minimum width
- `min-w-fit` - Fit content

### Height

- `h-full` - 100% height
- `h-screen` - 100vh (viewport height)
- `h-auto` - Auto height
- `h-fit` - Fit content
- `h-64` - 16rem (256px)

**Specific Heights:**

- `h-4` = 1rem (16px)
- `h-5` = 1.25rem (20px)
- `h-8` = 2rem (32px)
- `h-20` = 5rem (80px)

### Max Height

- `max-h-screen` - 100vh max
- `max-h-96` - 24rem (384px)
- `max-h-[200px]` - Custom 200px

### Min Height

- `min-h-screen` - 100vh minimum
- `min-h-full` - 100% minimum
- `min-h-[200px]` - Custom 200px

---

## Typography

### Font Family

- `font-sans` - Sans-serif font
- `font-serif` - Serif font
- `font-mono` - Monospace font

### Font Size

- `text-xs` - 0.75rem (12px)
- `text-sm` - 0.875rem (14px)
- `text-base` - 1rem (16px)
- `text-lg` - 1.125rem (18px)
- `text-xl` - 1.25rem (20px)
- `text-2xl` - 1.5rem (24px)
- `text-3xl` - 1.875rem (30px)
- `text-4xl` - 2.25rem (36px)
- `text-5xl` - 3rem (48px)
- `text-6xl` - 3.75rem (60px)

### Font Weight

- `font-thin` - 100
- `font-light` - 300
- `font-normal` - 400
- `font-medium` - 500
- `font-semibold` - 600
- `font-bold` - 700
- `font-extrabold` - 800
- `font-black` - 900

### Line Height

- `leading-none` - 1
- `leading-tight` - 1.25
- `leading-snug` - 1.375
- `leading-normal` - 1.5
- `leading-relaxed` - 1.625
- `leading-loose` - 2

### Text Alignment

- `text-left` - Left align
- `text-center` - Center align
- `text-right` - Right align
- `text-justify` - Justify

### Text Decoration

- `underline` - Underline text
- `line-through` - Strike through
- `no-underline` - Remove underline

### Text Transform

- `uppercase` - UPPERCASE
- `lowercase` - lowercase
- `capitalize` - Capitalize First Letter
- `normal-case` - No transformation

### Text Overflow

- `truncate` - Truncate with ellipsis (...)
- `text-ellipsis` - Add ellipsis
- `text-clip` - Clip text

### Other Text Properties

- `italic` - Italic text
- `not-italic` - Remove italic
- `tracking-tight` - Tighter letter spacing
- `tracking-normal` - Normal letter spacing
- `tracking-wide` - Wider letter spacing

---

## Colors & Backgrounds

### Text Colors

Custom color palette used in examples:

- `text-white` - White text (#fff)
- `text-black` - Black text (#000)
- `text-blue-400` - Light blue (#4a9eff)
- `text-green-400` - Green (#4ade80)
- `text-red-400` - Red (#ff6b6b)
- `text-orange-500` - Orange (#ffa500)
- `text-gray-400` - Light gray (#ccc)
- `text-gray-500` - Medium gray (#888)
- `text-gray-600` - Dark gray (#666)

**Opacity Modifiers:**

- `text-white/50` - 50% opacity
- `text-blue-400/80` - 80% opacity

### Background Colors

- `bg-[#1a1a1a]` - Custom dark background
- `bg-[#2a2a2a]` - Card background
- `bg-[#3a3a3a]` - Border/accent background
- `bg-blue-400` - Blue background (#4a9eff)
- `bg-green-400` - Green background (#4ade80)
- `bg-red-400` - Red background (#ff6b6b)
- `bg-transparent` - Transparent background
- `bg-white` - White background
- `bg-black` - Black background

**Background Opacity:**

- `bg-red-400/10` - Red with 10% opacity
- `bg-blue-400/20` - Blue with 20% opacity

### Gradients

- `bg-gradient-to-r` - Left to right gradient
- `bg-gradient-to-b` - Top to bottom gradient
- `bg-gradient-to-tr` - Top-left to bottom-right
- `from-blue-400` - Gradient start color
- `to-purple-500` - Gradient end color
- `via-pink-500` - Gradient middle color

---

## Borders & Radius

### Border Width

- `border` - 1px border
- `border-0` - No border
- `border-2` - 2px border
- `border-4` - 4px border
- `border-t` - Top border only
- `border-r` - Right border only
- `border-b` - Bottom border only
- `border-l` - Left border only

### Border Color

- `border-[#3a3a3a]` - Custom border color
- `border-blue-400` - Blue border
- `border-transparent` - Transparent border

### Border Radius

- `rounded` - 0.25rem (4px)
- `rounded-sm` - 0.125rem (2px)
- `rounded-md` - 0.375rem (6px)
- `rounded-lg` - 0.5rem (8px)
- `rounded-xl` - 0.75rem (12px)
- `rounded-2xl` - 1rem (16px)
- `rounded-3xl` - 1.5rem (24px)
- `rounded-full` - 9999px (perfect circle/pill)
- `rounded-none` - No border radius
- `rounded-t-lg` - Top corners only
- `rounded-b-lg` - Bottom corners only

### Outline

- `outline-none` - Remove outline (use carefully for accessibility)
- `outline` - Add outline
- `outline-2` - 2px outline

---

## Flexbox

### Enable Flexbox

- `flex` - Enable flexbox

### Flex Direction

- `flex-row` - Horizontal (default)
- `flex-col` - Vertical
- `flex-row-reverse` - Horizontal reversed
- `flex-col-reverse` - Vertical reversed

### Flex Wrap

- `flex-wrap` - Allow wrapping
- `flex-nowrap` - No wrapping (default)

### Justify Content (main axis)

- `justify-start` - Align to start
- `justify-center` - Center items
- `justify-end` - Align to end
- `justify-between` - Space between items
- `justify-around` - Space around items
- `justify-evenly` - Even spacing

### Align Items (cross axis)

- `items-start` - Align to start
- `items-center` - Center items
- `items-end` - Align to end
- `items-stretch` - Stretch to fill
- `items-baseline` - Align baselines

### Align Self (individual item)

- `self-start` - Align self to start
- `self-center` - Center self
- `self-end` - Align self to end
- `self-stretch` - Stretch self

### Flex Grow/Shrink

- `flex-1` - Grow and shrink equally (flex: 1 1 0%)
- `flex-auto` - Grow and shrink based on content
- `flex-initial` - Shrink but don't grow
- `flex-none` - Don't grow or shrink
- `grow` - Allow growing
- `shrink` - Allow shrinking
- `shrink-0` - Don't shrink

---

## Grid

### Enable Grid

- `grid` - Enable CSS grid

### Grid Template Columns

- `grid-cols-1` - 1 column
- `grid-cols-2` - 2 columns
- `grid-cols-3` - 3 columns
- `grid-cols-4` - 4 columns
- `grid-cols-12` - 12 columns (common for layouts)
- `grid-cols-[repeat(auto-fit,minmax(300px,1fr))]` - Responsive auto-fit

### Grid Template Rows

- `grid-rows-1` - 1 row
- `grid-rows-2` - 2 rows
- `grid-rows-3` - 3 rows

### Grid Column Span

- `col-span-1` - Span 1 column
- `col-span-2` - Span 2 columns
- `col-span-full` - Span all columns

### Grid Row Span

- `row-span-1` - Span 1 row
- `row-span-2` - Span 2 rows

### Gap (spacing between cells)

- `gap-2` - 0.5rem gap
- `gap-4` - 1rem gap
- `gap-6` - 1.5rem gap
- `gap-x-4` - Horizontal gap
- `gap-y-4` - Vertical gap

---

## Effects & Filters

### Box Shadow

- `shadow-sm` - Small shadow
- `shadow` - Default shadow
- `shadow-md` - Medium shadow
- `shadow-lg` - Large shadow
- `shadow-xl` - Extra large shadow
- `shadow-2xl` - 2xl shadow
- `shadow-none` - No shadow
- `shadow-[0_4px_12px_rgba(0,0,0,0.3)]` - Custom shadow

### Opacity

- `opacity-0` - Fully transparent (0%)
- `opacity-50` - 50% opacity
- `opacity-75` - 75% opacity
- `opacity-100` - Fully opaque (100%)

### Mix Blend Mode

- `mix-blend-normal` - Normal blending
- `mix-blend-multiply` - Multiply blending
- `mix-blend-screen` - Screen blending

---

## Transitions & Animations

### Transition Property

- `transition` - All properties
- `transition-colors` - Colors only
- `transition-transform` - Transform only
- `transition-opacity` - Opacity only
- `transition-all` - All properties (shorthand)

### Transition Duration

- `duration-75` - 75ms
- `duration-100` - 100ms
- `duration-150` - 150ms
- `duration-200` - 200ms
- `duration-300` - 300ms
- `duration-500` - 500ms

### Transition Timing

- `ease-linear` - Linear timing
- `ease-in` - Ease in
- `ease-out` - Ease out
- `ease-in-out` - Ease in and out

### Transform

- `scale-0` - Scale to 0%
- `scale-100` - Normal scale (100%)
- `scale-105` - Scale to 105%
- `scale-110` - Scale to 110%
- `rotate-0` - No rotation
- `rotate-45` - 45 degrees
- `rotate-90` - 90 degrees
- `rotate-180` - 180 degrees
- `translate-x-1` - Move right 0.25rem
- `translate-y-1` - Move down 0.25rem
- `-translate-y-0.5` - Move up 0.125rem
- `-translate-y-2` - Move up 0.5rem

### Animation

- `animate-spin` - Spin animation (loading spinners)
- `animate-pulse` - Pulse animation
- `animate-bounce` - Bounce animation

---

## Interactivity

### Cursor

- `cursor-pointer` - Pointer cursor (hand)
- `cursor-default` - Default cursor
- `cursor-not-allowed` - Not allowed cursor
- `cursor-move` - Move cursor

### User Select

- `select-none` - Disable text selection
- `select-text` - Enable text selection
- `select-all` - Select all on click

### Pointer Events

- `pointer-events-none` - Ignore pointer events
- `pointer-events-auto` - Enable pointer events

### Resize

- `resize` - Allow resizing
- `resize-none` - Disable resizing

---

## Positioning

### Position

- `static` - Static (default)
- `relative` - Relative positioning
- `absolute` - Absolute positioning
- `fixed` - Fixed positioning
- `sticky` - Sticky positioning

### Position Values

- `top-0` - Top: 0
- `right-0` - Right: 0
- `bottom-0` - Bottom: 0
- `left-0` - Left: 0
- `inset-0` - All sides: 0
- `top-4` - Top: 1rem
- `top-1/2` - Top: 50%
- `-top-4` - Negative top

### Z-Index

- `z-0` - z-index: 0
- `z-10` - z-index: 10
- `z-20` - z-index: 20
- `z-50` - z-index: 50
- `z-[1000]` - Custom z-index: 1000

---

## Responsive Design

### Breakpoint Prefixes

Apply classes only at specific screen sizes:

- `sm:` - Small screens (640px+)
- `md:` - Medium screens (768px+)
- `lg:` - Large screens (1024px+)
- `xl:` - Extra large (1280px+)
- `2xl:` - 2X large (1536px+)

**Examples:**

```html
<!-- Grid: 1 column mobile, 2 on medium, 3 on large -->
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3">
  <!-- Hide on mobile, show on medium+ -->
  <div class="hidden md:block">
    <!-- Full width mobile, 50% on large -->
    <div class="w-full lg:w-1/2"></div>
  </div>
</div>
```

---

## States (hover, focus, etc.)

### Hover State

Apply styles on hover:

- `hover:bg-blue-500` - Background on hover
- `hover:text-white` - Text color on hover
- `hover:scale-105` - Scale on hover
- `hover:shadow-lg` - Shadow on hover

### Focus State

Apply styles when focused:

- `focus:outline-none` - Remove outline on focus
- `focus:ring-2` - Add ring on focus
- `focus:border-blue-400` - Border color on focus
- `focus:bg-white` - Background on focus

### Active State

Apply when clicked:

- `active:scale-95` - Scale down when clicked
- `active:bg-blue-600` - Darker background

### Disabled State

Apply when disabled:

- `disabled:opacity-50` - Fade when disabled
- `disabled:cursor-not-allowed` - Cursor when disabled

### Group Hover

Parent-child hover interaction:

```html
<div class="group">
  <p class="group-hover:text-blue-400">Appears blue on parent hover</p>
</div>
```

### Focus Within

Apply when child is focused:

- `focus-within:border-blue-400`

### Other States

- `first:` - First child
- `last:` - Last child
- `even:` - Even children
- `odd:` - Odd children
- `checked:` - Checked inputs
- `placeholder:` - Placeholder styles

---

## Common Patterns Used in Study Guides

### Card Component

```html
<div class="bg-[#2a2a2a] border-2 border-[#3a3a3a] rounded-xl p-6">
  <!-- Card content -->
</div>
```

### Button (Primary)

```html
<button
  class="bg-blue-400 text-black font-bold px-6 py-3 rounded-lg 
               hover:bg-blue-300 transition-all duration-200 
               hover:-translate-y-0.5 cursor-pointer"
>
  Click me
</button>
```

### Input Field

```html
<input
  class="w-full bg-[#1a1a1a] border-2 border-[#3a3a3a] 
              text-white px-4 py-3 rounded-lg text-base 
              focus:outline-none focus:border-blue-400"
/>
```

### Flex Container (centered)

```html
<div class="flex items-center justify-center gap-4">
  <!-- Items -->
</div>
```

### Grid Layout (responsive)

```html
<div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
  <!-- Grid items -->
</div>
```

### Container (centered with max-width)

```html
<div class="max-w-6xl mx-auto px-5">
  <!-- Content -->
</div>
```

---

## Custom Color Palette

Our dark theme uses these custom colors:

```css
/* In your global CSS or Tailwind config */
--bg-primary: #1a1a1a; /* Main background */
--bg-secondary: #2a2a2a; /* Cards, panels */
--bg-tertiary: #3a3a3a; /* Borders, accents */
--color-blue: #4a9eff; /* Primary blue */
--color-green: #4ade80; /* Success green */
--color-red: #ff6b6b; /* Error red */
--color-orange: #ffa500; /* Warning orange */
--text-muted: #888; /* Muted text */
--text-secondary: #ccc; /* Secondary text */
--text-primary: #fff; /* Primary text */
```

Use in Tailwind:

- `bg-[#1a1a1a]` - Bracket notation for custom colors
- `text-[#4a9eff]` - Custom text colors
- `border-[#3a3a3a]` - Custom border colors

---

## Arbitrary Values

Tailwind 4 allows arbitrary values with brackets:

- `w-[350px]` - Exact 350px width
- `h-[calc(100vh-80px)]` - Calculated height
- `text-[#1a1a1a]` - Custom color
- `top-[13px]` - Exact positioning
- `grid-cols-[200px_1fr_1fr]` - Custom grid template
- `shadow-[0_4px_12px_rgba(0,0,0,0.3)]` - Custom shadow

---

## Real-World Common Patterns

### Navigation Bar

```html
<!-- Sticky top navigation -->
<nav
  class="fixed top-0 left-0 right-0 bg-[#1a1a1a] border-b border-[#3a3a3a] 
            z-50 backdrop-blur-lg bg-opacity-90"
>
  <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex items-center justify-between h-16">
      <div class="flex items-center gap-8">
        <h1 class="text-2xl font-bold text-[#4a9eff]">Logo</h1>
        <div class="hidden md:flex gap-6">
          <a href="#" class="text-[#ccc] hover:text-white transition-colors"
            >Home</a
          >
          <a href="#" class="text-[#ccc] hover:text-white transition-colors"
            >About</a
          >
          <a href="#" class="text-[#ccc] hover:text-white transition-colors"
            >Contact</a
          >
        </div>
      </div>
      <button class="md:hidden text-white">☰</button>
    </div>
  </div>
</nav>
```

### Form with Validation

```html
<!-- Login form with error states -->
<form class="w-full max-w-md mx-auto space-y-4">
  <div>
    <label class="block text-sm font-medium text-[#ccc] mb-2">Email</label>
    <input
      type="email"
      class="w-full bg-[#1a1a1a] border-2 border-[#3a3a3a] text-white 
             px-4 py-3 rounded-lg focus:outline-none focus:border-[#4a9eff]
             placeholder:text-[#888]"
      class:border-[#ff6b6b]="{hasError}"
      placeholder="you@example.com"
    />
    {#if hasError}
    <p class="text-[#ff6b6b] text-sm mt-1">Invalid email address</p>
    {/if}
  </div>

  <button
    type="submit"
    class="w-full bg-[#4a9eff] text-black font-semibold py-3 rounded-lg
           hover:bg-[#6ab0ff] transition-all duration-200
           disabled:opacity-50 disabled:cursor-not-allowed"
  >
    Sign In
  </button>
</form>
```

### Loading Spinner

```html
<!-- Centered loading spinner -->
<div class="flex items-center justify-center min-h-screen">
  <div class="relative">
    <div
      class="w-16 h-16 border-4 border-[#3a3a3a] border-t-[#4a9eff] 
                rounded-full animate-spin"
    ></div>
    <p class="text-[#888] text-sm mt-4 text-center">Loading...</p>
  </div>
</div>

<!-- Inline button spinner -->
<button class="flex items-center gap-2 px-6 py-3 bg-[#4a9eff] rounded-lg">
  <div
    class="w-5 h-5 border-2 border-black border-t-transparent 
              rounded-full animate-spin"
  ></div>
  <span>Processing...</span>
</button>
```

### Modal/Dialog

```html
<!-- Full-screen modal overlay -->
<div
  class="fixed inset-0 bg-black/80 backdrop-blur-sm z-50 
            flex items-center justify-center p-4"
>
  <div
    class="bg-[#2a2a2a] rounded-xl max-w-md w-full p-6 
              shadow-2xl animate-[slideIn_0.3s_ease]"
  >
    <div class="flex items-center justify-between mb-4">
      <h2 class="text-xl font-bold text-white">Confirm Action</h2>
      <button class="text-[#888] hover:text-white transition-colors">✕</button>
    </div>

    <p class="text-[#ccc] mb-6">Are you sure you want to proceed?</p>

    <div class="flex gap-3">
      <button
        class="flex-1 bg-[#3a3a3a] text-white py-2 rounded-lg
                     hover:bg-[#4a4a4a] transition-colors"
      >
        Cancel
      </button>
      <button
        class="flex-1 bg-[#ff6b6b] text-white py-2 rounded-lg
                     hover:bg-[#ff8787] transition-colors"
      >
        Confirm
      </button>
    </div>
  </div>
</div>
```

### Toast Notification

```html
<!-- Top-right toast notification -->
<div
  class="fixed top-4 right-4 z-50 max-w-sm
            bg-[#2a2a2a] border-2 border-[#4ade80] rounded-lg p-4
            shadow-lg animate-[slideIn_0.3s_ease]"
>
  <div class="flex items-start gap-3">
    <span class="text-2xl">✓</span>
    <div class="flex-1">
      <h3 class="font-semibold text-white mb-1">Success!</h3>
      <p class="text-sm text-[#ccc]">Your changes have been saved.</p>
    </div>
    <button class="text-[#888] hover:text-white">✕</button>
  </div>
</div>
```

### Data Table

```html
<!-- Responsive data table -->
<div class="overflow-x-auto rounded-lg border border-[#3a3a3a]">
  <table class="w-full">
    <thead class="bg-[#2a2a2a] border-b border-[#3a3a3a]">
      <tr>
        <th class="text-left py-3 px-4 text-[#ccc] font-semibold">Name</th>
        <th class="text-left py-3 px-4 text-[#ccc] font-semibold">Email</th>
        <th class="text-left py-3 px-4 text-[#ccc] font-semibold">Status</th>
        <th class="text-right py-3 px-4 text-[#ccc] font-semibold">Actions</th>
      </tr>
    </thead>
    <tbody class="bg-[#1a1a1a]">
      <tr
        class="border-b border-[#3a3a3a] hover:bg-[#2a2a2a] transition-colors"
      >
        <td class="py-3 px-4 text-white">John Doe</td>
        <td class="py-3 px-4 text-[#888]">john@example.com</td>
        <td class="py-3 px-4">
          <span
            class="bg-[#4ade80] text-black px-2 py-1 rounded text-xs font-semibold"
          >
            Active
          </span>
        </td>
        <td class="py-3 px-4 text-right">
          <button class="text-[#4a9eff] hover:underline">Edit</button>
        </td>
      </tr>
    </tbody>
  </table>
</div>
```

### Dropdown Menu

```html
<!-- Dropdown with hover/focus states -->
<div class="relative inline-block">
  <button
    class="flex items-center gap-2 px-4 py-2 bg-[#2a2a2a] 
                 border border-[#3a3a3a] rounded-lg hover:bg-[#3a3a3a]"
  >
    <span>Options</span>
    <span class="text-xs">▼</span>
  </button>

  <!-- Dropdown content -->
  <div
    class="absolute right-0 mt-2 w-48 bg-[#2a2a2a] border border-[#3a3a3a] 
              rounded-lg shadow-xl z-10 overflow-hidden"
  >
    <a
      href="#"
      class="block px-4 py-2 text-[#ccc] hover:bg-[#3a3a3a] 
                       hover:text-white transition-colors"
    >
      Edit Profile
    </a>
    <a
      href="#"
      class="block px-4 py-2 text-[#ccc] hover:bg-[#3a3a3a] 
                       hover:text-white transition-colors"
    >
      Settings
    </a>
    <hr class="border-[#3a3a3a]" />
    <a
      href="#"
      class="block px-4 py-2 text-[#ff6b6b] hover:bg-[#3a3a3a] 
                       transition-colors"
    >
      Sign Out
    </a>
  </div>
</div>
```

### Tabs

```html
<!-- Horizontal tabs -->
<div class="border-b border-[#3a3a3a]">
  <div class="flex gap-1">
    <button
      class="px-4 py-2 text-[#4a9eff] border-b-2 border-[#4a9eff] 
                   font-semibold"
    >
      Overview
    </button>
    <button class="px-4 py-2 text-[#888] hover:text-white transition-colors">
      Analytics
    </button>
    <button class="px-4 py-2 text-[#888] hover:text-white transition-colors">
      Settings
    </button>
  </div>
</div>
```

### Badges & Status Indicators

```html
<!-- Status badges -->
<span class="bg-[#4ade80] text-black px-2 py-1 rounded text-xs font-semibold">
  Success
</span>

<span class="bg-[#ffa500] text-black px-2 py-1 rounded text-xs font-semibold">
  Pending
</span>

<span class="bg-[#ff6b6b] text-white px-2 py-1 rounded text-xs font-semibold">
  Error
</span>

<span class="bg-[#4a9eff] text-black px-2 py-1 rounded text-xs font-semibold">
  Info
</span>

<!-- Dot indicator -->
<div class="flex items-center gap-2">
  <span class="w-2 h-2 bg-[#4ade80] rounded-full"></span>
  <span class="text-[#ccc]">Online</span>
</div>
```

### Avatar Component

```html
<!-- Single avatar -->
<img
  src="/avatar.jpg"
  alt="User"
  class="w-10 h-10 rounded-full border-2 border-[#4a9eff]"
/>

<!-- Avatar with status -->
<div class="relative inline-block">
  <img src="/avatar.jpg" alt="User" class="w-12 h-12 rounded-full" />
  <span
    class="absolute bottom-0 right-0 w-3 h-3 bg-[#4ade80] 
               border-2 border-[#1a1a1a] rounded-full"
  ></span>
</div>

<!-- Avatar stack -->
<div class="flex -space-x-2">
  <img
    src="/avatar1.jpg"
    class="w-10 h-10 rounded-full border-2 border-[#1a1a1a]"
  />
  <img
    src="/avatar2.jpg"
    class="w-10 h-10 rounded-full border-2 border-[#1a1a1a]"
  />
  <img
    src="/avatar3.jpg"
    class="w-10 h-10 rounded-full border-2 border-[#1a1a1a]"
  />
  <div
    class="w-10 h-10 rounded-full bg-[#3a3a3a] border-2 border-[#1a1a1a]
              flex items-center justify-center text-xs text-[#ccc]"
  >
    +5
  </div>
</div>
```

### Skeleton Loader

```html
<!-- Content skeleton while loading -->
<div class="space-y-4 animate-pulse">
  <div class="h-4 bg-[#2a2a2a] rounded w-3/4"></div>
  <div class="h-4 bg-[#2a2a2a] rounded"></div>
  <div class="h-4 bg-[#2a2a2a] rounded w-5/6"></div>
  <div class="h-32 bg-[#2a2a2a] rounded"></div>
</div>

<!-- Card skeleton -->
<div class="bg-[#2a2a2a] rounded-lg p-6 animate-pulse">
  <div class="flex items-center gap-4 mb-4">
    <div class="w-12 h-12 bg-[#3a3a3a] rounded-full"></div>
    <div class="flex-1 space-y-2">
      <div class="h-4 bg-[#3a3a3a] rounded w-1/4"></div>
      <div class="h-3 bg-[#3a3a3a] rounded w-1/3"></div>
    </div>
  </div>
  <div class="space-y-2">
    <div class="h-3 bg-[#3a3a3a] rounded"></div>
    <div class="h-3 bg-[#3a3a3a] rounded w-5/6"></div>
  </div>
</div>
```

### Empty State

```html
<!-- No data empty state -->
<div class="flex flex-col items-center justify-center py-16 text-center">
  <div
    class="w-20 h-20 bg-[#2a2a2a] rounded-full flex items-center 
              justify-center text-4xl mb-4"
  >
    📭
  </div>
  <h3 class="text-xl font-semibold text-white mb-2">No items yet</h3>
  <p class="text-[#888] mb-6 max-w-sm">
    Get started by creating your first item
  </p>
  <button
    class="bg-[#4a9eff] text-black px-6 py-3 rounded-lg font-semibold
                 hover:bg-[#6ab0ff] transition-colors"
  >
    Create Item
  </button>
</div>
```

### Search Bar

```html
<!-- Search input with icon -->
<div class="relative max-w-md">
  <input
    type="search"
    placeholder="Search..."
    class="w-full bg-[#1a1a1a] border-2 border-[#3a3a3a] text-white 
           pl-10 pr-4 py-3 rounded-lg focus:outline-none focus:border-[#4a9eff]
           placeholder:text-[#888]"
  />
  <span class="absolute left-3 top-1/2 -translate-y-1/2 text-[#888]"> 🔍 </span>
</div>
```

### Pagination

```html
<!-- Page navigation -->
<div class="flex items-center justify-center gap-2">
  <button
    class="px-3 py-2 bg-[#2a2a2a] border border-[#3a3a3a] rounded
                 hover:bg-[#3a3a3a] disabled:opacity-50"
    disabled
  >
    ←
  </button>

  <button class="px-4 py-2 bg-[#4a9eff] text-black rounded font-semibold">
    1
  </button>
  <button class="px-4 py-2 bg-[#2a2a2a] text-white rounded hover:bg-[#3a3a3a]">
    2
  </button>
  <button class="px-4 py-2 bg-[#2a2a2a] text-white rounded hover:bg-[#3a3a3a]">
    3
  </button>
  <span class="px-2 text-[#888]">...</span>
  <button class="px-4 py-2 bg-[#2a2a2a] text-white rounded hover:bg-[#3a3a3a]">
    10
  </button>

  <button
    class="px-3 py-2 bg-[#2a2a2a] border border-[#3a3a3a] rounded
                 hover:bg-[#3a3a3a]"
  >
    →
  </button>
</div>
```

### Sidebar Layout

```html
<!-- App layout with sidebar -->
<div class="flex h-screen">
  <!-- Sidebar -->
  <aside class="w-64 bg-[#1a1a1a] border-r border-[#3a3a3a] overflow-y-auto">
    <div class="p-6">
      <h2 class="text-xl font-bold text-white mb-6">Navigation</h2>
      <nav class="space-y-2">
        <a
          href="#"
          class="flex items-center gap-3 px-4 py-3 bg-[#2a2a2a] 
                          text-[#4a9eff] rounded-lg"
        >
          <span>📊</span>
          <span class="font-medium">Dashboard</span>
        </a>
        <a
          href="#"
          class="flex items-center gap-3 px-4 py-3 text-[#888] 
                          hover:bg-[#2a2a2a] hover:text-white rounded-lg transition-colors"
        >
          <span>📁</span>
          <span>Projects</span>
        </a>
        <a
          href="#"
          class="flex items-center gap-3 px-4 py-3 text-[#888] 
                          hover:bg-[#2a2a2a] hover:text-white rounded-lg transition-colors"
        >
          <span>⚙️</span>
          <span>Settings</span>
        </a>
      </nav>
    </div>
  </aside>

  <!-- Main content -->
  <main class="flex-1 overflow-y-auto bg-[#0a0a0a] p-6">
    <h1 class="text-3xl font-bold text-white mb-6">Page Content</h1>
    <!-- Content here -->
  </main>
</div>
```

### Hero Section

```html
<!-- Landing page hero -->
<section
  class="min-h-screen flex items-center justify-center 
                bg-gradient-to-b from-[#1a1a1a] to-[#0a0a0a] px-4"
>
  <div class="max-w-4xl text-center">
    <h1
      class="text-5xl md:text-7xl font-bold text-white mb-6 
               leading-tight"
    >
      Build Amazing Apps
    </h1>
    <p class="text-xl text-[#ccc] mb-8 max-w-2xl mx-auto">
      The modern framework for creating fast, reactive web applications
    </p>
    <div class="flex flex-col sm:flex-row gap-4 justify-center">
      <button
        class="bg-[#4a9eff] text-black px-8 py-4 rounded-lg 
                     text-lg font-bold hover:bg-[#6ab0ff] transition-all
                     hover:-translate-y-1 shadow-lg"
      >
        Get Started
      </button>
      <button
        class="bg-transparent border-2 border-[#4a9eff] text-[#4a9eff] 
                     px-8 py-4 rounded-lg text-lg font-bold 
                     hover:bg-[#4a9eff] hover:text-black transition-all"
      >
        Learn More
      </button>
    </div>
  </div>
</section>
```

### Breadcrumbs

```html
<!-- Navigation breadcrumb trail -->
<nav class="flex items-center gap-2 text-sm">
  <a href="#" class="text-[#888] hover:text-white transition-colors">Home</a>
  <span class="text-[#666]">/</span>
  <a href="#" class="text-[#888] hover:text-white transition-colors"
    >Products</a
  >
  <span class="text-[#666]">/</span>
  <span class="text-white font-semibold">Electronics</span>
</nav>
```

### Progress Bar

```html
<!-- Loading progress bar -->
<div class="w-full bg-[#2a2a2a] rounded-full h-2 overflow-hidden">
  <div
    class="bg-[#4a9eff] h-full rounded-full transition-all duration-300"
    style="width: 65%"
  ></div>
</div>

<!-- With label -->
<div>
  <div class="flex justify-between text-sm text-[#ccc] mb-2">
    <span>Upload Progress</span>
    <span>65%</span>
  </div>
  <div class="w-full bg-[#2a2a2a] rounded-full h-2">
    <div class="bg-[#4ade80] h-full rounded-full" style="width: 65%"></div>
  </div>
</div>
```

### Accordion

```html
<!-- Collapsible accordion item -->
<div class="border border-[#3a3a3a] rounded-lg overflow-hidden">
  <button
    class="w-full flex items-center justify-between px-6 py-4 
                 bg-[#2a2a2a] hover:bg-[#3a3a3a] transition-colors text-left"
  >
    <span class="font-semibold text-white">What is Svelte?</span>
    <span class="text-[#888]">▼</span>
  </button>
  <div class="px-6 py-4 bg-[#1a1a1a] text-[#ccc] leading-relaxed">
    Svelte is a radical new approach to building user interfaces...
  </div>
</div>
```

---

## Tips for Learning

1. **Mobile-First**: Design for mobile first, then add `md:`, `lg:` for larger screens
2. **Combine States**: `hover:bg-blue-400 focus:ring-2` - Multiple states work together
3. **Use Arbitrary Values Sparingly**: Stick to Tailwind's scale when possible
4. **Group Related Classes**: Keep layout, colors, and states grouped for readability
5. **DevTools**: Inspect elements to see which Tailwind classes are applied
6. **Copy Real Patterns**: Start with proven patterns and customize them for your needs
7. **Component Library**: Build a personal library of these patterns for reuse

---

## Quick Reference Cheat Sheet

### Spacing Scale

- `0` = 0px
- `1` = 0.25rem (4px)
- `2` = 0.5rem (8px)
- `3` = 0.75rem (12px)
- `4` = 1rem (16px)
- `5` = 1.25rem (20px)
- `6` = 1.5rem (24px)
- `8` = 2rem (32px)
- `10` = 2.5rem (40px)
- `12` = 3rem (48px)
- `16` = 4rem (64px)
- `20` = 5rem (80px)
- `24` = 6rem (96px)

### Font Sizes

- `xs` = 12px
- `sm` = 14px
- `base` = 16px
- `lg` = 18px
- `xl` = 20px
- `2xl` = 24px
- `3xl` = 30px
- `4xl` = 36px
- `5xl` = 48px

---

This reference covers all Tailwind classes used in your Svelte study guides. Keep it handy while coding! 🚀
