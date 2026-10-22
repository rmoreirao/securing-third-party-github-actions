# Securing Third-Party GitHub Actions at Scale

An opinionated guide for platform teams and developers on how to securely onboard, adopt and monitor third-party GitHub Actions, without slowing anyone down.

**[Read the guide](https://rmoreirao.github.io/securing-third-party-github-actions/)** · **[Slides](https://rmoreirao.github.io/securing-third-party-github-actions/slides/principles-then-practice.html)** · **[Slides (PDF)](https://rmoreirao.github.io/securing-third-party-github-actions/slides/principles-then-practice.pdf)**

## What's inside

- **Part 1: Principles and best practices.** Ten principles and a settings checklist that work for any organization.
- **Part 2: How Contoso secured its third-party actions.** A fictional story, built from real incidents, that walks through every step: bootstrap, request, onboarding, adoption, monitoring, lifecycle, compromise response and break-glass.
- **Extras.** An onboarding checklist, a request form, a catalog entry example and further reading.
- **Slides: principles, then practice.** A 30-minute deck in two clear parts: the ten principles first, then Contoso applying them through steps 0–7. It has 27 minutes of timed speaker notes, 3 minutes for questions and reference slides after the close. This is the deck the site and this README link to.
- **Original slides.** The earlier 30-minute deck, which mixes the incident, principles, lifecycle and settings checklist. It's still built and published at [`/slides/`](https://rmoreirao.github.io/securing-third-party-github-actions/slides/) ([PDF](https://rmoreirao.github.io/securing-third-party-github-actions/slides/deck.pdf)), but nothing links to it.

## This repository follows the guide

The [workflow that publishes this site](.github/workflows/pages.yml) applies the guide's own principles:

- It uses only GitHub's own `actions/*` actions, each pinned to a full commit SHA.
- It starts with `permissions: {}`, and each job asks only for what it needs.
- It sets `persist-credentials: false` on checkout, and uses no secrets.
- Build tools come from a lockfile (`npm ci --ignore-scripts`), not from an unpinned `npx` download.
- The Jekyll theme is pinned to a commit SHA too.

One known gap: `actions/jekyll-build-pages` runs a container image referenced by tag, not by digest. It comes from a trusted publisher, and the job holds no secrets and only a read-only token.

## Repository layout

| Path | What it is |
|---|---|
| [`docs/index.md`](docs/index.md) | The guide, published with GitHub Pages and the [Just the Docs](https://github.com/just-the-docs/just-the-docs) theme |
| [`docs/slides/principles-then-practice.md`](docs/slides/principles-then-practice.md) | The linked slides, written in [Marp](https://marp.app/) Markdown: principles first, then Contoso's application, with timed speaker notes |
| [`docs/slides/deck.md`](docs/slides/deck.md) | The original deck, still built and published at `/slides/` |
| [`docs/assets/`](docs/assets/) | The lifecycle diagram shared by the guide and slides |
| [`.github/workflows/pages.yml`](.github/workflows/pages.yml) | Builds the slides and the site, then deploys to GitHub Pages on every push to `main` |

### Site layout

The desktop sidebar is fixed at `14rem` (224px at the default font size). Adjust `$nav-width` in [`docs/_sass/custom/setup.scss`](docs/_sass/custom/setup.scss) to resize it. The layout overrides in [`docs/_sass/custom/custom.scss`](docs/_sass/custom/custom.scss) prevent the theme from adding an extra sidebar gutter on wide screens and center the article within the remaining viewport space. The reading column is capped at `53rem` (848px at the default font size), plus the theme's normal content padding. On narrower screens, it shrinks to fit. The mobile menu is unchanged.

Tables size to their content rather than stretching across the reading column. Wide tables retain the theme's horizontal scrolling inside their own container, without widening the page.

## Build locally

**Slides** need Node.js 18 or later:

```shell
npm ci --ignore-scripts
npm run slides        # docs/slides/index.html and docs/slides/deck.pdf
npm run slides:pptx   # PowerPoint export (each slide as an image)
npm run slides:watch  # live preview while you edit
npm run slides:two-parts  # separate principles-then-practice.html and .pdf
```

The alternative build doesn't overwrite the original deck or its HTML/PDF. Both decks are built in CI and published under `/slides/`; generated files stay gitignored.

**The site** builds with the same container the workflow uses. Build both decks first so they're included:

```shell
docker run --rm -v "${PWD}:/github/workspace" \
  -e GITHUB_WORKSPACE=/github/workspace -e INPUT_SOURCE=./docs -e INPUT_DESTINATION=./_site \
  ghcr.io/actions/jekyll-build-pages:v1.0.13
```

## Disclaimer

The views in this guide are my own and aren't official Microsoft guidance. Contoso and Fabrikam are fictional. The incidents referenced are public. GitHub settings and limits change often, so check the linked docs before you rely on them.

## License

The guide, the slides and the diagram are licensed under [CC BY 4.0](LICENSE). Code samples and workflow files are licensed under [MIT](LICENSE-CODE).
