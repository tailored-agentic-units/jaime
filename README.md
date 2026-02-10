# Jaime's Dev Blog

Development updates and technical concepts from the TAU ecosystem.

**Live**: [tailored-agentic-units.github.io/jaime](https://tailored-agentic-units.github.io/jaime/)

## Stack

- Jekyll 4.4, Kramdown (GFM), Rouge syntax highlighting
- Plugins: jekyll-feed, jekyll-paginate-v2, jekyll-seo-tag, jekyll-sitemap
- CSS cascade layers (tokens → reset → theme → layout → components → syntax)
- GitHub Pages deployment via GitHub Actions

## Local Development

**Prerequisites**: Ruby, Bundler

```bash
bundle install
bundle exec jekyll serve          # http://127.0.0.1:4000/jaime/
bundle exec jekyll serve --drafts # include _drafts/
```

> The site uses `baseurl: /jaime`. All asset references in templates must use the `| relative_url` filter.

## Project Structure

```
_posts/              # Published posts (YYYY-MM-DD-slug.md)
_drafts/             # Work-in-progress posts
_layouts/            # default → home, post, category
_includes/           # nav, head, post-card, pagination, video
assets/css/          # CSS cascade layers (see below)
.claude/             # Claude Code config, skills, style profile
.github/workflows/   # GitHub Actions deployment
```

## CSS Architecture

The stylesheet uses CSS cascade layers, declared and imported in `assets/css/index.css`:

| Layer | File | Purpose |
|-------|------|---------|
| `tokens` | `tokens.css` | Design tokens (colors, spacing, typography) |
| `reset` | `reset.css` | Browser normalization |
| `theme` | `theme.css` | Dark/light theme via `prefers-color-scheme` |
| `layout` | `layout.css` | Page structure and grid |
| `components` | `components.css` | Post cards, tags, navigation |
| `syntax` | `syntax.css` | Rouge code highlighting (dark + light variants) |

## Content

**Categories** are dynamic — defined by post frontmatter and auto-generated via jekyll-paginate-v2 autopages. Current categories: `progress`, `engineering`. New categories appear automatically when used in a post.

### Post Frontmatter

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS
tags: [tag-1, tag-2]
category: progress
excerpt: "One-sentence summary for the post listing."
---
```

### URL Structure

```
/jaime/{category}/YYYY/MM/DD/{slug}.html
```

### Media

Binary assets are hosted via GitHub Releases to avoid git history bloat:

- **Tag pattern**: `post/{slug}`
- **Download URL**: `https://github.com/tailored-agentic-units/jaime/releases/download/post/{slug}/{filename}`
- **Local staging**: `assets/media/` (gitignored) — rewritten to release URLs on publish

## Dev Blog Skill (`/dev-blog`)

A Claude Code skill that provides a structured authoring pipeline: **capture → draft → publish**.

| Command | Description |
|---------|-------------|
| `/dev-blog init` | Scaffold a new blog with Jekyll, CSS layers, and GitHub Pages deployment |
| `/dev-blog calibrate` | Generate or update the writing style profile from samples |
| `/dev-blog capture <slug>` | Stage a development entry into a named capture bucket |
| `/dev-blog draft <bucket>` | Generate a rough draft from captured entries |
| `/dev-blog publish <draft>` | Finalize a draft, release media, and deploy |

The writing style profile at `.claude/context/style-profile.md` calibrates voice for draft generation. It is produced by `calibrate` and consumed by `draft` and `publish`.

## Deployment

Pushing to `main` triggers the GitHub Actions workflow (`.github/workflows/pages.yml`):

1. Checks out the repository
2. Sets up Ruby 4.0 with bundler cache
3. Builds with `JEKYLL_ENV=production`
4. Deploys via `actions/deploy-pages`
