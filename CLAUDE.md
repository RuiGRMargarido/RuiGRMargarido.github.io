# CLAUDE.md

Guidance for coding agents working in this repository.

This is the single source of truth. `AGENTS.md` is a pointer to this file, so that Claude Code and Codex share one set of instructions instead of two copies that drift apart. Keep edits here.

## What this is

Rui Margarido's personal portfolio site (`RuiGRMargarido.github.io`): a static HTML/CSS/vanilla-JS single-page site, no framework.

He is a Mechanical Engineer who managed the mesh factory at Baptista e Irmão SA for seven years (04/2018 to 08/2025) **while** taking a Software Engineering degree at ISEP (2023, concluded 2026). The two tracks ran in parallel, not in sequence, and that concurrency is the central story the site is built to communicate. The Journey timeline exists specifically to make it visible.

## Commands

```bash
npm install     # installs webpack tooling (only needed for local preview, see below)
npm start       # webpack-dev-server on http://localhost:8080, live-reload
npm run build   # webpack production build into dist/
```

There is no test suite (`npm test` is a placeholder that exits non-zero) and no lint config.

**Important:** GitHub Pages serves `index.html` and `styles.css` directly from the repository root of the `main` branch. There is no build step in the deployment path. `webpack`/`dist/` exist in the repo but are not what gets published; `npm start` is only a convenience for local preview, and is required for embedded YouTube videos to actually load, since they refuse to embed under a `file://` origin.

A plain static server (`python -m http.server <port>`) is enough for everything except the YouTube embeds, and is the quicker option when checking layout.

## Architecture

Everything lives in two files at the repo root: `index.html` and `styles.css`. All interactive behavior is in inline `<script>` blocks at the bottom of `index.html` (no separate JS files, no build-time bundling of app logic; `js/app.js` is an empty leftover and unused).

Section order in `index.html`:

`header#home` (hero) → `nav#navbar` (sticky) → `#about` → `#journey` → `#projects` → `#experience` → `footer`

- **`#journey`** is an alternating vertical timeline, newest first, under a `Now` marker. Each `li.timeline-item` carries `soft` (software track) or `mech` (mechanical track) plus `left`/`right`, and **the `left`/`right` classes must strictly alternate down the list**. Inserting an entry in the middle means flipping every entry below it; inserting at the top only works without flipping if the new entry takes the opposite side of the current first one. Ongoing entries get `.ongoing` on the `li` and an `.ongoing-pill` next to the date. Cards deep-link into `#projects` / `#experience`.
- **`#projects`** has two parts: `.projects-index`, a compact scannable grid of `.index-card` links (title, one-line teaser, 2 to 3 skill chips plus a `+N` counter), and below it `.projects-deep-dive`, the full case-study `article.project-item` cards in the same order. **The `+N` counter must equal the deep-dive skill count minus the chips shown**, so it needs updating whenever skills change. This split replaced a manual two-column pairing that left uneven gaps, because card heights vary a lot.
- **`#experience`** holds the full-depth mechanical engineering roles (BIL, LWC Metal, Jamarcol), each with logo, dates, description, skills and embedded videos.

The hero includes a `<canvas id="constellation-canvas">` particle-network animation (vanilla JS, no library) that respects `prefers-reduced-motion` and pauses its `requestAnimationFrame` loop via `IntersectionObserver` when scrolled out of view.

Design tokens live at the top of `styles.css` as CSS custom properties: `--paper`, `--ink`, `--copper` (mechanical track), `--teal` (software track), `--font-display` (Lora) and `--font-body` (Inter). This is the "Constellation" design; see git history for prior rejected directions.

`assets/` holds the linked PDFs and the homelab network diagram (SVG).

### Things that have bitten before

- **Cascade order.** `styles.css` is one long file with no layers. A rule added near the top loses to an equally specific rule further down. Overrides intended to win must go later in the file, or carry higher specificity. The `#projects .skills-list li` rule in particular needs `#projects` in any override.
- **Mobile.** Justified body text leaves large ragged gaps at phone widths and is switched to left-aligned under 640px. The nav uses an invisible `li.nav-break` to force a consistent wrap point, because otherwise the break lands differently at 375px and 414px. Wide content (the network diagram) pans horizontally inside its own container rather than shrinking; the page body must never scroll sideways.
- **Verify in a browser, at both desktop and 375px, before committing.** Several regressions here were invisible in the diff.

### Content completeness constraint

Every job, project, link (company site / GitHub / Bitbucket / PDF), embedded video and skill tag in `index.html` is real content the owner asked to keep. None of it is placeholder. When editing, preserve all of it; this is a reorganization and update target, not something to trim for brevity.

**Do not claim skills that have not been exercised.** The homelab card is the live example: its "Skills Used" list covers only what the project's own checklist marks as built, and the roadmap keywords (Kubernetes, Terraform, Ansible, Prometheus, Grafana) sit in a separate, visually muted "Planned for later phases" list. The project status bar on the same card shows those phases at 0%, so listing them as skills would contradict the page itself.

## Writing style

- **Never use the em dash (`—`) in site copy.** Use a comma, colon, semicolon or hyphen. This applies to visible text; existing CSS comments are not worth churning.
- Prose should not read as AI-generated. Prefer concrete facts over three sentences restating the same claim.
- The About section is first-person and personal; keep that register.

## Git workflow for this project

`main` is the live, published site. Merging into it publishes immediately, with no review or staging environment, so treat a merge to `main` as a deliberate publish action rather than routine integration. Work on a branch, open a PR, and merge only when the owner says so.

For an initiative that benefits from running in parallel without blocking other work, use a **git worktree**: a separate sibling folder, e.g. `RuiGRMargarido.github.io - <branch-name>`, rather than switching branches in place.

Reference the relevant issue number in commit messages (e.g. `(#25)`) so GitHub cross-links the commit to its card; use `Closes #N` in the PR body, not in individual commits, so the issue auto-closes on merge.

Work is tracked on a GitHub Project board ("Portfolio Redesign", https://github.com/users/RuiGRMargarido/projects/5) with a `Track` field (Design / Content) and a `Status` field (Backlog / Todo / In Progress / Done). Prefer real Issues over draft cards so commits and PRs can link to them.

### GitHub Actions

`.github/workflows/auto-assign.yml` is the only workflow in this repo. It fires on `issues.opened` and `pull_request.opened` and assigns the new issue/PR to `RuiGRMargarido` via `actions/github-script`. There is no CI, build or deploy workflow: GitHub Pages publishes `main` directly, so nothing here runs tests or builds anything.

## Related repositories

- **`RuiGRMargarido/homelab`** (public) is the source for the homelab project card. Its `README.md` and `docs/CHECKLIST.md` are the source of truth for that card's description, network diagram and progress numbers. **The percentages on the site are copied by hand and go stale**; recount from the checklist when the project moves on.
- **`RuiGRMargarido/career-profile`** (private, local) holds the CV, LinkedIn and per-niche ATS keyword analysis. Useful when deciding which keywords the site should carry.
