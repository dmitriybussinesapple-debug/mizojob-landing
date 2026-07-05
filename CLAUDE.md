# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A single-page static landing site for **MizoJob**, a Telegram job-search bot (`@MizoJobRoBot`). Primary language of the content is Russian. There is no build system, package manager, linter, or test suite — the entire site is one hand-written `index.html` with inline CSS and vanilla JS, plus two video assets.

## Development

There are no build or install steps. To preview locally, serve the directory with any static server (opening `index.html` directly via `file://` also works, but videos autoplay more reliably over HTTP):

```bash
python3 -m http.server 8000
```

## Deployment

Pushing to `main` deploys the repository root to GitHub Pages via `.github/workflows/deploy.yml` (also triggerable via `workflow_dispatch`). Everything in the repo is published as-is — keep unrelated files out of `main`.

## Architecture of index.html

The file is organized into three blocks, each with `/* ============ */` banner comments marking sections. Keep edits within the matching banner section for the piece you're changing — CSS, markup, and JS for a given section are intentionally labeled with the same name (e.g. "SECTION 1 · VACANCIES").

1. **Inline `<style>`** — design tokens as CSS custom properties in `:root` (coral/peach palette, radii, `--ease`), then per-section styles. Fonts are Manrope (body) and Space Grotesk (headings) from Google Fonts.
2. **Markup** — five sections in order: hero (fullscreen `video2.mp4` background), Section 1 "vacancies" (fan of glass cards), Section 2 "stats" (count-up plates), bottom CTA video block (`video1.mp4`), footer.
3. **Inline `<script>`** (single IIFE at the end of `<body>`) — all behavior lives here.

### Scroll-scrub animation system

The two middle sections are **scroll-scrubbed**, not time-based. The pattern, shared via the `makeScene(track, apply)` helper in the script:

- Each section has a tall `__track` element (`160vh` / `150vh`) containing a `position: sticky; height: 100vh` viewport.
- `makeScene` maps the track's scroll position to a progress value 0..1 and calls `apply(p)` per frame. The applied value eases toward the scroll target (factor `0.085` in the rAF loop), so fast flings play out with lag while slow scrolling feels 1:1.
- Section 1: five `.fc` cards fly in from off-screen into a fan while the headline shrinks; per-card start/end transforms live in the `cfg` object keyed by class name (`fc--1`…`fc--5`), as `[startX vw, startY vh, finalX px, finalY px, finalRot deg]`.
- Section 2: stat rows collapse upward into a column and their numbers count up with scroll; per-row config is `[startY vh, delay, countTarget]`. The stat numbers also appear as `data-target` attributes in the markup — keep the JS `cfg` values and the markup in sync when changing stats.

To tune these animations, edit the `cfg` arrays and `SPAN` constants in the script, not the CSS — JS writes `transform`/`opacity` inline and overrides stylesheet values.

### Conventions and constraints

- **Reduced motion is a hard requirement**: everything checks `prefers-reduced-motion`. `makeScene` calls `apply(1)` to jump scenes to their final frame, reveals render instantly, and videos are paused. Any new animation must follow this pattern.
- Videos have a fallback path (`guardVideo`): if a video errors or hasn't loaded within 2.5s, an animated-gradient fallback layer is shown (currently wired for the bottom CTA video only). The hero `poster="video2-poster.jpg"` references a file that does not exist in the repo.
- The RU/EN language switcher in the nav is **visual only** — it toggles the active button but does not translate anything.
- Scroll reveals use the `.reveal` class + `IntersectionObserver`, with `data-delay="1|2|3"` for stagger.
- All CTAs point to `https://t.me/MizoJobRoBot`.
