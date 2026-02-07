# Dev Blog — Claude Code Instructions

## Project

Jekyll 4.x blog deployed to GitHub Pages via GitHub Actions. Hosted at `https://tailored-agentic-units.github.io/jaime/`.

## Key Details

- **baseurl**: `/jaime` — all asset references must use `| relative_url` filter
- **Layouts**: `default` (HTML shell) → `home` (index) + `post` (all entries)
- **Content types**: Updates (category: update) and concepts (category: concept) — same layout, differentiated by frontmatter
- **CSS**: Cascade layers in `assets/css/` — tokens, reset, theme, layout, components, syntax. Entry point is `index.css`.
- **Syntax highlighting**: Rouge GitHub (dark + light) via `prefers-color-scheme`
- **Media**: Gitignored `assets/media/` for local staging. Published media goes to GitHub Releases with tag `post/{slug}`.

## Post Creation

1. Create file in `_posts/` with naming convention `YYYY-MM-DD-slug.md`
2. Include frontmatter: layout, title, date, tags, category, excerpt
3. Reference `.claude/context/style-profile.md` for writing voice calibration

## Build

```bash
bundle exec jekyll build
bundle exec jekyll serve           # local preview at localhost:4000/jaime/
```

## Playwright

All Playwright MCP screenshots and artifacts must be saved to `.playwright-mcp/` — never to the repo root. Use absolute or relative paths prefixed with `.playwright-mcp/` for the `filename` parameter (e.g., `.playwright-mcp/preview.png`).

## Deployment

Push to `main` triggers `.github/workflows/pages.yml` — automatic build and deploy.
