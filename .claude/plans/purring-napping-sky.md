# Dev Blog — Implementation Plan

## Context

Setting up a personal developer blog at `tailored-agentic-units/jaime` on GitHub Pages. Jekyll 4.x with a custom GitHub Actions workflow, vanilla CSS from scratch, no theme gem. The blog publishes two content types — regular updates (primary) and technical concept documents — both using the same `post` layout, differentiated by tags/categories. This session focuses on getting the blog live with seed content. The dev-blog skill plugin follows once the site design is finalized.

## Decisions (from concept session)

- **Jekyll 4.x** via Bundler + custom Actions workflow (not github-pages gem)
- **Rouge theme**: gruvbox.dark + gruvbox.light (system `prefers-color-scheme` toggle)
- **URL**: `https://tailored-agentic-units.github.io/jaime/` (baseurl: `/jaime`)
- **Layouts**: `default` (HTML shell) → `home` (index) + `post` (all entries)
- **Seed posts**: One update + one concept from `.artifacts/` samples
- **Capture workflow**: `_capture/` directory with `[date]-[slug]` buckets (structure only, skill is future work)
- **Media**: GitHub Releases as CDN, tag pattern `post/{slug}` (future — no media in seed posts)

## Environment

- Ruby 4.0.1, Bundler 4.0.3, gh CLI authenticated as JaimeStill
- Repo does not exist yet on GitHub
- Working directory: `/home/jaime/tau/jaime/` (not yet a git repo)
- Existing files: `.artifacts/` (samples), `.claude/` (settings), `prompt.md` (arch concept)

---

## Execution Steps

### Step 1 — Create GitHub repo + initialize git

```bash
gh repo create tailored-agentic-units/jaime --public --description "Jaime's dev blog" --clone=false
cd /home/jaime/tau/jaime
git init
git remote add origin https://github.com/tailored-agentic-units/jaime.git
```

### Step 2 — Write Gemfile

```ruby
source "https://rubygems.org"

gem "jekyll", "~> 4.4"

group :jekyll_plugins do
  gem "jekyll-feed"
  gem "jekyll-seo-tag"
  gem "jekyll-sitemap"
end
```

Run `bundle install` to generate `Gemfile.lock` and install dependencies (including Rouge, which is a Jekyll dependency).

### Step 3 — Write `_config.yml`

