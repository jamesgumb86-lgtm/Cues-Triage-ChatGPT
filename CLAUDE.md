# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A single-file front-end tool for **CUES Triage** (Community Urgent Eyecare Service). The entire
application lives in `index.html`: markup, all CSS (inline `<style>` block plus the Tailwind CDN
build), and all JS in one inline `<script>` at the bottom. There is no backend, no build system, no
package manager, and no test suite — this is a client-only, offline-capable page (no patient data
is transmitted or persisted anywhere).

`cues-triage.zip` is an archived copy of `index.html`. If you edit `index.html`, keep the zip in
sync (`rm -f cues-triage.zip && zip -q cues-triage.zip index.html`) rather than letting it drift.

**This is a staff/call-handler decision-support aid, not a patient-facing self-diagnosis tool.**
It digitizes the *CUES Eligibility Screening/Triage* script (source: "CUES 2022 V1.0",
20/12/2021) used by whoever takes the incoming call — hence fields like time-of-call, taken-by,
and consent-to-record-sharing. The source document is nearly five years old; the app displays a
banner telling staff to confirm it's still the current approved protocol before relying on it. If
a newer version of the source protocol is ever supplied, the branching logic in the `<script>`
block (see below) needs updating to match — don't extrapolate or invent clinical rules beyond what
a source document states.

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

There are no lint or test commands configured in this repo. To sanity-check changes:
- `node --check` against the extracted `<script>` contents catches JS syntax errors.
- For behavioral changes, drive the flow with Playwright against a local static server rather than
  trusting a read-through — the branching logic below has enough paths that manual review alone
  tends to miss a broken transition. Chromium is expected to already be available at
  `/opt/pw-browsers/chromium` in this environment (see repo-level tooling notes if not).

## Architecture

Everything is a single IIFE at the bottom of `index.html` driving a set of `<section class="step"
data-step="...">` elements inside `<main id="triage-flow">`. Exactly one `.step` has the `.active`
class at a time; `showStep(key)`/`goTo(key)` toggle it by matching `data-step`. There's no router —
navigation is just direct calls between named steps.

**Fixed steps**: `intake` → `consent` → `categories` → (variable category steps) → `summary`, plus
two side branches reachable from anywhere: `emergency` (self-declared emergency shortcut, opened by
the header's Emergency button; returns to `state.preEmergencyStep` on exit) and `consent-blocked`
(terminal — reached if the patient doesn't consent to record sharing, since the source document
states that blocks CUES access entirely).

**Variable category steps**: after `categories`, the user picks any combination of the four symptom
categories (`cl`, `foreign`, `vision`, `flashes`). `state.categoryQueue` is built in fixed order
(`['cl','foreign','vision','flashes']`, filtered to what was selected) and walked one at a time via
`advanceCategory()`, which pushes an `{ title, text, severity }` outcome and either shows the next
category's step or renders `summary`. Picking "None of these / not listed here" instead skips the
category queue entirely and pushes a single fallback outcome. When adding a new category, mirror
this pattern: add it to `CATEGORY_META`, add a `<section data-step="cat-X">`, and push outcomes
through `advanceCategory()` rather than writing a parallel code path.

**Per-category branching mirrors the source document's decision tree exactly** — each category's
continue-button handler encodes the specific yes/no questions and thresholds from the CUES script
(e.g. the `cl` category's self-care-vs-appointment split depends on pain/light-sensitivity/vision-
change count, self-referral status, age, and days-since-onset; `vision` fast-tracks straight to a
telemedicine outcome if field loss and sudden double vision co-occur, bypassing its own sub-
questions). When a branch outcome says "discuss with a CUES practitioner" rather than a concrete
action, that mirrors the source document's own ambiguity — don't resolve that ambiguity by
inventing a firmer rule; it wasn't in the source.

**Severity styling**: outcomes carry a `severity` of `danger` / `warning` / `info`, rendered via
`.outcome-card.severity-*` on the summary screen. `danger` is reserved for outcomes that mean
"escalate now" (hospital eye service, immediate telemedicine); `warning` for "not suitable" or
"needs a human decision"; `info` for a routine next action (arrange appointment, self-care).

**Progress bar**: `updateMainFlowProgress()` treats `intake`/`consent`/`categories` as fixed
quarters and interpolates the remaining quarter across however many category steps are queued —
recompute this if the number of fixed steps changes.

**Theming**: colors/spacing live in `:root` CSS custom properties (`--bg`, `--primary`, `--danger`,
etc.) with dark-mode overrides under `@media (prefers-color-scheme: dark)`. Component classes
(`.elevated`, `.btn*`, `.chip*`, `.option-card`, `.outcome-card`, `.field-error`) are hand-rolled and
layered *underneath* Tailwind utilities used directly in the markup. Note `<html class="dark">` is
hardcoded but has no effect — there's no Tailwind config setting `darkMode: 'class'`, so the CDN
build's dark-mode utilities (`dark:text-slate-100`, etc.) only respond to the OS-level
`prefers-color-scheme`, not that class.
