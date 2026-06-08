# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A pedagogical companion for a French-language introductory course on LLMs ("Introduction aux LLMs").
The deliverable is a single static file, `index.html`, that bundles five interactive widgets into one
tabbed app. There is no build step, package manager, framework, or test suite — open `index.html` in a
browser, or visit the GitHub Pages deployment.

- Live site: https://inoceram-jb.github.io/intro-llm-widgets/
- Repo: https://github.com/Inoceram-jb/intro-llm-widgets (branch `main`)
- Deploy: GitHub Pages serves `index.html` from `main`. Push to `main` to publish; the build takes ~10–15s.

## File roles

- `index.html` — the shipped artifact. Self-contained, no external assets except Chart.js (CDN), used
  by the Analogies and Rétropropagation tabs. **This is the file that gets deployed.**
- `1 - bigram_candide_llm.html` … `5 - backpropagation_interactive.html` — the original standalone
  source widgets. All five are consolidated into `index.html`. Treat these as the editable originals /
  staging ground; when you change a widget, keep the matching source file and the `index.html` template
  copy in sync (they are duplicated by design).
- `Introduction aux LLMs.md` — the course script. It maps each widget to the exact pedagogical moment
  (with tab names) where it should be shown. Consult it to know what a widget is meant to teach.
- `README.md` — French user-facing summary of the four tabs.

## Architecture of `index.html`

The defining decision is **iframe isolation**: each widget was authored independently and they share
generic CSS class names (`.c-purple`, `.c-teal`, etc.) and global JS, so rendering them in the same
document would collide. Instead:

1. Each widget's full HTML lives in a `<template data-widget="N">` element (N = 0..4) in the host page.
2. A single runtime IIFE at the bottom of the file (search `var TABS`) builds the tab bar from the
   `TABS` array, then lazily instantiates one `<iframe>` per tab via `ensureFrame(i)`, setting
   `f.srcdoc = buildDoc(template.innerHTML)`.
3. `buildDoc()` wraps each widget fragment with three injected blocks before writing it into the iframe:
   - `THEME` — the entire design system reconstructed as inline `<style>` (CSS custom properties like
     `--color-text-primary`, plus the `.c-gray/.c-purple/.c-teal/.c-coral/.c-amber` SVG color classes).
     Every widget depends on these tokens; if you add a widget, its colors come from here.
   - `SHIMS` — stubs for the Claude Desktop host (`sendPrompt`, `completion`) and a `matchMedia`
     override that forces light mode (`prefers-color-scheme` always returns no-match).
   - `RESIZE` — posts the document height to the parent via `postMessage({__wh:true,h})`. The host
     listens for these messages and sets the matching iframe's height. This is how iframes auto-size;
     iframes have `scrolling="no"`.

Implications when editing:
- To change a widget's behavior, edit inside its `<template data-widget="N">` block.
- To add a tab, add the next `<template data-widget="N">`, add an entry to the `TABS` array (order and
  count must match the templates), and rely on `THEME` for styling (or extend `THEME`). The runtime
  wires everything else automatically.
- A widget's JS runs in its own iframe `window` — there is no shared state between tabs by design.
- Strings like `'<scr'+'ipt>'` are deliberately split so the injected scripts don't terminate the host
  `<script>` early. Keep that pattern when adding injected code.

## Widget-specific notes

- **Tab 1 (Bigramme)** trains a bigram model live in the browser. It supports multiple corpora via the
  `CORPORA` registry and `loadCorpus(key)`; corpus texts (Candide FR, Dr Jekyll & Mr Hyde EN) are
  embedded as template literals inside the widget. Corpus text comes from public-domain Project
  Gutenberg sources, normalized/sanitized for safe inline embedding.
- **Tab 3 (Analogies)** and **Tab 5 (Rétropropagation)** load Chart.js from a CDN (each via a
  `<script src>` inside its own template fragment, so each iframe loads its own copy) — they need
  network access; the other tabs work fully offline. Tab 5's charts are built lazily when its step 5/6
  pills are clicked.
- Widget SVGs use a fixed-token layout (`TOKEN_W`, `GAP`, `tokenX()` helpers). Watch the viewBox math:
  too many tokens at a fixed width overflows the SVG width and clips text.

## Conventions

- All user-facing text is **French**. Match that when editing UI copy, comments, and labels.
- Keep `index.html` self-contained and dependency-free (Chart.js CDN is the only sanctioned exception).
