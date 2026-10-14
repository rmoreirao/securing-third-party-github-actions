# Securing Third-Party GitHub Actions at Scale

An opinionated guide for platform teams and developers on how to securely onboard, adopt and monitor third-party GitHub Actions, without slowing anyone down.

**[Read the guide](https://rmoreirao.github.io/securing-third-party-github-actions/)** · **[Slides](https://rmoreirao.github.io/securing-third-party-github-actions/slides/)** · **[Slides (PDF)](https://rmoreirao.github.io/securing-third-party-github-actions/slides/deck.pdf)**

## What's inside

- **Part 1: Principles and best practices.** Ten principles and a settings checklist that work for any organization.
- **Part 2: How Contoso secured its third-party actions.** A fictional story, built from real incidents, that walks through every step: bootstrap, request, onboarding, adoption, monitoring, lifecycle, compromise response and break-glass.
- **Extras.** An onboarding checklist, a request form, a catalog entry example and further reading.
- **Slides.** A 30-minute summary for presenting the guide: the incident, the principles and the settings checklist.

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
| [`docs/slides/deck.md`](docs/slides/deck.md) | The slides, written in [Marp](https://marp.app/) Markdown, with speaker notes |
| [`docs/assets/`](docs/assets/) | The lifecycle diagram |
| [`.github/workflows/pages.yml`](.github/workflows/pages.yml) | Builds the slides and the site, then deploys to GitHub Pages on every push to `main` |

### Site layout

The desktop sidebar is fixed at `14rem` (224px at the default font size). Adjust `$nav-width` in [`docs/_sass/custom/setup.scss`](docs/_sass/custom/setup.scss) to resize it. The layout overrides in [`docs/_sass/custom/custom.scss`](docs/_sass/custom/custom.scss) prevent the theme from adding an extra sidebar gutter on wide screens and let the article fill the remaining viewport width, with normal content padding. The mobile menu is unchanged.

## Build locally

**Slides** need Node.js 18 or later:

```shell
npm ci --ignore-scripts
npm run slides        # docs/slides/index.html and docs/slides/deck.pdf
npm run slides:pptx   # PowerPoint export (each slide as an image)
npm run slides:watch  # live preview while you edit
```

**The site** builds with the same container the workflow uses. Build the slides first so they're included:

```shell
docker run --rm -v "${PWD}:/github/workspace" \
  -e GITHUB_WORKSPACE=/github/workspace -e INPUT_SOURCE=./docs -e INPUT_DESTINATION=./_site \
  ghcr.io/actions/jekyll-build-pages:v1.0.13
```

## Disclaimer

The views in this guide are my own and aren't official Microsoft guidance. Contoso and Fabrikam are fictional. The incidents referenced are public. GitHub settings and limits change often, so check the linked docs before you rely on them.

## License

The guide, the slides and the diagram are licensed under [CC BY 4.0](LICENSE). Code samples and workflow files are licensed under [MIT](LICENSE-CODE).
