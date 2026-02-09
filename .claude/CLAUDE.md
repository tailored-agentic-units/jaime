# Dev Blog — Claude Code Instructions

## Project

Jekyll 4.x blog deployed to GitHub Pages via GitHub Actions. Hosted at `https://tailored-agentic-units.github.io/jaime/`.

## Key Details

- **baseurl**: `/jaime` — all asset references must use `| relative_url` filter
- **Layouts**: `default` (HTML shell) → `home` (index) + `post` (articles) + `category` (filtered by category)
- **Categories**: Dynamic — defined by post frontmatter, auto-generated category pages via jekyll-paginate-v2 autopages at `/categories/{slug}/`. Nav links are driven by `site.categories`. Current: `progress`, `engineering`. Open-ended — new categories appear automatically when used in a post.
- **Pagination**: jekyll-paginate-v2 with autopages for category index pages
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
bundle exec jekyll serve               # local preview at localhost:4000/jaime/
```

## Playwright

All playwright-cli screenshots and recordings must be saved to `_capture/.playwright/`. Use `--filename=_capture/.playwright/<name>.png` for screenshots.

## Deployment

Push to `main` triggers `.github/workflows/pages.yml` — automatic build and deploy.
