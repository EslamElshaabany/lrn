# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal collection of Java / Spring / DSA study notes, published as a web site via GitHub Pages. The artifacts are Markdown notes (rendered by Jekyll) and standalone interactive HTML study guides. There are **no application sources and no tests** — work here is writing and editing prose/reference content, plus the light Jekyll/CSS scaffolding that presents it. The only "build" is Jekyll (for local preview); GitHub Pages builds it on push.

## Structure

- **`notes/`** — Markdown study notes, grouped by topic: `notes/java/` (basics, classes, oop, design-patterns, aop, spring-glossary, …) and `notes/dsa/` (data-structures, patterns). `notes/index.md` is the notes landing page. These are the bulk of the content.
- **`guides/`** — hand-authored interactive HTML study guides (`two-pointers.html`, `linked-list.html`, `sliding-window.html`) — single-file pages with inline `<style>`/`<script>` and interactive demos. No framework, no bundler.
- **`index.html`** — the site entry point / hub, linking to both the guides and the notes. When a new HTML guide is added, add a `guide-card` entry here (not-yet-written topics appear as `.soon` cards with a "Coming soon" badge).
- **Jekyll scaffolding** — `_config.yml`, `_layouts/note.html` (dark-theme wrapper for note pages), `assets/notes.css` (shared note styles). `Gemfile` pins the `github-pages` gem for local preview.
- **`README.md`** — human-facing index + how to preview locally.

## Conventions to preserve

- Notes are written in a **narrative, "why it exists" voice**, not dry API reference (see the intros of `notes/dsa/data-structures.md` / `notes/java/design-patterns.md`). Match this tone when extending a file.
- **Every note needs YAML front matter** (`title`, `topic`) — Jekyll only renders a file through the `note` layout if it has front matter; without it the raw Markdown is served. The `note` layout draws the title/eyebrow from front matter, so notes do **not** start with an `# H1` (the blockquote intro comes first).
- Notes **cross-reference each other with relative Markdown links** (e.g. `notes/java/aop.md` links to `[oop.md](oop.md)`, cross-topic uses `../dsa/patterns.md`). GitHub Pages' `jekyll-relative-links` makes these resolve on the site, and they also work in Obsidian/GitHub. Keep pointers accurate; don't duplicate a topic that already has its own file.
- The site shares a common visual language (dark theme, `--accent: #c8f542`, DM Mono / Instrument Serif fonts, grid-background + glow-blob decorations), defined in `index.html` and `assets/notes.css`. Reuse these variables/patterns for new guides and pages.

## Viewing

- **Notes / the full site:** `bundle exec jekyll serve` → http://localhost:4000 (or just read the Markdown on GitHub / in Obsidian).
- **Guides:** the `.html` files in `guides/` also open directly in a browser — no server needed.
