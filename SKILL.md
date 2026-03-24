---
name: nojs
metadata:
  version: 1.10.1
description: Expert-level knowledge of the No.JS HTML-first reactive framework for building dynamic web applications using only HTML attributes. Use this skill whenever the user mentions No.JS, NoJS, "no javascript framework", HTML-first framework, or is writing HTML with reactive attributes like bind, state, get, each, on:click, model, route, store, computed, watch, if/else, show/hide, foreach, validate, animate, drag, drop, t (i18n), class-*, style-*, or bind-*. Also use when the user asks about declarative HTML frameworks, zero-JS frameworks, or wants to build a web app without writing JavaScript. Even if the user doesn't mention No.JS by name, activate this skill when you see HTML attributes that match No.JS directive patterns.
---

# NoJS-Skill

No.JS is an HTML-first reactive framework (~24KB gzipped, zero dependencies) that replaces JavaScript with declarative HTML attributes. CSP-compliant via a custom expression parser (no eval). Include one `<script>` tag and start writing directives.

```html
<script src="https://cdn.no-js.dev/"></script>
<body base="https://jsonplaceholder.typicode.com">
  <div get="/users" as="users">
    <div each="user in users">
      <h2 bind="user.name"></h2>
      <p bind="user.email"></p>
    </div>
  </div>
</body>
```

No `app.mount()`, no `createApp()`, no build step. It just works.

## When to use

Use this skill when:

- The user mentions **No.JS**, **NoJS**, or the **HTML-first reactive framework**
- The user is writing HTML with No.JS directive attributes (`bind`, `state`, `get`, `each`, `on:click`, `model`, `route`, `store`, `foreach`, `validate`, `animate`, `drag`, `drop`, `t`, `class-*`, `style-*`, `bind-*`)
- The user asks about **declarative HTML frameworks** or wants to build a **web app without writing JavaScript**
- The user needs to **scaffold**, **validate**, or **debug** No.JS templates
- You see HTML attributes that match No.JS directive patterns, even without explicit mention

## Instructions

### 1. Understand the framework architecture

No.JS works by walking the DOM on `DOMContentLoaded`, matching HTML attributes to directives, and executing them by priority:

| Priority | Directives | Purpose |
|----------|-----------|---------|
| 0 | `state`, `store` | Initialize reactive data first |
| 1 | `get`, `post`, `put`, `patch`, `delete`, `error-boundary`, `i18n-ns` | Fetch data and error/i18n setup |
| 2 | `computed`, `watch` | Derive values and observe changes |
| 5 | `ref` | Element references |
| 10 | `if`, `else-if`, `else`, `switch`, `each`, `foreach`, `use`, `drag-list` | Structural (add/remove DOM) |
| 15 | `drag`, `drop` | Drag and drop setup |
| 16 | `drag-multiple` | Multi-select drag |
| 20 | `bind`, `bind-*`, `bind-html`, `model`, `class-*`, `style-*`, `on:*`, `show`, `hide`, `t`, `call`, `trigger` | Rendering, events, i18n, actions |
| 30 | `validate` | Form validation side effects |

Data lives in Proxy-backed reactive contexts that inherit from parent elements (like lexical scoping). When data changes, every bound element updates automatically.

### 2. Know the core directives

**Data Fetching** - `get="/url"` with `as="varName"`, `loading`, `error`, `empty`, `success`, `refresh`, `cached`, `into`, `debounce`, `headers`, `params`. URLs support interpolation: `get="/users/{userId}"` re-fetches reactively. Mutation verbs: `post`, `put`, `patch`, `delete`.

**State** - `state="{ key: value }"` (local), `store="name"` (global via `$store.name`), `computed="name" expr="expr"`, `watch="prop" on:change="handler"`, `persist`/`persist-fields`. Note: stores created via `NoJS.config({ stores })` won't be overwritten by later `<div store>` with the same name. Use `NoJS.notify()` after mutating stores from JavaScript outside expressions.

**Binding** - `bind="expr"` (text), `bind-html="expr"` (sanitized HTML), `bind-*="expr"` (any attribute), `model="prop"` (two-way for form inputs).

**Conditionals** - `if`/`then`/`else`, `else-if`, `show`/`hide` (CSS toggle), `switch`/`case`/`default`.

**Loops** - `each="item in items"`, `foreach="item" from="items"` (with `filter`, `sort`, `limit`, `offset`, `key`). Loop vars: `$index`, `$count`, `$first`, `$last`, `$even`, `$odd`.

**Events** - `on:click="expr"` with modifiers `.prevent`, `.stop`, `.once`, `.self`, `.debounce.300`, `.throttle.100`. Key mods: `.enter`, `.escape`, `.tab`, `.space`, `.delete`, `.backspace`, `.up`, `.down`, `.left`, `.right`, `.ctrl`, `.alt`, `.shift`, `.meta`. Lifecycle: `on:init`, `on:mounted`, `on:updated`, `on:unmounted`. Vars: `$event`, `$el`.

**Styling** - `class-active="cond"`, `class-map="{ ... }"`, `style-color="expr"`, `style-map="{ ... }"`.

**Forms** - `<form validate>` with `$form` context (`valid`, `dirty`, `submitting`, `errors`, `firstError`, `reset()`). Field rules: `validate="required,email,min:5"`. Per-field state: `$form.fields.email.valid`. Triggers: `validate-on="blur"`. Conditional: `validate-if="expr"`. Auto-disables submit buttons when `$form.submitting`. Custom: `NoJS.validator('name', fn)`.

