# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A personal collection of Java / Spring / DSA study notes. There is **no build system, no tests, no dependencies** — the artifacts are Markdown cheat sheets and standalone HTML study guides. Work here is writing and editing prose/reference content, not software.

## Structure

- **Markdown cheat sheets** (`Java_*.md`, `java_basics.md`) — the bulk of the content. Each is a self-contained topic (OOP, Classes, DSA, DSA_Patterns, Patterns, Spring_Glossary, AOP, etc.).
- **HTML study guides** (`two_pointers_pattern_guide.html`) — hand-authored, single-file pages with inline `<style>`/`<script>`, interactive demos. No framework, no bundler; open directly in a browser.
- **`index.html`** — the landing page linking to the HTML guides. When a new HTML guide is added, add a `guide-card` entry here (existing not-yet-written topics appear as `.soon` cards with a "Coming soon" badge).
- **`README.md`** — minimal index of Markdown files.

## Conventions to preserve

- Notes are written in a **narrative, "why it exists" voice**, not dry API reference (see the intros of `Java_DSA.md` / `Java_Patterns.md`). Match this tone when extending a file.
- Cheat sheets **cross-reference each other by filename** (e.g. `Java_Patterns.md` points to `Java_OOP.md` for OOP fundamentals). Keep these pointers accurate and avoid duplicating a topic that already has its own file.
- HTML guides share a common visual language (dark theme, `--accent: #c8f542`, DM Mono / Instrument Serif fonts, grid-background + glow-blob body decorations). Reuse these CSS variables/patterns for new guides so they match `index.html`.

## Viewing

Open any `.html` file directly in a browser — no server needed. Markdown files render on GitHub or any Markdown viewer.
