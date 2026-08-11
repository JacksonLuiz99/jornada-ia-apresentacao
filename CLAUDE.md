# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A self-contained HTML presentation split across `index.html` (markup + JS) and `styles.css` (all CSS) — a scrollytelling slide deck in Portuguese titled "Minha jornada com IA no desenvolvimento" (the author's personal journey adopting AI tools in software development, framed as a `git log`). It is not a software application: there is no package manager, build step, linter, or test suite.

There is no git repository here — this is a bare working directory.

## Working with the file

- Open `index.html` directly in a browser to view/preview it (double-click, `xdg-open index.html`, or serve the directory with `python3 -m http.server` and hit `/index.html` — useful when driving the page with browser automation, since `file://` tabs can behave differently from `http://` ones). `styles.css` is loaded via a relative `<link>`, so keep the two files in the same directory. The page loads Google Fonts (JetBrains Mono, IBM Plex Sans) from a CDN, so it needs internet access to render with the intended fonts. **Watch for browser HTTP caching** when iterating: reloading after an edit can silently serve the previous version (`ETag`/`Last-Modified`) — append a cache-busting query string (`?v=2`) or hard-reload if a change doesn't seem to show up.
- The deck has **no image carousel/evidence gallery anymore** — it was removed (screenshots were inconsistent sizes and looked rough projected). `assets/1.png` … `10.png` (renamed from `Bug dos anexos/`) still exist on disk but are no longer referenced by the HTML at all; they're just leftover raw screenshots, not wired into the page. Don't reintroduce a base64-embedded image carousel without checking with the presenter first — that's what was there before and got pulled.
- **The reveal-on-scroll system requires the literal class `reveal`** on any element that should fade/slide into view (the CSS sets `opacity:0` unconditionally on several component classes — e.g. `.carousel` before it was removed — and only `.reveal.in` brings it back to `opacity:1`). If you add a new visual block copying an existing component's CSS pattern (opacity:0 + transform, revealed via IntersectionObserver), remember to add `class="... reveal"` on the actual element in the markup — forgetting it silently renders the block invisible with no console error, which is exactly the bug that prompted this note (the old carousel had this bug from the start).

## Architecture

Three parts, split across two files:

1. **`styles.css`, linked from `<head>`** — CSS custom properties define the theme (dark by default, pink/violet accents — the "rosa" identity, see below — monospace/sans fonts), with a full light-theme override block. Key structural classes:
   - `.rail` / `.rail-dots` / `.rail-fill` — the fixed left-hand "commit" progress rail with clickable dots, one per slide.
   - `.theme-toggle` — the rosa/light switch fixed near the top-left.
   - `section[data-slide]` — each slide/"commit" is a full-height `<section>`; `.eyebrow` renders a fake commit hash + conventional-commit-style type (e.g. `feat(fluxo): era do copiar-e-colar`).
   - `.reveal` — elements that fade/slide in on scroll (driven by `IntersectionObserver` in the script) — see the caching/reveal note above.
   - `.pipeline` / `.pipe-step` — the numbered roadmap list used by the "Visão geral" overview slide.
   - `.continue-wrap` / `.cursor` — the typewriter-effect closing slide.