**Routing** - `<a route="/path">`, `<template route="/users/:id">`, `<main route-view>`. Named outlets: `outlet="sidebar"`. Context: `$route.params`, `$route.query`, `$route.path`, `$route.matched`. Guards: `guard="expr"`. File-based: `route-view src="pages/"`. Catch-all: `route="*"`. Programmatic: `NoJS.router.push()`, `.replace()`, `.back()`, `.forward()` (return Promises).

**Animations** - `animate="fadeIn"`, `transition="slide"`, `animate-stagger="50"`. Built-in: fadeIn, fadeOut, fadeInUp, slideInLeft, zoomIn, bounceIn, etc.

**i18n** - `t="key"`, `t-name="expr"`, `t-html` (render translation as sanitized HTML), `i18n-ns="namespace"`, pluralization `"one item | {count} items"`. Namespace mode: `loadPath: '/locales/{locale}/{ns}.json'`. Formatting filters: `currency`, `date`, `datetime`, `relative`, `number`, `percent`. Context: `$i18n.locale`.

**DnD** - `drag`, `drop="handler"`, `drag-list="items"`, `drag-multiple`.

**Templates** - `<template id="name">`, `use="tplId"`, `<slot>`, `<template src="/path.html">` (recursive loading), `include="#tpl"`. Template variables: `var="name"`. Lazy loading: `lazy` (auto), `lazy="priority"` (phase 0), `lazy="ondemand"` (routes only). Loading placeholder: `loading="#tpl"`.

**Refs** - `ref="name"` → `$refs.name`, `call="/url"`, `trigger="eventName"`.

**Errors** - `error="#tpl"`, `error-boundary`, `NoJS.config({ onError })`.

### 3. Use the expression syntax correctly

Expressions support JavaScript-like syntax against the reactive context:
- Property access: `user.name`, `items[0]`, `user?.address?.city`
- Arithmetic, comparisons, ternary, template literals
- Pipes (filters): `name | uppercase`, `price | currency:'USD'`
- Assignments: `count++`, `name = 'John'`
- Function calls: `items.push(newItem)`
- The evaluator uses an allow-list approach: `_SAFE_GLOBALS` for JS built-ins and `_BROWSER_GLOBALS` for curated browser APIs. `fetch`, `XMLHttpRequest`, `localStorage`, `sessionStorage`, `WebSocket`, and `indexedDB` are NOT on the allow-list. Spread operations filter `_FORBIDDEN_PROPS` (`__proto__`, `constructor`, `prototype`)

### 4. Apply filters via pipe syntax

`bind="value | filter1 | filter2:arg"` - 32 built-in filters across text, numbers, arrays, dates, and utilities. See [references/filters.md](references/filters.md) for the complete list. Custom: `NoJS.filter('name', fn)`.

### 5. Use the public API when needed

```javascript
NoJS.config({ baseApiUrl, headers, timeout, retries, router, i18n, stores })
NoJS.init(root?)           // Auto-called by CDN; manual for ESM/CJS
NoJS.directive(name, { priority, init(el, attrName, value) })
NoJS.filter(name, fn)      // Custom filter
NoJS.validator(name, fn)   // Custom validator
NoJS.i18n({ loadPath, defaultLocale, fallbackLocale, persist })
NoJS.on(event, callback)   // Global event bus
NoJS.interceptor('request' | 'response', fn)
NoJS.store                 // Access global stores
NoJS.notify()              // Trigger UI update after external mutation
NoJS.router                // push(), replace(), back(), forward()
NoJS.locale = 'pt'         // Get/set locale
NoJS.version               // "1.10.0"
```

### 6. Follow these rules when generating No.JS code

1. **Always set `as` on fetch directives** - `get="/users" as="users"` not just `get="/users"`
2. **Use `each` for simple lists, `foreach` for complex** - foreach adds filter/sort/limit/key
3. **Use `show`/`hide` for frequent toggles, `if` for rare** - show is CSS-only, if recreates DOM
4. **Use templates for reuse** - `<template id="name">` + `then="name"` or `use="name"`
5. **Scope state close to usage** - Put `state` on the nearest common ancestor
6. **Use `$store` for cross-component data** - Auth, theme, cart, notifications
7. **Add `key` on loops** - `each="item in items" key="item.id"` for efficient updates
8. **Use filters for display** - `bind="price | currency"` not inline formatting
9. **Events use colon syntax** - `on:click` not `onclick` or `on-click`
10. **Validate forms declaratively** - `<form validate>` + `validate="required,email"` on inputs

### 7. Validate templates for common mistakes

Check for: directive typos (`bnd`→`bind`, `on-click`→`on:click`), missing `as` on `get`, `each` without `in`, `foreach` without `from`, `model` on non-form elements, unsanitized `bind-html`. See [references/validation.md](references/validation.md) for the full checklist.

### 8. Consult reference files for deep details

- [references/directives.md](references/directives.md) - Complete directive reference with all attributes and examples
- [references/filters.md](references/filters.md) - All 32 built-in filters with syntax
- [references/api.md](references/api.md) - Full JavaScript API reference
- [references/patterns.md](references/patterns.md) - Common patterns, scaffolds, and best practices
- [references/validation.md](references/validation.md) - Template validation rules and common mistakes

## Ecosystem

- **Website**: https://no-js.dev/
- **CDN**: https://cdn.no-js.dev/
- **npm**: `npm install @erickxavier/no-js`
- **GitHub**: https://github.com/ErickXavier/no-js
- **VS Code Extension**: NoJS LSP (completions, diagnostics, hover docs for 39+ directives)
- **Full docs**: https://no-js.dev/llms-full.txt
