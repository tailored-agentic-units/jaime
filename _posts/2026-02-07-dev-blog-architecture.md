---
layout: post
title: "Dev Blog System — Architecture Concept"
date: 2026-02-07 10:00:00
tags: [dev-blog, architecture, jekyll, github-pages]
category: engineering
excerpt: "Architecture concept for a developer blog built on GitHub Pages with Jekyll, managed through a Claude Code skill plugin."
---

A stylized developer blog built on [GitHub Pages](https://pages.github.com/) with Jekyll, managed entirely through a Claude Code skill plugin. No themes, no Node.js, no build toolchain beyond what GitHub Pages provides natively. Push markdown, get HTML.

---

## Core Principles

- Start at the lowest level using latest supported features, expand complexity as needed
- Vanilla CSS from scratch — no frameworks, no preprocessors
- Jekyll without a theme gem — raw Liquid layouts, full control
- Deterministic operations separated from non-deterministic (style inference runs once, not per-post)
- Single skill with sub-command routing, not multiple independent skills
- Media stored outside git history via GitHub Releases (one release per post)

---

## Technology Stack

| Layer | Choice | Rationale |
|-------|--------|-----------|
| Static site generator | Jekyll 4.x | Custom Actions workflow, processes markdown, respects frontmatter, allows raw HTML |
| Markdown engine | kramdown (GFM) | Jekyll default, supports fenced code blocks with language hints |
| Syntax highlighting | Rouge | No JS dependency, generates static CSS via `rougify` |
| Syntax theme | Base16-derived (dark + light) | System-responsive via `prefers-color-scheme`, custom palette designed through the theme-design skill |
| CSS | Vanilla, cascade layers | Full control, 8-layer architecture (reset, tokens, typography, colors, layout, borders, transitions, components) |
| Media hosting | GitHub Releases | Per-post release bundles, stable download URLs, zero git history bloat |
| Deployment | GitHub Actions | Push to `main` triggers build and deploy to Pages |
| Automation | Claude Code skill plugin | Single skill, sub-command architecture with context layer |

---

## Repository Structure

```
jaime/
├── .github/
│   └── workflows/
│       └── pages.yml              # Jekyll build + deploy to Pages
├── .claude/
│   ├── CLAUDE.md                  # Project-level Claude Code instructions
│   ├── context/
│   │   └── style-profile.md       # Generated writing voice calibration
│   └── skills/                    # Skill plugins (dev-blog, theme-design)
├── _config.yml
├── _layouts/
│   ├── default.html               # Base HTML5 shell (head, nav, footer)
│   ├── home.html                  # Index page with post listing
│   ├── post.html                  # Individual blog entry (all content types)
│   └── category.html              # Category index with pagination
├── _includes/
│   ├── head.html                  # Meta, CSS, SEO, feed
│   ├── nav.html                   # Site navigation with category dropdown
│   ├── post-card.html             # Reusable post listing card
│   ├── pagination.html            # Prev/next page controls
│   └── video.html                 # Reusable video tag partial
├── _posts/                        # Published markdown (date-prefixed)
├── _drafts/                       # Authoring target, excluded from production
├── _capture/                      # Capture buckets (excluded from Jekyll build)
├── assets/
│   └── css/
│       ├── index.css              # Layer order declaration + imports
│       ├── reset.css              # Modern CSS reset
│       ├── tokens.css             # Design tokens (colors, spacing, type scale)
│       ├── typography.css         # Font assignments, heading scale, code
│       ├── colors.css             # Semantic color mapping + syntax highlighting
│       ├── layout.css             # Page structure, vertical rhythm, spacing
│       ├── borders.css            # Border styles for tables, dividers, code
│       ├── transitions.css        # Link and interactive element transitions
│       └── components.css         # Nav, post cards, tags, pagination
├── 404.html                       # Custom 404 page
├── Gemfile
└── index.md                       # Home page
```

### Key Structural Decisions

- **`_drafts/`** is the default authoring target. Jekyll ignores drafts in production, creating a review gate before publish.
- **`_capture/`** is excluded from the Jekyll build via `_config.yml`. Capture buckets live in the repo for traceability but never render.
- **`assets/media/`** is gitignored entirely. It exists only as a local staging area — on publish, media is uploaded to a GitHub Release and the directory is created on demand.
- **One `post` layout** handles all content types. Differentiation is expressed through frontmatter tags and categories, not separate templates. Category index pages are auto-generated via jekyll-paginate-v2 autopages.

---

## CSS Architecture

The stylesheet is organized into cascade layers, with each layer defined in its own file and imported through a single entry point:

```css
/* index.css */
@layer reset, tokens, typography, colors, layout, borders, transitions, components;

@import url("./reset.css");
@import url("./tokens.css");
@import url("./typography.css");
@import url("./colors.css");
@import url("./layout.css");
@import url("./borders.css");
@import url("./transitions.css");
@import url("./components.css");
```

### Layer Responsibilities

| Layer | Purpose |
|-------|---------|
| **reset** | Modern CSS reset — box-sizing, margin removal, media defaults, text wrapping |
| **tokens** | CSS custom properties — theme colors, Base16 palette, spacing, type scale, fonts. Light/dark variants via `prefers-color-scheme` |
| **typography** | Font assignments per voice (body, chrome, code), heading scale, line heights |
| **colors** | Semantic color mapping for surfaces, text, links, and Rouge syntax highlighting classes |
| **layout** | Page structure — content width, vertical rhythm, heading spacing, responsive breakpoints |
| **borders** | Border styles for tables, horizontal rules, code blocks, blockquotes |
| **transitions** | Link hover/focus transitions and interactive element effects |
| **components** | Nav dropdown, post cards, tags, pagination, post headers |

### Light and Dark Mode

The color system follows a "theme colors as source of truth" architecture. Three role-based theme colors are defined as direct hex values, and Base16 accent slots derive from them where applicable:

```css
:root {
  /* Theme colors — source of truth */
  --color-chrome: #4cd48a;
  --color-interactive: #5e8ef0;
  --color-emphasis: #e464a0;

  /* Base16 accents — derived from theme where applicable */
  --base0B: var(--color-chrome);
  --base0D: var(--color-interactive);
  --base0E: var(--color-emphasis);
  /* ... remaining accents use independent values */
}
```

Light mode overrides the theme colors and freely remaps Base16 accent slots without conditional logic:

```css
@media (prefers-color-scheme: light) {
  :root {
    --color-chrome: #0e8848;
    --color-interactive: #1858d0;
    --color-emphasis: #d02878;

    /* Accents remapped for light mode balance */
    --base0A: var(--color-emphasis);  /* pink — types */
    --base0E: #5a38c0;               /* indigo — keywords */
  }
}
```

Syntax highlighting maps Rouge token classes to Base16 slots in `colors.css`, so both dark and light modes receive appropriate highlighting from the same set of rules.

---

## Media Strategy

### Problem

Git history bloats with binary files. GitHub Pages does not serve Git LFS-tracked files (resolves to pointer files).

### Solution

GitHub Releases as a per-post CDN.

- **Tag pattern**: `post/{slug}` (e.g., `post/introducing-the-dev-blog`)
- **All media** for a post uploaded as release assets
- **Stable download URLs**: `https://github.com/tailored-agentic-units/jaime/releases/download/post/{slug}/{filename}`

### Video Support

Raw HTML passes through kramdown unchanged. A reusable include handles the common case:

```liquid
{% raw %}{% include video.html src="https://github.com/.../demo.mp4" %}{% endraw %}
```

Resolved by `_includes/video.html`:

```html
<video controls width="100%">
  <source src="{{ include.src }}" type="video/mp4">
</video>
```

---

## Capture / Draft / Publish Workflow

A three-stage authoring pipeline that captures development progress incrementally rather than reconstructing it at the end of the week.

### Capture

As noteworthy work is completed, a brief entry is staged into a named bucket:

```
_capture/
└── 2026-02-07-introducing-the-dev-blog/
    ├── 01-blog-scaffold.md
    ├── 02-css-architecture.md
    └── media/
        └── screenshot.png
```

Each bucket follows the naming convention `[date]-[slug]`. Entries within are ordered by sequence number and include a description with optional media references.

### Draft

When it's time to write, the `draft` command reads all entries from a specified bucket and generates a rough draft in `_drafts/` with validated frontmatter. The [style profile](#skill-plugin-design) calibrates the voice. Media references point to local staging paths.

### Publish

The `publish` command finalizes the draft:

1. Promotes the post from `_drafts/` to `_posts/` with the current date prefix
2. Creates a GitHub Release tagged `post/{slug}`
3. Uploads all associated media as release assets
4. Rewrites media paths in the post to release download URLs
5. Commits and pushes

---

## Skill Plugin Design

The blog is managed through a single Claude Code skill with sub-command routing. The skill is distributed as a plugin — the blog-specific content (posts, captures, style profile) is generated at runtime, not shipped with the skill.

### Command Routing

| User Intent | Command | Description |
|-------------|---------|-------------|
| "set up my blog" | `init` | Scaffold repository, configure Pages, generate CSS |
| "calibrate my writing style" | `calibrate` | Analyze writing samples, generate style profile |
| "capture this" | `capture` | Stage entry into a capture bucket |
| "draft from this bucket" | `draft` | Generate rough draft from captured entries |
| "publish the post" | `publish` | Finalize draft, release media, deploy |

### Context Layer

| File | Type | Location | Created By |
|------|------|----------|------------|
| `style-profile.md` | Generated (user-specific) | `.claude/context/style-profile.md` | `calibrate` |
| `layout-schemas.md` | Static (ships with skill) | Skill `context/` directory | `init` |

The style profile lives at the project level, not within the skill — each user generates their own via the `calibrate` command. If the profile doesn't exist when `draft` or `publish` runs, the skill triggers calibration first.

---

## Deployment

The site deploys automatically via a GitHub Actions workflow triggered on push to `main`:

```yaml
jobs:
  build:
    steps:
      - uses: actions/checkout@v4
      - uses: ruby/setup-ruby@v1
        with:
          ruby-version: "4.0"
          bundler-cache: true
      - run: bundle exec jekyll build
        env:
          JEKYLL_ENV: production
      - uses: actions/upload-pages-artifact@v3

  deploy:
    needs: build
    steps:
      - uses: actions/deploy-pages@v4
```

The workflow requires `pages: write` and `id-token: write` permissions. The repository's Pages settings must be configured to deploy from GitHub Actions rather than a branch.

---

## Key Principles

1. **Push markdown, get HTML** — the entire publishing pipeline is git-native
2. **Media outside git** — GitHub Releases serve binary assets without bloating history
3. **System-responsive design** — light and dark modes follow user preference with zero JavaScript
4. **Capture incrementally, publish periodically** — the authoring workflow matches how development actually happens
5. **Skill automates, user owns** — the plugin manages process; content, voice, and style belong to the author
