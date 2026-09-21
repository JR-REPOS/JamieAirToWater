# PR Preview Deployments to GitHub Pages

This repo is prepared for per-PR preview deployments. Once enabled, every pull
request is published to:

```
https://jayreddin.github.io/JamieAirToWater/pr-preview/pr-<number>/
```

A sticky comment with the link is posted on the PR, the preview is refreshed on
every new commit, and it is removed automatically when the PR is closed.
(Previews for PRs from forks are not supported by the underlying action.)

## Why these files are not already active

GitHub Pages previews require the site to be served from a branch, and the
two workflow files below must live in `.github/workflows/`. Applying them
needs two one-time repository settings changes (steps 1-2 below) and adding
the files (step 3).

> Note: `actions/deploy-pages` has a native `preview` input, but it is in
> closed alpha and not publicly available, so this uses the battle-tested
> `rossjrw/pr-preview-action` approach instead.

## Setup (one-time)

### 1. Switch Pages to branch deployment

**Settings > Pages > Source**: select **Deploy from a branch**, branch
`gh-pages`, folder `/ (root)`.

The `gh-pages` branch has already been created with the current site content,
so the live site keeps working through the switch.

### 2. Allow workflows to write

**Settings > Actions > General > Workflow permissions**: select
**Read and write permissions** (needed to push previews to `gh-pages` and
comment on PRs).

### 3. Add the workflow files

Create the two files below at `.github/workflows/deploy.yml` and
`.github/workflows/preview.yml` (via the GitHub web UI: **Add file > Create
new file**, or a local git push), then delete `.github/workflows/static.yml`
(it is replaced by `deploy.yml`).

---

## `.github/workflows/deploy.yml`

Publishes `main` to the root of the `gh-pages` branch on every push.
`.github/` is excluded from the published site, and `pr-preview/` is
protected so production deploys never delete PR previews.

```yaml
# Deploys the production site to GitHub Pages by publishing the repo
# content to the root of the gh-pages branch.
name: Deploy site to GitHub Pages

on:
  push:
    branches: ["main"]

  # Allow running manually from the Actions tab
  workflow_dispatch:

permissions:
  contents: write

# One deployment at a time; let in-progress production deployments finish.
concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Assemble site
        run: |
          mkdir _site
          rsync -a --exclude '.git' --exclude '.github' --exclude '_site' ./ _site/

      - name: Publish to gh-pages branch
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          branch: gh-pages
          folder: _site
          clean: true
          # Never delete the PR previews that live in pr-preview/
          clean-exclude: |
            pr-preview
            pr-preview/**
```

## `.github/workflows/preview.yml`

Deploys each pull request to `pr-preview/pr-<number>/` on the `gh-pages`
branch and comments the preview URL on the PR.

```yaml
# Deploys a preview of every pull request to GitHub Pages at:
#   https://jayreddin.github.io/JamieAirToWater/pr-preview/pr-<number>/
name: Deploy PR preview

on:
  pull_request:
    types:
      - opened
      - reopened
      - synchronize
      - closed

permissions:
  contents: write        # push the preview to the gh-pages branch
  pull-requests: write   # post the preview-link comment on the PR

concurrency: preview-${{ github.ref }}

jobs:
  deploy-preview:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        if: github.event.action != 'closed'
        uses: actions/checkout@v4

      - name: Assemble site
        if: github.event.action != 'closed'
        run: |
          mkdir _site
          rsync -a --exclude '.git' --exclude '.github' --exclude '_site' ./ _site/

      - name: Deploy preview
        uses: rossjrw/pr-preview-action@v1
        with:
          source-dir: ./_site
          preview-branch: gh-pages
          umbrella-dir: pr-preview
          # Don't use the third-party QR code service in the PR comment
          qr-code: false
```

## How it works

- `deploy.yml` keeps the production site at
  `https://jayreddin.github.io/JamieAirToWater/` in sync with `main`.
- `preview.yml` copies each PR's content into `pr-preview/pr-<number>/` on
  the same branch. Because the whole site is copied, relative links
  (`../JSON/...`, `Heat Pumps/...`) work inside the preview.
- Closing the PR deletes its preview directory.
