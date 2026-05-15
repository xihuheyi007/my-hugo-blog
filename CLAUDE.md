# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

A Chinese-language Hugo blog ("静水流深") deployed at blog.baoyuchen.top, using the PaperMod theme via git submodule.

## Build & Development Commands

```bash
hugo server -D          # Local dev server with drafts
hugo server             # Local dev server (production-like)
hugo --minify           # Production build (output to public/)
hugo new posts/<name>.md  # Create new post from archetype
```

No package.json, Makefile, or CI config exists — builds are pure Hugo.

## Architecture

### Config (hugo.yml)
- Goldmark passthrough extension preserves LaTeX delimiters (`$...$`, `$$...$$`, `\[...\]`, `\(...\)`) through Hugo's markdown renderer so MathJax can process them client-side.
- `params.math: true` enables math globally; per-post front matter `math: true` also works.
- `unsafe: true` on Goldmark renderer allows raw HTML in markdown.

### Theme
- **PaperMod** theme as git submodule in `themes/PaperMod`. Do not edit theme files directly; override via `layouts/` and `assets/`.

### Custom Layouts

All custom layouts override PaperMod defaults:

| File | Purpose |
|------|---------|
| `layouts/index.html` | Homepage with profile card (avatar, title, social icons) followed by paginated article list. Replaces PaperMod's default homepage. |
| `layouts/partials/head.html` | Full override — sets up CSS bundles, favicons, RSS, theme-toggle JS (reads localStorage), search assets, and noscript fallback. Calls `extend_head.html` at the end. |
| `layouts/partials/header.html` | Nav bar with logo, theme-toggle button (3-state: auto→light→dark→auto), language switcher, and nav menu with inline SVG icons per menu item. |
| `layouts/partials/footer.html` | Copyright, Hugo/PaperMod credit, scroll-to-top button, smooth anchor scrolling, code-copy buttons, and the theme-toggle click handler (3-state cycle). |
| `layouts/partials/extend_head.html` | Conditionally loads MathJax 4 when `math` param is set. |
| `layouts/partials/math.html` | MathJax 4 CDN script with LaTeX delimiters and safe extension loaded. |
| `layouts/partials/toc.html` | Floating TOC sidebar with scroll-based active heading highlighting. Becomes sticky on wide screens via JS width calculation against CSS vars. |

### Custom CSS (assets/css/extended/)

| File | Purpose |
|------|---------|
| `fonts.css` | Loads SF Pro Text (Latin) and LXGW WenKai Screen (CJK) from `static/fonts/`. Body fallback: SFProText → LXGWWenKaiScreenR. |
| `toc.css` | Floating TOC positioning, sticky sidebar, active-link styling. Key vars: `--article-width: 650px`, `--toc-width: 230px`. |
| `math.css` | Prevents page-level horizontal scroll from wide formulas; makes display-math containers independently scrollable. |
| `profile.css` | Reduces homepage profile section padding. |
| `headings.css` | Custom sizes for h1 (26px), h2 (22px), h3 (18px) within post content. |
| `theme-toggle.css` | Controls visibility of sun/moon/auto icons based on `data-theme` and `data-theme-mode` attributes. |
| `nav-icons.css` | Vertical alignment and spacing for SVG icons in nav menu items. |

### Custom Archetype

`archetypes/default.md` generates a minimal front matter template with `date`, `draft: false`, and a title derived from the filename.

### Content
- All posts in `content/posts/` as flat `.md` files.
- `content/posts/archives.md` uses `layout: "archives"` for the timeline page at `/archives/`.
- Front matter: `title`, `date`, `draft`, `categories`, `tags`, and optionally `math: true`.

### Static Assets
- `static/fonts/` — two woff2 font files (SF Pro Text, LXGW WenKai Screen)
- `static/favicon.ico`

## Key Conventions

- Posts are written in Chinese with English technical terms.
- Math expressions use LaTeX syntax (MathJax 4 rendering). Both Hugo's Goldmark passthrough and MathJax handle delimiters — Goldmark preserves them, MathJax renders them.
- `hasCJKLanguage: true` — Hugo correctly handles CJK word count and summary truncation.
- `buildDrafts: false`, `buildFuture: false`, `buildExpired: false` — only published, non-expired content appears in production.
- Pagination is 10 posts per page.
- The `public/` directory is committed to git (likely for direct deployment).
- Theme toggle has 3 states cycling auto → light → dark → auto, persisted in localStorage key `pref-theme`.
