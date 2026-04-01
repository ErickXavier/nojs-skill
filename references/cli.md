# NoJS CLI Reference

The official command-line tool for the No.JS framework. Scaffold projects, compile and optimize HTML for production, run a dev server with live reload, and validate templates.

## Install

```bash
# macOS / Linux
curl -fsSL https://no-js.dev/install.sh | sh

# Or via Cargo
cargo install nojs-cli
```

After installing, the `nojs` command is available globally.

---

## Commands Overview

| Command | Alias | Description |
|---------|-------|-------------|
| `nojs init [path]` | `i` | Scaffold a new No.JS project |
| `nojs build [dir]` | `b` | Compile and optimize HTML for production |
| `nojs serve [path]` | `s` | Local dev server with SSE live reload |
| `nojs validate [files]` | `v` | Validate No.JS templates and i18n |
| `nojs help` | | Show help |
| `nojs version` | | Show CLI version |

All commands support `-h` or `--help` for command-specific help.

---

## `nojs init`

Interactive wizard that generates a ready-to-go No.JS project.

### Usage

```bash
nojs init [path] [options]
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--name <name>` | Project name | Directory name |
| `--routing, -r <y\|n>` | Enable SPA routing | Prompt |
| `--i18n <y\|n>` | Enable internationalization | Prompt |
| `--locales <list>` | Comma-separated locale codes | `en,pt` |
| `--default-locale <locale>` | Default locale | First locale |
| `--api <url>` | Base API URL for fetch directives | None |
| `--yes, -y` | Skip wizard, accept defaults | Off |

### Interactive Wizard

When not all flags are provided and `--yes` is not set, the wizard asks:

1. **Project name** (default: directory name)
2. **Use SPA routing?** (y/N)
3. **Use i18n?** (y/N)
4. If i18n: **Locales** (default: `en, pt`)
5. If i18n: **Default locale** (default: first locale)
6. **Base API URL** (default: none)

Boolean flags accept: `y`, `yes`, `s`, `sim`, `true`, `1` (case-insensitive).

### Generated Files

```plaintext
my-app/
├── index.html              # Main entry with CDN script, routes, i18n
├── assets/
│   └── style.css           # Base stylesheet
├── pages/                  # (if routing)
│   ├── home.tpl
│   └── about.tpl
├── locales/                # (if i18n)
│   ├── en.json
│   └── pt.json
└── nojs.config.json        # Project metadata
```

### Examples

```bash
# Interactive mode
nojs init my-app

# Non-interactive with all options
nojs init my-app --routing --i18n --locales en,pt,es --api https://api.example.com --yes

# Minimal project, no prompts
nojs init my-app -y
```

---

## `nojs serve`

Local HTTP dev server with Server-Sent Events (SSE) live reload.

### Usage

```bash
nojs serve [path] [options]
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--port <port>` | Port number | `3000` |
| `--open` | Open browser on start | Off |
| `--quiet, -q` | Suppress request logging | Off |
| `--no-reload` | Disable live reload | Off |

### Features

- **Live reload via SSE** — watches all files (except hidden and `node_modules/`), broadcasts reload on changes with 100ms debounce
- **SPA fallback** — serves root `index.html` for unmatched clean URLs (no extension), enabling client-side routing
- **Path traversal protection** — resolves all paths and rejects requests outside the served root, blocks null bytes
- **Colored request logging** — green (2xx), cyan (3xx), yellow (4xx), red (5xx)
- **MIME detection** — supports `.html`, `.tpl`, `.css`, `.js`, `.mjs`, `.json`, `.svg`, `.png`, `.jpg`, `.gif`, `.webp`, `.ico`, `.woff`, `.woff2`, `.ttf`, `.md`, `.xml`, `.txt`

### Reload Script

When live reload is enabled, the server injects an SSE client script before `</body>` in every HTML response. The script auto-reconnects on disconnect (2s delay). Max 100 concurrent SSE clients.

### Examples

```bash
nojs serve                  # Serve current directory on port 3000
nojs serve ./docs/          # Serve a specific directory
nojs serve --port 8080      # Custom port
nojs serve --open           # Open browser on start
nojs serve --quiet          # No request logging
nojs serve --no-reload      # Static server only
```

---

## `nojs build`

