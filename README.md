# Jamie Air To Water

A static reference site for air-to-water heat pump servicing: error-code lookup,
installer codes, manuals, and fault-finding videos.

**Live site:** https://jayreddin.github.io/JamieAirToWater/index.html

## Structure

- `index.html` — landing page with links to each brand page
- `Heat Pumps/` — one page per brand (error-code search, PDF manual viewer,
  installer codes) plus fault-video pages and the AI assistant page
- `JSON/` — error-code and troubleshooting databases fetched by the brand pages
- `PDF/` — manufacturer manuals and error-code documents
- `images/` — images
- `style.css` — shared stylesheet

## Deployment

Deployed to GitHub Pages via `.github/workflows/static.yml` on every push to
`main` (or manually from the Actions tab).

PR preview deployments (each pull request published to its own Pages URL)
are prepared but not yet enabled - see [docs/pr-previews.md](docs/pr-previews.md)
for the ready-to-use workflow files and the two one-time repository settings
they require.