Key configuration:
- `baseurl: "/jaime"` (critical for org project site)
- `url: "https://tailored-agentic-units.github.io"`
- kramdown with Rouge, GFM input
- Syntax highlighting: `rouge` with `gruvbox.dark` (Note: Rouge's CSS class is `gruvbox` for the dark variant — will verify after gem install)
- Exclude: `_capture`, `.artifacts`, `samples`, `prompt.md`, `Gemfile`, `Gemfile.lock`, `README.md`
- Plugins: jekyll-feed, jekyll-seo-tag, jekyll-sitemap

### Step 4 — Generate syntax.css (light + dark)

Generate both gruvbox variants and wrap them in `prefers-color-scheme` media queries:

```bash
bundle exec rougify style gruvbox.dark > /tmp/syntax-dark.css
bundle exec rougify style gruvbox.light > /tmp/syntax-light.css
```

Combine into `assets/css/syntax.css`:
```css
@media (prefers-color-scheme: dark) {
  /* contents of syntax-dark.css */
}
@media (prefers-color-scheme: light) {
  /* contents of syntax-light.css */
}
```

If `gruvbox.dark`/`gruvbox.light` aren't valid names, check `bundle exec rougify style list`.

### Step 5 — Write GitHub Actions workflow

**File**: `.github/workflows/pages.yml`

Triggers on push to `main`. Uses:
1. `actions/checkout`
2. `ruby/setup-ruby` with bundler-cache
3. `bundle exec jekyll build` with `JEKYLL_ENV=production`
4. `actions/upload-pages-artifact` pointing to `_site/`
5. `actions/deploy-pages`

Permissions: `pages: write`, `id-token: write`, `contents: read`

### Step 6 — Create directory structure

```
_layouts/default.html
_layouts/home.html
_layouts/post.html
_includes/head.html
_includes/nav.html
_includes/video.html
_posts/
_drafts/
_capture/        (with .gitkeep)
assets/css/style.css
assets/css/syntax.css   (generated in Step 4)
assets/media/    (with .gitkeep)
```

### Step 7 — Write layouts and includes

**`_layouts/default.html`** — Base HTML5 shell:
- DOCTYPE, html lang, head include, nav include
- `{{ content }}` block
- Minimal footer

**`_layouts/home.html`** — Inherits default:
- Site title/tagline
- Post listing loop (`site.posts`) with date, title, excerpt, tags
- Links to individual posts

**`_layouts/post.html`** — Inherits default:
- Post title, date, tags
- `{{ content }}` for the markdown body
- Optional previous/next navigation

**`_includes/head.html`**:
- Meta charset, viewport
- SEO tag (`{% seo %}`)
- CSS links: `{{ "/assets/css/style.css" | relative_url }}`, syntax.css
- Feed link (`{% feed_meta %}`)

**`_includes/nav.html`**:
- Site title linking to home
- Minimal navigation

**`_includes/video.html`**:
- `<video>` tag with `{{ include.src }}` source

### Step 8 — Design and write CSS (`assets/css/style.css`)

**Use `/frontend-design` skill** for this step.

CSS architecture follows the pattern from `agent-lab/web/app/client/design/`:
- **Layered structure**: `@layer tokens, reset, theme, layout, components;`
- **Tokens layer**: CSS custom properties for spacing, type scale, fonts (same `--font-sans` / `--font-mono` stacks)
- **Light/dark via `prefers-color-scheme`**: System-responsive color tokens, gruvbox-inspired palette
- **Reset layer**: Modern CSS reset (box-sizing, margin collapse, media defaults)
- **Theme layer**: Body defaults from tokens

Requirements:
- Vanilla CSS, no frameworks — minimal and clean
- Light + dark mode via `prefers-color-scheme: dark/light` media queries
- Color tokens that complement gruvbox syntax highlighting in both modes
- Single-column reading layout, max-width for readability (~65ch)
- Typography-first: clean hierarchy for h1-h6, code blocks, tables, blockquotes
- Responsive (mobile-friendly)
- Style the post listing on home page
- Style tags/categories display
- Style code blocks (wrapping, scrolling for wide blocks)
- Style tables (the concept posts have many comparison tables)
- Keep it minimal — effective, clean, professional. Not fancy.

### Step 9 — Write `index.md`

```yaml
---
layout: home
title: Home
---
```

Brief intro text (or none — let the post listing speak).

### Step 10 — Create seed posts

**Seed post 1 — Regular update** (`_posts/2026-02-07-introducing-the-dev-blog.md`):
Describes the dev-blog strategy — why we're capturing weekly development efforts, how they'll be communicated to the broader group, and how the blog demonstrates capabilities as they emerge. Written in the same voice as the shadow-clone-update samples (professional, narrative, strategic framing). Frontmatter:
```yaml
---
layout: post
title: "Introducing the Dev Blog"
date: 2026-02-07
tags: [dev-blog, workflow, communication]
category: update
excerpt: "Establishing a structured approach to capturing and communicating weekly development progress."
---
```

**Seed post 2 — Concept** (`_posts/2026-02-06-dev-blog-architecture.md`):
Describes the dev-blog system architecture itself — Jekyll on GitHub Pages, vanilla CSS, GitHub Actions deployment, capture/draft/publish workflow, GitHub Releases media strategy, and the planned dev-blog skill plugin. Structured like the concept samples (hierarchical sections, tables, clear technical specification). Frontmatter:
```yaml
---
layout: post
title: "Dev Blog System Architecture"
date: 2026-02-06
tags: [dev-blog, architecture, jekyll, github-pages]
category: concept
excerpt: "Architecture concept for a developer blog built on GitHub Pages with Jekyll, managed through a Claude Code skill plugin."
---
```

Both posts use content from the concept session discussions and the architecture document in `prompt.md`, adapted into their respective formats. The `.artifacts/` samples serve as voice/style references only.

### Step 11 — Write supporting files

- **`.gitignore`**: `_site/`, `.sass-cache/`, `.jekyll-cache/`, `.jekyll-metadata`, `vendor/`, `.bundle/`
- **`CLAUDE.md`**: Project-level instructions for Claude Code (blog conventions, baseurl reminder, post creation workflow)

### Step 12 — Local build verification

```bash
bundle exec jekyll build
# Verify _site/ output has correct structure
# Check that baseurl is correctly applied in generated HTML
```

Optionally `bundle exec jekyll serve --baseurl "/jaime"` if a local preview is needed.

### Step 13 — Initial commit + push

```bash
git add -A
git commit -m "Initial blog scaffold with Jekyll 4.x and seed posts"
git branch -M main
git push -u origin main
```

### Step 14 — Enable GitHub Pages

Configure the repo to deploy from GitHub Actions:
```bash
gh api repos/tailored-agentic-units/jaime/pages \
  --method POST \
  --field build_type=workflow
```

Or configure manually: Settings → Pages → Source: GitHub Actions.

### Step 15 — Verify deployment

Wait for the Actions workflow to complete, then verify `https://tailored-agentic-units.github.io/jaime/` renders correctly.

---

## Verification

1. `bundle exec jekyll build` succeeds with zero errors
2. `_site/index.html` exists and lists both seed posts
3. `_site/` contains correctly rendered HTML for both posts
4. All asset paths include `/jaime/` prefix
5. GitHub Actions workflow completes successfully
6. Live site at `https://tailored-agentic-units.github.io/jaime/` renders correctly
7. Both seed posts are readable with proper syntax highlighting, tables, and formatting

## Deferred to Post-Blog (same session or follow-on)

Once the blog is live and the site design is finalized, we'll have all the concrete details needed to build the skill:

- `.claude/skills/dev-blog/` skill plugin with sub-command router (SKILL.md, commands/, context/)
- `capture` command — stage entries into `_capture/[date]-[slug]/` buckets
- `draft` command — generate rough draft from a bucket
- `publish` command — finalize draft, upload media to GitHub Release, promote to `_posts/`, push
- `calibrate` command — style profiling from writing samples
- GitHub Releases media strategy implementation
- Additional layouts if content types diverge during use
- Custom domain setup
- Concept document: formalize into `.claude/context/concepts/dev-blog.md`
