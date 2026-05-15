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

### Config
- **hugo.yml** — single config file (not TOML). All site params, menus, and markup settings live here.
- Math rendering is toggled globally via `params.math: true` and per-post via front matter `math: true`.

### Theme
- **PaperMod** theme as git submodule in `themes/PaperMod`. Do not edit theme files directly; override via `layouts/` and `assets/`.

### Custom Layouts (layouts/partials/)
- **extend_head.html** — conditionally loads MathJax 4 when `math` param is set
- **math.html** — MathJax 4 CDN script with LaTeX delimiters (`$...$` inline, `$$...$$` block, `\[...\]` display)
- **toc.html** — custom floating TOC sidebar with scroll-based active heading highlighting. Becomes sticky on wide screens (> article-width + toc-width + gaps)

### Custom CSS (assets/css/extended/)
- **fonts.css** — loads SF Pro Text (Latin) and LXGW WenKai Screen (CJK) from `static/fonts/`. Body font fallback chain: SFProText → LXGWWenKaiScreenR.
- **toc.css** — floating TOC positioning, sticky sidebar, active-link styling. Key CSS vars: `--article-width: 650px`, `--toc-width: 230px`.

### Content
- All posts in `content/posts/` as flat `.md` files (no subdirectories except generated page bundles).
- **archives.md** uses `layout: "archives"` for the timeline page at `/archives/`.
- Front matter uses `title`, `date`, `draft`, `categories`, `tags`, and optionally `math: true`.

### Static Assets
- `static/fonts/` — two woff2 font files (SF Pro Text, LXGW WenKai Screen)
- `static/favicon.ico`

## Key Conventions

- Posts are written in Chinese with English technical terms.
- Math expressions use LaTeX syntax (MathJax 4 rendering).
- `hasCJKLanguage: true` is set — Hugo correctly handles CJK word count and summary truncation.
- `buildDrafts: false`, `buildFuture: false`, `buildExpired: false` — only published, non-expired content appears in production.
- Pagination is 10 posts per page.
- The `public/` directory is committed to git (likely for direct deployment).
