# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A single-file front-end prototype for **CUES Triage** (Community Urgent Eyecare Service) — a
patient-intake/triage tool. The entire application currently lives in `index.html`: markup, all
CSS (inline `<style>` block plus the Tailwind CDN build), and a `<script>` placeholder with no
logic wired up yet. There is no backend, no build system, no package manager, and no test suite.

`cues-triage.zip` is an archived copy of `index.html` (verified byte-identical to the repo root
file). If you edit `index.html`, keep `cues-triage.zip` in sync (re-zip it) rather than letting it
drift, since it appears to be the canonical distributable snapshot.

## Running / developing

There is no build step. Open `index.html` directly in a browser, or serve the directory with any
static file server, e.g.:

```bash
python3 -m http.server 8000
```

Styling is powered by the Tailwind CDN script (`<script src="https://cdn.tailwindcss.com">`), so a
network connection is required for Tailwind utility classes to be generated at load time. There is
no `tailwind.config.js` — any Tailwind customization must be done via an inline config script tag
if ever needed.

There are no lint or test commands configured in this repo.

## Architecture / conventions worth knowing

- **Single-page, single-step so far**: `index.html` currently renders only the "Patient details"
  step of the triage flow (name, DOB, phone, symptom onset) inside `<main id="triage-flow">`. The
  two action buttons ("Start emergency triage" / "Proceed to symptom selection") and the header's
  "Start over" button are not yet wired to any behavior — the `<script>` block at the bottom is an
  empty placeholder. Expect to build out additional triage steps/screens inside `#triage-flow` and
  drive them from that script block.
- **Progress bar hook**: `#progress-bar` (`.progress-fill`) is present in the header but not yet
  updated by any JS — width is hardcoded to `0%` in CSS. Wire this up when adding multi-step
  navigation.
- **Theming via CSS variables**: colors/spacing live in `:root` custom properties (`--bg`,
  `--primary`, `--text`, `--radius`, etc.) defined in the `<style>` block, with a second set of
  overrides under `@media (prefers-color-scheme: dark)`. Component classes (`.elevated`, `.btn`,
  `.btn-primary`, `.btn-quiet`, `.chip`, `.appbar`, `.progress-track`/`.progress-fill`) are custom,
  hand-rolled classes layered *underneath* Tailwind utility classes used directly in the markup —
  don't assume everything is a Tailwind utility; check the `<style>` block first.
- **Dark mode quirk**: `<html class="dark">` is hardcoded, but dark-mode styling actually applies
  via the `prefers-color-scheme: dark` media query (both in the custom CSS variables and via
  Tailwind's `dark:` utility variants, e.g. `dark:text-slate-100`). Since there's no Tailwind config
  setting `darkMode: 'class'`, the CDN build defaults to media-strategy dark mode — the `class="dark"`
  attribute on `<html>` has no effect on Tailwind's `dark:` variants. Don't assume toggling that
  class will switch themes; it won't under the current Tailwind CDN setup.
