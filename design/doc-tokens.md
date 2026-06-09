# Documentation design tokens

Canonical, single source of truth for the **shared documentation style** across every
AI Agent Assembly doc surface (the central hub + each SDK/component doc site).

**Strategy (Path A):** every repo keeps its native doc generator (mdBook, MkDocs
Material, Docusaurus, Hugo/Hextra). Each generator maps the tokens below to its own
theming variables via the ready-made snippets in [`snippets/`](./snippets). Change a
value here → update each repo's snippet → all sites stay in sync.

**Source legend:**
- **(dashboard)** — taken verbatim from `agent-assembly/dashboard/src/styles.css`.
- **(dashboard, adj.)** — same hue as the dashboard, lightness tuned for dark-mode contrast.
- **(default)** — readability-tuned default (the dashboard token sheet defines only a light `:root`; dark neutrals are derived here).

## Brand accent

| Token | Light | Dark | Source |
|---|---|---|---|
| `accent` | `#6366f1` | `#818cf8` | dashboard `--accent` / (dashboard, adj.) |
| `accent-hover` | `#4f46e5` | `#a5b4fc` | dashboard `--accent-hover` / (dashboard, adj.) |
| `text-on-accent` | `#ffffff` | `#0e0e0e` | dashboard `--text-on-accent` / (default) |

Use the accent **only** for links, active nav, focus rings, and primary buttons — never for decoration.

## Neutrals

| Token | Light | Dark | Source |
|---|---|---|---|
| `bg` | `#f5f4f0` | `#0e0e0e` | dashboard `--paper` / `--ink` (adj.) |
| `surface` | `#ffffff` | `#1a1a1a` | dashboard `--paper-2` / (default) |
| `text` | `#111827` | `#ededed` | dashboard `--text-primary` / (default) |
| `text-muted` | `#6b7280` | `#a3a3a3` | dashboard `--text-muted` / (default) |
| `border` | `#e5e7eb` | `#2e2e2e` | dashboard `--surface-card-border` / (default) |

Backgrounds are intentionally **off**-white / **off**-black (not `#ffffff` / `#000000`) for reading comfort.

## Typography

| Token | Value | Source |
|---|---|---|
| body font | `Inter, ui-sans-serif, system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif` | dashboard base + extended tail |
| mono font | `"JetBrains Mono", ui-monospace, SFMono-Regular, "SF Mono", Menlo, Consolas, "Liberation Mono", monospace` | dashboard base + extended tail |
| base size | `16px` (mobile) → `17px` at ≥ 60rem | (default) |
| line-height (body) | `1.65` | (default) |
| **measure** (max line length) | `72ch` | (default) |
| h1 / h2 / h3 | `2rem`/700 · `1.5rem`/700 · `1.25rem`/600, **sentence case**, stop at h3 | (default) |

The capped measure (`72ch`) is the single biggest readability win — Docusaurus and Hextra default far wider.

## Admonitions (4 types only)

| Type | Edge/accent (light → dark) | Tint bg (light → dark) |
|---|---|---|
| `note` (info / blue) | `#1d3a7a` → `#6e94c2` | `#d6dfee` → `#16233f` |
| `tip` (ok / green) | `#22592a` → `#5fd17a` | `#d4e4d2` → `#15301a` |
| `warning` (amber) | `#8a5a00` → `#e0a92e` | `#f5e6c4` → `#3a2c0a` |
| `danger` (red) | `#b8291e` → `#f08a80` | `#f6dad6` → `#3a1512` |

All four light hues are the dashboard hi-fi semantic colors; dark variants are contrast-tuned.

## Dark mode

Default to **follow the OS preference** with a manual toggle on every site. mdBook (hub + core) uses
named themes — set `default-theme` to the branded light and `preferred-dark-theme` to the branded dark.

## Consuming this kit

| Repo | Generator | Snippet to copy |
|---|---|---|
| agent-assembly-docs (hub), agent-assembly (core) | mdBook | [`snippets/mdbook-additional.css`](./snippets/mdbook-additional.css) |
| python-sdk | MkDocs Material | [`snippets/mkdocs-extra.css`](./snippets/mkdocs-extra.css) + [`snippets/mkdocs-palette.yml`](./snippets/mkdocs-palette.yml) |
| node-sdk, inner-document | Docusaurus | [`snippets/docusaurus-custom.css`](./snippets/docusaurus-custom.css) |
| go-sdk | Hugo / Hextra | [`snippets/hextra-custom.css`](./snippets/hextra-custom.css) |

Brand assets (logo, favicon, social card) live in [`brand/`](./brand) — **placeholders**; replace with final artwork when available.