Integrated compiler and optimization pipeline with 25 passes. Always performs full AOT compilation — no separate `--compile` flag needed.

### Usage

```bash
nojs build [options]
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--input <glob>` | HTML files glob pattern | `**/*.html` |
| `--output <dir>` | Output directory | In-place |
| `--config <path>` | Config file path | Auto-detect |
| `--dry-run` | Preview without writing | |

### What It Does

`nojs build` runs a single integrated pipeline that includes:

- **Expression compilation** — AOT-compiles directive expressions into pre-parsed AST cache and pre-compiled JavaScript functions. Annotates elements with `data-nojs-e` and injects `<script>` blocks before `</body>`.
- **Template compilation** — generates template descriptors and factory functions for `<template>` elements. Annotates templates with `data-nojs-desc` and injects `<script id="__nojs_factories">` before `</body>`.
- **Resource hints** — adds `<link rel="preload">` and `<link rel="preconnect">` for fetch directive URLs and route template sources. Skips interpolated URLs (`{...}` expressions).
- **Head attribute injection** — extracts static `page-*` directive values and injects `<title>`, `<meta>`, `<link rel="canonical">`, and JSON-LD into `<head>`.
- **Speculation rules** — generates a Speculation Rules API script from `<template route="...">` definitions for near-instant navigation.
- **Open Graph / Twitter meta** — generates OG and Twitter Card meta tags from `page-*` directives.
- **Sitemap generation** — generates `sitemap.xml` from route definitions and canonical URLs.
- **Image optimization** — adds lazy loading, LCP preload, and `fetchpriority` hints to `<img>` tags.

### Examples

```bash
nojs build                         # Compile and optimize all HTML in-place
nojs build --output dist/          # Write to dist/
nojs build --dry-run               # Preview changes
```

---

## `nojs validate`

Validate No.JS templates against 10 rules. CI-friendly with JSON output.

### Usage

```bash
nojs validate [glob] [options]
```

### Options

| Option | Description | Default |
|--------|-------------|---------|
| `--format <format>` | Output format: `pretty` or `json` | `pretty` |

### Rules

#### Errors (cause exit code 1)

| Rule | Triggers when |
|------|--------------|
| `fetch-missing-as` | `get`, `post`, `put`, `patch`, `delete` without `as` attribute |
| `each-missing-in` | `each="..."` without the `in` keyword |
| `foreach-missing-from` | `foreach="..."` without `from` attribute |
| `validate-outside-form` | `validate="..."` outside a `<form validate>` |
| `event-empty-handler` | `on:*` attribute with empty value |

#### Warnings (exit code 0)

| Rule | Triggers when |
|------|--------------|
| `model-non-form-element` | `model` on non-input/select/textarea elements |
| `bind-html-warning` | Any use of `bind-html` (reminder to trust content) |
| `route-without-route-view` | `<template route>` exists but no `route-view` element |
| `loop-missing-key` | `each` or `foreach` without `key` attribute |
| `duplicate-store-name` | Multiple `store="name"` with the same name |

### Output Formats

**Pretty** (default):
```
src/index.html
  x get="/users" is missing the "as" attribute [fetch-missing-as]
  ! each="item" without a "key" attribute [loop-missing-key]

1 error(s), 1 warning(s) in 1 file(s)
```

**JSON** (for CI):
```json
[{
  "filePath": "src/index.html",
  "issues": [{
    "rule": "fetch-missing-as",
    "severity": "error",
    "message": "..."
  }]
}]
```

### Exit Codes

- **0** — No errors (warnings may exist)
- **1** — One or more errors found

### Examples

```bash
nojs validate *.html                  # Validate HTML files
nojs validate src/ --format json      # JSON output for CI pipelines
nojs validate "**/*.html"             # Recursive glob
```

---

## `nojs.config.json`

Project configuration file generated by `nojs init`.

```json
{
  "name": "my-app",
  "version": "0.1.0"
}
```

| Key | Type | Description |
|-----|------|-------------|
| `name` | string | Project name |
| `version` | string | Project version (semver) |

---

## Typical Workflow

```bash
# 1. Scaffold a project
nojs init my-app --routing --i18n --locales en,pt -y
cd my-app

# 2. Develop with live reload
nojs serve --open

# 3. Validate templates before deploy
nojs validate *.html

# 4. Compile and optimize for production
nojs build --output dist/
```
