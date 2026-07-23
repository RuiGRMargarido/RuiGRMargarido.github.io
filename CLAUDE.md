# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Rui Margarido's personal portfolio site (`RuiGRMargarido.github.io`) — a static HTML/CSS/vanilla-JS single-page site, no framework. He is a Mechanical Engineer (BIL production manager since 04/2018) now studying Software Engineering at ISEP (since 2023, concurrent with the BIL job, not sequential) — that concurrency is the central "story" the site is built to communicate.

## Commands

```bash
npm install     # installs webpack tooling (only needed for local preview, see below)
npm start       # webpack-dev-server on http://localhost:8080, live-reload
npm run build   # webpack production build into dist/
```

There is no test suite (`npm test` is a placeholder that exits non-zero) and no lint config.

**Important:** GitHub Pages serves `index.html` and `styles.css` directly from the repository root of the `main` branch — there is no build step in the deployment path. `webpack`/`dist/` exist in the repo but are not what gets published; `npm start` is only a convenience for local preview (and is required for embedded YouTube videos to actually load, since they refuse to embed under a `file://` origin).

## Architecture

Everything lives in two files at the repo root: `index.html` and `styles.css`. All interactive behavior is inline `<script>` blocks at the bottom of `index.html` (no separate JS files, no build-time bundling of app logic — `js/app.js` is an empty leftover and unused).

Current section structure (`index.html`): `header#home` (hero) → `nav#navbar` (sticky) → `#about` → `#journey` (alternating vertical timeline, compact overview only) → `#experience` (full-depth mechanical engineering roles: BIL, LWC Metal, Jamarcol — each with logo, dates, description, skills, embedded videos) → `#projects` (full-depth software projects — each with description, GitHub/Bitbucket/PDF links, skills, embedded videos). Journey is a lightweight chronological overview; the real depth (paragraphs, skill lists, videos) lives in the two dedicated sections below it, not inline in the timeline.

Hero includes a `<canvas id="constellation-canvas">` particle-network animation (vanilla JS, no library) — respects `prefers-reduced-motion` and pauses its `requestAnimationFrame` loop via `IntersectionObserver` when scrolled out of view.

Design tokens live at the top of `styles.css` as CSS custom properties (`--paper`, `--ink`, `--copper` for the mechanical track, `--teal` for the software track, `--font-display` / `--font-body`). This is the "Constellation" design (see git history for prior rejected directions).

### Content completeness constraint

Every job, project, link (company site / GitHub / Bitbucket / PDF), embedded video, and skill tag currently in `index.html` is real content the site owner asked to keep — none of it is placeholder. When editing content, preserve all of it; this is a reorganization/update target, not something to trim for brevity. The About paragraph in particular should stay verbatim unless the owner explicitly asks to change the wording.

## Git workflow for this project

`main` is the live, published site — merging into it publishes immediately (no review/staging environment), so treat a merge to `main` as a deliberate publish action, not a routine integration step. The Constellation redesign and its content updates have already been merged (PRs #7 and #8); `main` is currently the only active branch, checked out directly in the primary folder (`RuiGRMargarido.github.io`).

For a future initiative that benefits from running in parallel without blocking other work (a new redesign direction, a large batch of content changes), reuse the pattern from that redesign: create a dedicated branch in its own **git worktree** — a separate sibling folder, e.g. `RuiGRMargarido.github.io - <branch-name>`, not just a branch switch in place — work there independently, and merge back into `main` once ready to publish.

Each active line of work should have an open GitHub PR kept as a long-lived tracking PR (not merged until the owner explicitly decides to publish) — commits keep landing on the same branch/PR rather than opening a new PR each time. Reference the relevant issue number in commit messages (e.g. `(#5)`) so GitHub cross-links the commit to its card; use `Closes #N` in the PR body (not in individual commits) so the issue auto-closes on merge.

Work is tracked on a GitHub Project board ("Portfolio Redesign", https://github.com/users/RuiGRMargarido/projects/5) with a `Track` field (Design / Content) and a `Status` field (Backlog / Todo / In Progress / Done). Prefer real Issues over draft cards so commits/PRs can link to them.

### GitHub Actions

`.github/workflows/auto-assign.yml` is the only workflow in this repo — it fires on `issues.opened` and `pull_request.opened` and assigns the new issue/PR to `RuiGRMargarido` via `actions/github-script`. There is no CI/build/deploy workflow: GitHub Pages publishes `main` directly (see Commands above), so nothing here runs tests or builds anything.