2. **Body markup** — a `<main>` containing an ordered sequence of `<section>` elements: `#hero` (intro), a "Visão geral" roadmap slide (`.pipeline`/`.pipe-step`), ~17 `data-slide` sections narrating the journey (ChatGPT → Antigravity → VS Code+Claude → adopting Spec-Driven Development and its 7 steps → tooling tried and dropped → mistakes/lessons learned), and `#closing`. Slide order in the DOM is the presentation order — inserting/removing a `<section data-slide>` automatically updates the rail dot count and the `commit NN/total` HUD counter (computed dynamically in JS, not hardcoded; the static `01/18` in the raw HTML is just a pre-JS fallback — update it to match `document.querySelectorAll('[data-slide]').length + 1` if you add/remove slides, though it's cosmetic only, overwritten on first scroll-tick).

3. **`<script>` (end of body), plain JS, no framework/build tooling** — four independent behaviors:
   - **Theme toggle**: `#themeToggle` flips `data-theme="light"`/absent on `<html>` (absent = the default "rosa" dark theme), persisted to `localStorage['jornada-ia-theme']`. A tiny inline script in `<head>` (before the stylesheet `<link>`) reads that key synchronously to avoid a flash of the wrong theme on load. Light-theme variable overrides live in `:root[data-theme="light"]{...}` right after the base `:root{...}` block — if you add a new color, define it in both places, and re-check contrast (see below) since rosa/light need very different lightness values for the same semantic color.
   - **Reveal-on-scroll**: one `IntersectionObserver` watches all `.reveal` elements and staggers `.in` class additions per-slide (90 ms stagger) to trigger the CSS fade/slide transition.
   - **Git rail / HUD**: on scroll, computes scroll percentage for `.rail-fill` height and which section is "active" to highlight the corresponding rail dot and update `#hud` text.
   - **Closing typing animation**: triggered once via `IntersectionObserver` on `#closing`, types out "Continue..." character-by-character.

## Color contrast

The light theme's `--amber`/`--ok`/`--err`/`--ink-faint` values were deliberately chosen (not eyeballed) to clear WCAG AA (≥4.5:1 for the small mono text they're used on — eyebrow labels, inline `<code>`, chips, banner labels — checked against the *worst-case* background, `--panel-2`, since it's the darkest of the three light surfaces). The original guesses for `--amber` (#b8790a, ~3.2:1) and `--ink-faint` (#97a0b5, ~2.3:1) failed that check. If you touch either theme's palette, recompute contrast (relative-luminance WCAG formula) for every `--ink*`/`--amber`/`--violet`/`--ok`/`--err` pair against every background it's actually used on in that theme — don't eyeball it, this file has already regressed once from an unchecked guess.

The default (dark) theme's `--void`/`--panel-2`/`--line` are pixel-sampled as-is from the reference PDF (see below), but `--violet` (#a48be2) and `--err` (#f66670) are nudged lighter than their sampled source (#9a81df / #f2545b) — the raw sampled violet only cleared 4.46:1 against `--panel-2` (#2f1d63), just under the 4.5:1 AA floor, since that panel color is far more saturated than the light theme's `--panel-2`. Same rule applies here: don't reuse the raw sampled hex for a foreground/text role without rechecking contrast against `--panel-2`.

## Content notes

- Copy is written in Brazilian Portuguese and structured as a fake `git log --graph`: each slide's `.eyebrow` has a short hash + conventional-commit type/scope, reinforcing the narrative device.
- Presenter delivers this live over a Discord screen share — the deck has no export/PDF path and isn't meant to need one.
- The "Passo 1–7" slides narrate a real Spec-Driven-Development pipeline (SDD) that is documented in `docs/.ai/` (see below) — when correcting or extending that part of the story, cross-check against those files rather than guessing command/agent names, since the deck is meant to describe the actual workflow accurately.

## `docs/.ai/` — the real SDD pipeline the deck narrates

This directory is the actual source of truth for the tooling described in the "Passo 1" through "Passo 7" slides — it is a separate, tool-agnostic set of markdown artifacts (rules/skills/agents/workflows) that, in the author's real Angular project, gets projected into `.claude/` via `npm run setup:ai`. It documents the author's real project workflow (an Angular app using an internal `@pjc/*` component kit), not something invented for the presentation — it is reference material copied in here, not this repo's own tooling. Key facts worth knowing before editing that part of the deck:

- **Phase 1 — spec**: `/sdd <ideia>` (not a subagent — `docs/.ai/agents/pm-spec-architect.md` is explicitly a *persona* the main chat assumes, because Phase 1 needs a real back-and-forth conversation with the human). Produces `.sdd/specs/<slug>.spec.md`; stops for human approval.
- **Phase 2 — tasks**: `/gerar-tasks <spec>` delegates to the `dev-lead` subagent, which decomposes the approved spec into per-file tasks under `.sdd/tasks/<slug>-v<versão>/`, each tagged `Nível: Pleno` or `Nível: Sênior`. Stops for human review of the task list.
- **Phase 3 — implementation**: `/implementar-tasks <readme> <spec>` (the author also uses a personal `/implementar-tasks-v2` variant on some projects, not present in this `docs/.ai/`) orchestrates dispatch — each task's `Nível` routes it to the `dev-pleno` subagent (copies an existing pattern) or `dev-senior` subagent (defines a new pattern); independent tasks (empty `Depende de:`) run in parallel. Gate is `npm run check` (lint + test + build) — that's what the `dev-lead`/`dev-pleno`/`dev-senior` agent files and `revisar-dod.md` state; `workflows/implementar-tasks.md` itself only mentions lint + build, a small inconsistency in the source material rather than something to "fix" in the deck.
- **Git/commit rules** (`docs/.ai/rules/git-workflow.md`, `docs/.ai/rules/commit-and-test.md`): never commit/branch/push without an explicit request in that interaction; commit messages are objective and explain *why*, not a file-by-file list; never add an AI co-author line (`Co-Authored-By: Claude...`) to a commit message. Note the naming is a little misleading — `commit-and-test.md`'s actual content is just the no-AI-signature rule, not a "tests must pass" gate (that gate is `npm run check`, enforced separately by the implementer agents' pre-completion checklists). The deck resolves this by keeping "tests must pass" grounded in `npm run check` (Passo 6) and the no-signature/no-autonomous-commit rules under `git-workflow` (Passo 7) — don't merge them back under a single "commit-and-test" label.

## Reference deck

`docs/Como-utilizo-IA-no-meu-fluxo-de-desenvolvimento.pdf` is an earlier/parallel 10-slide Gamma-made version of the same story (dark theme, pink `#ff8aaf`/violet-blue `#1f56d2` accents on a `#0c0524` background), used as a source to cross-check and enrich this HTML deck (e.g. the "Visão geral" pipeline slide, "vibe coding", and the SIGEP test-flow example were absorbed from it; its sampled colors were also used as a WCAG-contrast benchmark when fixing the light theme, above — its own palette sits at 8.9:1/14:1 for headings/body, which is what the fix targets). Treat it as reference material, not something to sync automatically, since some of its details (e.g. it names `/pm-spec-architect` as if it were the literal slash command, and frames `commit-and-test` as "tests must pass") are simplifications that this HTML deck deliberately corrects against `docs/.ai/`.

Color history (2026-08-11, same day, two decisions in sequence — don't resurface the first one as if it still held): the deck originally kept an amber/violet identity (`--amber:#f0b429`, `--violet:#7c83fd`) deliberately *not* matching this PDF, confirmed with the presenter. Later the same day the presenter reversed that: the PDF's pink/violet palette was pixel-sampled (via `pdftoppm` + color clustering, not eyeballed) and first added as an opt-in third theme, then promoted to replace amber/violet as the **default** dark theme outright — the old amber/violet hexes no longer appear anywhere in `styles.css`. The toggle is binary again (`data-theme="light"`/absent, absent = rosa), same mechanism as before, just with the default palette swapped: `--void:#0c0524`, `--panel-2:#2f1d63`, `--line:#48367c`, `--amber:#ff8aaf`, `--violet:#a48be2` (contrast-adjusted, see above). The light theme was untouched throughout.
