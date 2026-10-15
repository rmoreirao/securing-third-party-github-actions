# Copilot instructions

This repository is a documentation project: a single-page guide on securing third-party GitHub Actions, published with GitHub Pages (Jekyll + Just the Docs), and a Marp slide deck that summarizes it in 30 minutes.

## Build and preview

There are no tests or linters. Validation means building the slides and, when layout or Jekyll syntax changes, the site.

```shell
npm ci --ignore-scripts   # always --ignore-scripts: the repo follows its own supply-chain advice
npm run slides            # docs/slides/index.html + docs/slides/deck.pdf
npm run slides:watch      # live preview
```

- `docs/slides/index.html`, `deck.pdf` and `deck.pptx` are gitignored. CI builds them; never commit them.
- Marp doesn't report overflowing content. After editing slides, render them to images in a temp folder and inspect the changed ones (`slide.001.png`, `slide.002.png`, … in deck order):

  ```shell
  npx marp docs/slides/deck.md --html --images png --allow-local-files -o <temp-dir>/slide.png
  ```

- On Linux runners Chrome may need `CHROME_NO_SANDBOX=1` (as in CI).
- The site builds with the same container as CI; see the `docker run` command in [README.md](../README.md#build-locally). Build the slides first so they're included.

## Architecture

- [docs/index.md](../docs/index.md): the whole guide on one page.
  - **Part 1**: TL;DR, ten principles (h4 headings), "Upstream, mirror or wrapper?", settings checklist.
  - **Part 2**: the Contoso story. Prologue, Chapters 1–10, Epilogue. Chapters 3–10 map to lifecycle steps 0–7, with sub-steps 2a/2b and 5a/5b/5c.
  - **Extras**: onboarding checklist (Critical/High/Medium/Low), request form, catalog entry example, further reading.
- [docs/slides/deck.md](../docs/slides/deck.md): the Marp deck. It's excluded from Jekyll (`exclude:` in `docs/_config.yml`); its built HTML/PDF are published at `/slides/`. It references the diagram as `../assets/...`.
- [docs/assets/third-party-actions-lifecycle.svg](../docs/assets/third-party-actions-lifecycle.svg): hand-written SVG with plain `<text>` labels plus `<title>`/`<desc>`, used by both the guide and the deck.
- [.github/workflows/pages.yml](workflows/pages.yml): on push to `main`, builds the slides, then the site with `actions/jekyll-build-pages`, then deploys to Pages.
- Theme: `just-the-docs` via `remote_theme`, pinned to a commit SHA in `docs/_config.yml`. Overrides live in `docs/_sass/custom/` (`setup.scss` for `$nav-width`, `custom.scss` for the centered reading column, table sizing and h4 principle headings). Footer text is in `docs/_includes/footer_custom.html`.

## Keep repeated content in sync

The same facts appear in several places. When one changes, grep for the others and update them too. After guide changes, check whether the deck needs updating.

- **Principles**: the TL;DR list, the h4 sections under "The principles", and the deck's "The principles (1/2)" and "(2/2)".
- **Team roles** (platform, security, Infra, dev teams as sponsor and consumers): the "Who owns what" paragraph, the Chapter 2 "cast" and "Who does what" tables, the Part 2 "process at a glance" table, and the deck's "Four teams, one process" slide and principles (2/2) takeaway.
- **Settings checklist**: the guide's "Settings checklist" and the deck's "Settings checklist (1/2)" and "(2/2)", item for item.
- **Numbers** (onboarding days, minimum ages, expiry windows, break-glass limits, the 1,000-entry allowlist limit and 800 warning): the Chapter 2 "The numbers" table, the chapter text, the deck (principle 9, allowlist slide, speaker notes) and the SVG labels.
- **Step names and numbers 0–7**: the "process at a glance" table, chapter headings, the SVG `<text>` and `<desc>`, and the deck's lifecycle slide.
- **Story outcomes** (29 minutes to block, 250+ credentials rotated, 64 repositories in the drill, …): chapters, the Chapter 9 and Epilogue tables, and deck speaker notes.
- **README.md**: "What's inside" describes the guide and slides; "This repository follows the guide" describes `pages.yml`.

## Conventions

### Jekyll and Markdown

- The body of `docs/index.md` is wrapped in `<!-- {% raw %} -->` … `<!-- {% endraw %} -->` so Liquid doesn't process `${{ }}` expressions. Keep new content inside these markers.
- kramdown syntax is used: `{: .no_toc }`, `{: .note }`, `{:toc}`, `<details markdown="block">`.
- In-page links use kramdown's auto-generated IDs, where `·` and `—` drop out and leave a double hyphen (for example `#chapter-3-cleaning-house-step-0--bootstrap`). Renaming a heading breaks them: grep for `](#` and fix the links.
- Chapters are h3; principles and sub-steps are h4.
- Line endings are LF (`.gitattributes`).

### Writing style (guide and slides)

- Plain English, short sentences, contractions, and second person in Part 1. Bullets often start with a bold sentence: `**Pin everything to a full commit SHA.** Tags move. Commits don't.`
- Don't use em dashes in prose; use a colon, a comma or a new sentence. They appear only in the Part headings and the onboarding checklist's criticality labels.
- Step labels use a middle dot: `0 · Bootstrap`, `2a · Trusted Publisher Onboarding`.
- Story characters carry a role tag: **Sam (Security)**, **Marta (Platform)**, and **Ana (Payments)** or **Ana (Sponsor)** depending on her role in the scene. Write "Infra team" with a capital I.
- Contoso and Fabrikam (publisher of `helm-diff-action`) are fictional. The real incidents (`tj-actions/changed-files`, CVE-2025-30066; the Trivy action) are public record, so don't invent details about them. The story runs from the March 2025 incident to the March 2026 epilogue.
- Name GitHub settings exactly as the UI does, in bold: `**Require actions to be pinned to a full-length commit SHA:** on`.
- The "personal views, not official Microsoft guidance" disclaimer appears in the guide's note, README, site footer and title slide. Keep them consistent.

### Slides (Marp)

- Slides are separated by `---`. Per-slide directives: `<!-- _class: lead -->` for title slides, `<!-- _class: dense -->` for smaller text and tables.
- Reusable styles are defined in the deck's front-matter `style:`: `.takeaway` (closing one-liner box), `.cols` (two-column grid), `ul.check` (checkbox lists written as raw `<ul class="check"><li>` HTML). Use `<style scoped>` for one-off tweaks.
- Speaker notes go in a trailing `<!-- … -->` comment on each slide.
- Slides are 16:9 (1280×720). Keep the deck to a 30-minute talk.

### The repository follows its own guide

`pages.yml` and the build must keep doing what the README promises:

- Pin every `uses:` to a full 40-character SHA with a `# vX.Y.Z` comment.
- `permissions: {}` at the workflow level, with the minimum per job; `persist-credentials: false` on checkout; no secrets.
- Install tools from the lockfile with `npm ci --ignore-scripts`, never an unpinned `npx` download. `@marp-team/marp-cli` is pinned to an exact version.
- Keep `remote_theme` pinned to a commit SHA.
- If any of this changes, update the "This repository follows the guide" section in README.md.

### Licensing

The guide, slides and diagram are CC BY 4.0 ([LICENSE](../LICENSE)). Code samples and workflow files are MIT ([LICENSE-CODE](../LICENSE-CODE)).
