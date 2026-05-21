# Routing Directives

Client-side SPA routing with params, guards, nested routes, file-based routing, named outlets, and View Transitions.

## Contents

- [route](#route) -- define a route or navigation link
- [route-view](#route-view) -- route outlet for matched template
- [File-Based Routing](#file-based-routing) -- automatic route resolution from folder structure
- [route-active](#route-active) -- CSS class for active route links
- [guard](#guard) -- route guard with redirect
- [lazy](#lazy) -- control when route templates are fetched
- [$route Context](#route-context) -- path, params, query, hash, matched
- [Programmatic Navigation](#programmatic-navigation) -- $router.push, back, replace
- [Nested Routes](#nested-routes) -- nested route-view outlets
- [Named Outlets](#named-outlets) -- multiple route-view outlets
- [outlet](#outlet-attribute) -- assign template to named outlet
- [Anchor Links](#anchor-links) -- hash-based anchor navigation
- [Programmatic Route Registration](#programmatic-route-registration) -- NoJS.router.register
- [$router.push() and $router.replace() Return Promises](#routerpush-and-routerreplace-return-promises)
- [Automatic 404 Fallback](#automatic-404-fallback) -- default and custom 404 pages
- [Remote 404 Templates](#remote-404-templates) -- external 404 template loading
- [File-Based Routing 404](#file-based-routing-404) -- HTTP 404 fallback chain
- [Named Outlet Wildcards](#named-outlet-wildcards) -- per-outlet wildcard fallback
- [$route.matched](#routematched) -- explicit vs wildcard match detection
- [Route Head Attributes](#route-head-attributes) -- page-title, page-description, page-canonical, page-jsonld on routes
- [Accessibility -- focusBehavior](#accessibility--focusbehavior) -- automatic focus management
- [Scroll Behavior](#scroll-behavior) -- scroll position after navigation
- [Hash Mode Warning](#hash-mode-warning) -- SEO impact of hash URLs
- [SPA Deployment Fallback](#spa-deployment-fallback) -- server configuration for history mode
- [View Transition API](#view-transition-api) -- native route transitions
  - [How It Works](#how-it-works)
  - [Built-in Presets](#built-in-presets) -- slide, fade, scale, none
  - [Direction Detection](#direction-detection) -- forward/backward awareness
  - [Custom CSS Animations](#custom-css-animations) -- override/extend presets
  - [Accessibility](#accessibility) -- prefers-reduced-motion
  - [Configuration](#configuration) -- enable/disable View Transition API
  - [Legacy Class-Based Transitions](#legacy-class-based-transitions-deprecated-for-routes)
- [Quick Reference](#quick-reference) -- minimal route transition setup

---

## Routing

Client-side SPA routing with params, guards, nested routes, file-based routing, and named outlets.

### `route`

Define a route or create a navigation link.

**Syntax:** `<a route="/path">` (navigation link) or `<template route="/path">` (route definition)

Route patterns support `:param` placeholders and `*` wildcard for catch-all/404.

```html
<!-- Navigation links -->
<a route="/">Home</a>
<a route="/about">About</a>
<a route="/users/:id">User Detail</a>

<!-- Route templates -->
<template route="/">
  <h1>Home</h1>
</template>

<template route="/users/:id">
  <div get="/api/users/{$route.params.id}" as="user">
    <h1 bind="user.name"></h1>
  </div>
</template>

<!-- Catch-all 404 -->
<template route="*">
  <h1>404 -- Page Not Found</h1>
  <p>Path <code bind="$route.path"></code> does not exist.</p>
</template>
```

### `route-view`

Route outlet that renders the matched route template.

**Syntax:** `<main route-view>` or `<aside route-view="name">`

**Attributes:**

| Attribute | Default | Description |
|-----------|---------|-------------|
| `route-view` | (default outlet) | Renders matched route. Value = outlet name for named outlets |
| `src` | `"pages"` | Base folder for file-based routing |
| `route-index` | `"index"` | Filename for the root route `/` |
| `ext` | `".tpl"` | File extension (fallback: `".html"`) |
| `i18n-ns` | -- | When present, auto-derives i18n namespace from filename |

```html
<!-- Standard outlet -->
<main route-view></main>

<!-- File-based routing -->
<main route-view src="./pages/" route-index="overview"></main>

<!-- Named outlets -->
<main route-view></main>
<aside route-view="sidebar"></aside>
```

### File-Based Routing

Point `route-view` at a folder and No.JS resolves routes to template files automatically:

```html
<main route-view src="./pages/" route-index="overview"></main>
```

Given this folder structure:
```
pages/
  overview.tpl    -> /
  analytics.tpl   -> /analytics
  users.tpl       -> /users
  settings.tpl    -> /settings
```

Explicit `<template route="...">` declarations always take priority, so you can mix both approaches.

### `route-active`

CSS class added to active route links.

**Syntax:** `<a route="/path" route-active="active-class">`

```html
<a route="/" route-active="active">Home</a>
<a route="/about" route-active="active">About</a>
<!-- Exact match only (won't match /users/123) -->
<a route="/users" route-active-exact="active">Users</a>
```

### `guard`

Route guard -- prevents navigation if condition is falsy. Redirects to specified path.

**Syntax:** `<template route="/admin" guard="expression" redirect="/login">`

```html
<template route="/dashboard" guard="$store.auth.user" redirect="/login">
  <h1>Dashboard</h1>
</template>

<template route="/login" guard="!$store.auth.user" redirect="/dashboard">
  <form post="/api/login">...</form>
</template>
```

### `lazy`

Control when route templates are fetched.

**Syntax:** `<template route="/path" lazy="ondemand">` or `lazy="priority"`

| Value | Phase | Behaviour |
|-------|-------|-----------|
| (absent) | Auto | Active route loads before first render; others preload silently after |
| `lazy="priority"` | 0 | Fetched first, before all other templates |
| `lazy="ondemand"` | On demand | Only fetched the first time the user navigates to that route |

```html
<template route="/heavy" src="./pages/heavy.tpl" lazy="ondemand"></template>
<template route="/critical" src="./critical.tpl" lazy="priority"></template>
```

### `$route` Context

| Property | Description |
|----------|-------------|
| `$route.path` | Current path (e.g. `"/users/42"`) |
| `$route.params` | Route parameters (e.g. `{ id: "42" }`) |
| `$route.query` | Query string params (e.g. `{ q: "hello" }`) |
| `$route.hash` | URL hash (e.g. `"#section"`) |
| `$route.matched` | `true` if an explicit route matched, `false` for wildcard/fallback |

### Programmatic Navigation

```html
<button on:click="$router.push('/users/42')">Go to User</button>
<button on:click="$router.back()">Go Back</button>
<button on:click="$router.replace('/new-path')">Replace</button>
```

### Nested Routes

```html
<template route="/settings">
  <nav>
    <a route="/settings/profile">Profile</a>
    <a route="/settings/security">Security</a>
  </nav>
  <div route-view></div>  <!-- Nested route content -->
</template>
<template route="/settings/profile">
  <h2>Profile Settings</h2>
</template>
```

### Named Outlets

```html
<main route-view></main>
<aside route-view="sidebar"></aside>

<template route="/home">
  <h1>Home page</h1>
</template>
<template route="/home" outlet="sidebar">
  <nav>Home navigation</nav>
</template>
```

Outlets with no matching template for the active route are cleared on navigation.

### `outlet` Attribute

Assign a route template to a specific named outlet.

**Syntax:** `<template route="/path" outlet="outletName">`

```html
<template route="/dashboard" outlet="sidebar">
  <nav>Dashboard Nav</nav>
</template>
```

If omitted, the template renders in the default (unnamed) `route-view`.

### Anchor Links

When using hash mode (`useHash: true`), anchor links with `#` are handled automatically with smooth scrolling. Active anchor links receive the `.active` class.

```html
<a route="#features">Features</a>
<a route="#pricing">Pricing</a>

<section id="features">...</section>
<section id="pricing">...</section>
```

### Programmatic Route Registration

Register routes programmatically via the router:

```javascript
NoJS.router.register('/path', templateElement, 'outletName');
```

### `$router.push()` and `$router.replace()` Return Promises

Programmatic navigation is asynchronous:

```html
<button on:click="await $router.push('/users/42')">Navigate</button>
```

### Automatic 404 Fallback

If no `route="*"` wildcard is defined, No.JS automatically shows a minimal 404 message. Define your own for a custom fallback:

```html
<template route="*">
  <h1>Page not found</h1>
  <a route="/">Go home</a>
</template>
```

### Remote 404 Templates

Wildcard routes support `src` for remote template loading:

```html
<template route="*" src="/templates/404.html"></template>
```

### File-Based Routing 404

When file-based routing gets an HTTP 404 for a template file, it triggers the wildcard route fallback chain.

### Named Outlet Wildcards

Named outlets support their own wildcards with a fallback chain: local outlet wildcard → global wildcard → built-in empty.

```html
<template route="*" outlet="sidebar">
  <p>No sidebar for this page</p>
</template>
```

### `$route.matched`

Use `$route.matched` to check if an explicit route was matched (vs wildcard fallback):

```html
<template route="*">
  <div if="!$route.matched">
    <h1>404 - <span bind="$route.path"></span> not found</h1>
  </div>
</template>
```

### Route Head Attributes

Set `<head>` metadata declaratively on each `<template route>`. Updated on every navigation. Expressions can use `$route` and `$store`.

| Attribute | Description |
|-----------|-------------|
| `page-title` | Sets `document.title`. Value is a No.JS expression (single-quoted strings: `'About | Site'`) |
| `page-description` | Creates/updates `<meta name="description">` |
| `page-canonical` | Creates/updates `<link rel="canonical">` |
| `page-jsonld` | Creates/updates `<script type="application/ld+json" data-nojs>`. Value is a JSON string with `{placeholder}` interpolation |

```html
<!-- Static title -->
<template route="/about" page-title="'About Us | My Store'">
  <h1>About</h1>
</template>

<!-- Dynamic from route params -->
<template route="/products/:id"
          page-title="'Product ' + $route.params.id + ' | Store'"
          page-description="'View product ' + $route.params.id"
          page-canonical="'/products/' + $route.params.id"
          page-jsonld='{"@type":"Product","name":"{$route.params.id}"}'>
  <h1>Product Detail</h1>
</template>

<!-- Expression from global store -->
<template route="/account" page-title="$store.user.name + ' — My Account'">
  <h1>Account</h1>
</template>
```

- Evaluated once per navigation (not continuously reactive — `$store` changes after navigation do not re-update the title)
- String literals require single quotes inside the double-quoted attribute: `page-title="'My Title'"` ✅, backticks are not valid in HTML attributes
- Default outlet only — secondary outlets do not update head metadata
- If a `<div hidden page-title>` body directive is also present, whichever runs last wins

### Accessibility -- `focusBehavior`

Enable automatic focus management after SPA navigation:

```javascript
NoJS.config({ router: { focusBehavior: 'auto' } });
```

When `'auto'`, focus moves to the first suitable target in this order: `[autofocus]` → `[tabindex="-1"]` → `h1` → outlet element. Default is `'none'` (no change to focus).

### Scroll Behavior

Control scroll position after route navigation:

```javascript
NoJS.config({ router: { scrollBehavior: 'top' } });
```

| Value | Description |
|-------|-------------|
| `"top"` | Scroll to top of page after navigation |
| `"smooth"` | Smooth-scroll to top after navigation |
| `"preserve"` | Keep current scroll position (no scroll change) |

### Hash Mode Warning

Enabling `useHash: true` logs a console warning about SEO impact (hash URLs are not indexed by search engines). Silence it with:

```javascript
NoJS.config({ router: { useHash: true, suppressHashWarning: true } });
```

### SPA Deployment Fallback

History mode requires the server to return `index.html` for all routes. Common configs:

```nginx
# nginx
location / { try_files $uri $uri/ /index.html; }
```

```apache
# Apache .htaccess
RewriteRule ^ index.html [L]
```

```
# Netlify _redirects
/*  /index.html  200
```

```json
// Vercel vercel.json
{ "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }] }
```

---

## View Transition API

No.JS uses the [View Transition API](https://developer.mozilla.org/en-US/docs/Web/API/View_Transition_API) for route transitions by default. When a `route-view` outlet has a `transition` attribute, route changes are wrapped in `document.startViewTransition()` for smooth, native-quality page transitions.

### How It Works

1. **Outlet setup** -- No.JS dynamically sets `view-transition-name: route-content` on any `route-view` element that has a `transition` attribute.
2. **Navigation trigger** -- When the router navigates, the DOM swap (removing old content, inserting new content) is wrapped inside `document.startViewTransition({ update, types })`.
3. **Direction detection** -- The router tracks navigation direction (forward vs backward) and passes it as a type to `startViewTransition({ types: ['slide', 'forward'] })`. This enables direction-aware CSS animations.
4. **CSS animations** -- Built-in preset CSS targets `::view-transition-old(route-content)` and `::view-transition-new(route-content)` pseudo-elements with scoping via `:active-view-transition-type()`.
5. **Fallback** -- If the View Transition API is not supported by the browser, the DOM swap happens instantly without animation.

### Built-in Presets

| Preset | Description |
|--------|-------------|
| `slide` | Directional slide animation. Detects forward/backward navigation and slides content left/right accordingly. |
| `fade` | Cross-fade between the old and new route content. |
| `scale` | Old content scales down and fades out; new content scales up and fades in. |
| `none` | No transition animation. The DOM swap happens instantly. |

```html
<!-- Slide with automatic forward/backward direction -->
<main route-view transition="slide"></main>

<!-- Cross-fade -->
<main route-view transition="fade"></main>

<!-- Scale -->
<main route-view transition="scale"></main>

<!-- No animation -->
<main route-view transition="none"></main>
```

### Direction Detection

The `slide` preset automatically detects navigation direction:

- **Forward** -- triggered by `$router.push()`, clicking `<a route>` links, or `$router.forward()`
- **Backward** -- triggered by `$router.back()`, browser back button, or `popstate` events going backward in history

Direction types are passed to `document.startViewTransition({ types })`:
- Forward navigation: `types: ['slide', 'forward']`
- Backward navigation: `types: ['slide', 'backward']`

CSS rules use `:active-view-transition-type()` to scope animations by direction:

```css
:active-view-transition-type(forward) {
  &::view-transition-old(route-content) {
    animation: slide-out-left 0.3s ease;
  }
  &::view-transition-new(route-content) {
    animation: slide-in-right 0.3s ease;
  }
}

:active-view-transition-type(backward) {
  &::view-transition-old(route-content) {
    animation: slide-out-right 0.3s ease;
  }
  &::view-transition-new(route-content) {
    animation: slide-in-left 0.3s ease;
  }
}
```

### Custom CSS Animations

Override or extend built-in presets by targeting the View Transition pseudo-elements:

```css
/* Target the old (outgoing) and new (incoming) route content */
::view-transition-old(route-content) {
  animation: my-custom-out 0.4s ease-out;
}
::view-transition-new(route-content) {
  animation: my-custom-in 0.4s ease-in;
}

/* Scope to a specific preset */
:active-view-transition-type(fade) {
  &::view-transition-old(route-content) {
    animation: custom-fade-out 0.5s ease;
  }
  &::view-transition-new(route-content) {
    animation: custom-fade-in 0.5s ease;
  }
}

/* Define custom keyframes */
@keyframes my-custom-out {
  from { opacity: 1; transform: translateY(0); }
  to { opacity: 0; transform: translateY(-20px); }
}
@keyframes my-custom-in {
  from { opacity: 0; transform: translateY(20px); }
  to { opacity: 1; transform: translateY(0); }
}
```

### Accessibility

Built-in presets respect `prefers-reduced-motion`:

```css
@media (prefers-reduced-motion: reduce) {
  ::view-transition-old(route-content),
  ::view-transition-new(route-content) {
    animation-duration: 0.01s !important;
  }
}
```

Users who prefer reduced motion see instant transitions without jarring animations.

### Configuration

The View Transition API is enabled by default. Control it via `NoJS.config()`:

```javascript
// Default -- View Transition API enabled
NoJS.config({
  router: {
    viewTransition: true   // default
  }
});

// Opt-out -- fall back to legacy class-based transitions
NoJS.config({
  router: {
    viewTransition: false
  }
});
```

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `router.viewTransition` | boolean | `true` | Enable/disable View Transition API for route changes. When `false`, `transition` on `route-view` uses the legacy class-based system. |

### Legacy Class-Based Transitions (Deprecated for Routes)

When `router.viewTransition` is set to `false`, or for non-router elements, the `transition` attribute uses the class-based system:

| Class | When |
|-------|------|
| `{name}-enter` | Start state of enter |
| `{name}-enter-active` | Active state of enter |
| `{name}-enter-to` | End state of enter |
| `{name}-leave` | Start state of leave |
| `{name}-leave-active` | Active state of leave |
| `{name}-leave-to` | End state of leave |

Class-based transitions on `route-view` are deprecated in favor of the View Transition API. They remain the default for non-router elements (`if`, `show`, etc.).

---

## Quick Reference

### Minimal Route Transition Setup

```html
<script src="https://cdn.no-js.dev/"></script>

<nav>
  <a route="/">Home</a>
  <a route="/about">About</a>
</nav>

<!-- Just add transition="slide" to the outlet -->
<main route-view transition="slide"></main>

<template route="/">
  <h1>Home</h1>
</template>

<template route="/about">
  <h1>About</h1>
</template>
```

That is all you need. No JavaScript configuration required -- the View Transition API is enabled by default.
