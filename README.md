# NoJS Skill

AI skill that gives [Claude Code](https://claude.com/claude-code) and compatible AI tools expert-level knowledge of the **[No.JS](https://no-js.dev)** framework — the HTML-first reactive framework for building dynamic web applications using only HTML attributes.

## What it does

When installed, this skill automatically activates whenever you work with No.JS directives. Your AI assistant will:

- **Know all 39+ directives** with correct priorities, syntax, and companion attributes
- **Generate valid No.JS templates** following best practices (scoped state, keyed loops, proper filters)
- **Validate templates** for common mistakes (missing `as`, wrong event syntax, unsanitized `bind-html`)
- **Explain any directive** with working examples
- **Apply 32 built-in filters** correctly via pipe syntax
- **Use the full public API** (config, router, i18n, stores, interceptors)

## Installation

### Claude Code

```bash
claude skill install github:ErickXavier/nojs-skill
```

### Manual

Copy `SKILL.md` and the `references/` directory into your skills directory.

## Files

| File | Purpose |
| --- | --- |
| `SKILL.md` | Main skill file — framework overview, directive priorities, expression syntax, API, rules |
| `references/directives.md` | Complete directive reference with all attributes and examples |
| `references/filters.md` | All 32 built-in filters with syntax and arguments |
| `references/api.md` | Full JavaScript API reference |
| `references/patterns.md` | Common patterns, scaffolds, and best practices |
| `references/validation.md` | Template validation rules and common mistakes |

## Activation

The skill activates when it detects:

- Mentions of **No.JS**, **NoJS**, or **HTML-first framework**
- HTML attributes matching No.JS directives (`bind`, `state`, `get`, `each`, `on:click`, `model`, `route`, `store`, `computed`, `watch`, `if`, `show`, `foreach`, `validate`, `animate`, `drag`, `drop`, `t`, `class-*`, `style-*`, `bind-*`)
- Questions about **declarative HTML frameworks** or building web apps **without JavaScript**

## Ecosystem

| Project | Description |
| --- | --- |
| [No.JS](https://github.com/ErickXavier/no-js) | Core framework (~24KB gzipped, zero dependencies) |
| [NoJS LSP](https://github.com/ErickXavier/nojs-lsp) | VS Code extension — completions, diagnostics, hover docs |
| [NoJS MCP](https://github.com/ErickXavier/nojs-mcp) | MCP server for AI assistants |
| **NoJS Skill** | This project — AI skill for Claude Code |

## Version

This skill documents **No.JS v1.10.0**. The framework source code is the ground truth — see [CONTRIBUTING.md](CONTRIBUTING.md) for how to keep the skill in sync.

## License

[MIT](LICENSE)
