# lrn

Personal learning notes — Java, Spring, and DSA — plus interactive study guides.
Documented for future review, published as a web site via GitHub Pages.

**Site:** https://eslamelshaabany.github.io/lrn/

## Structure

```
index.html        # site entry point (hub → guides + notes)
guides/           # hand-authored interactive HTML study guides
notes/            # markdown study notes (Jekyll-rendered on the site)
  java/           #   Java & Spring
  dsa/            #   data structures & algorithms
_layouts/         # Jekyll layout for note pages
assets/           # shared CSS for note pages
_config.yml       # Jekyll config
```

## Contents

- **Notes** — [notes/](notes/) ([Java](notes/java/), [DSA](notes/dsa/))
- **Guides** — [Two Pointers](guides/two-pointers.html) ·
  [Linked List](guides/linked-list.html) ·
  [Sliding Window](guides/sliding-window.html)

Notes are plain Markdown: read them on the site, on GitHub, or in Obsidian.

## Local preview

```sh
bundle install
bundle exec jekyll serve   # → http://localhost:4000
```
