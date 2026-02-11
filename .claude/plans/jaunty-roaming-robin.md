# Plan: Create README.md

## Context

The repository has no README.md. One is needed to describe the blog, its architecture, the Claude Code `dev-blog` skill workflow, and how to build/test locally. The file is already listed in `_config.yml` `exclude:` (line 46), so it won't affect the Jekyll build.

## Approach

Create `README.md` at the repository root with these sections:

### 1. Header & Description
- Project name: **Jaime's Dev Blog**
- One-line description from `_config.yml`: "Development updates and technical concepts from the TAU ecosystem."
- Live URL: `https://tailored-agentic-units.github.io/jaime/`

### 2. Stack
- Jekyll 4.4, Kramdown (GFM), Rouge syntax highlighting
- Plugins: jekyll-feed, jekyll-paginate-v2, jekyll-seo-tag, jekyll-sitemap
- CSS cascade layers (tokens → reset → theme → layout → components → syntax)
- GitHub Pages deployment via GitHub Actions

### 3. Local Development
- Prerequisites: Ruby, Bundler
- `bundle install`
- `bundle exec jekyll serve` → `http://127.0.0.1:4000/jaime/`
- `bundle exec jekyll serve --drafts` to include `_drafts/`
- Note about `baseurl: /jaime` requiring `| relative_url` filter on asset references

### 4. Project Structure
- Brief tree showing key directories: `_posts/`, `_drafts/`, `_layouts/`, `_includes/`, `assets/css/`, `.claude/`, `.github/workflows/`

### 5. CSS Architecture
- Layer order and purpose of each file in `assets/css/`
- Dark/light theme support via `prefers-color-scheme`

### 6. Content
- Categories: dynamic, driven by post frontmatter (current: `progress`, `engineering`)
- Post frontmatter schema (layout, title, date, tags, category, excerpt)
- URL structure: `/{baseurl}/{category}/YYYY/MM/DD/{slug}.html`
- Media strategy: GitHub Releases with tag `post/{slug}`

### 7. Dev Blog Skill (`/dev-blog`)
- Overview of the Claude Code skill and its pipeline: capture → draft → publish
- Command table: init, calibrate, capture, draft, publish
- Style profile calibration at `.claude/context/style-profile.md`

### 8. Deployment
- Push to `main` triggers GitHub Actions workflow
- Builds with `JEKYLL_ENV=production`, deploys via `actions/deploy-pages`

## Files Modified

- **Create**: `README.md` (repo root)

## Verification

- `bundle exec jekyll build` — confirm README.md is excluded and build succeeds
- Visual review of the rendered README on GitHub (or locally via markdown preview)
