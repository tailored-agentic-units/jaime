# Init — Blog Scaffolding

## Purpose

Scaffold a new dev blog with Jekyll 4.x, CSS cascade layers, and GitHub Pages deployment.

## Prerequisites

### Ruby Validation

Check for Ruby installation before proceeding:

```bash
which ruby
```

If Ruby is not found, recommend [mise](https://mise.jdx.dev/) for installation:

```bash
mise install ruby
mise use -g ruby
```

After installation, verify the version:

```bash
ruby --version
```

Ruby 4.x is required. Jekyll 4.x and Bundler should be installed via gem:

```bash
gem install jekyll bundler
```

## Workflow

### Step 1: Collect Metadata

Use AskUserQuestion to collect the required init metadata. See the Init Metadata table in SKILL.md for the full list of fields.

Required:
- Repository name (used as baseurl)
- Repository owner (GitHub org or username)
- Site title (nav header and page titles)
- Author name (footer copyright and meta tags)
- Dark theme (Rouge theme name — see [themes.md](../references/themes.md))
- Light theme (Rouge theme name — see [themes.md](../references/themes.md))

Optional:
- Pages domain (defaults to `{owner}.github.io`)

Derive computed values:
- `baseurl` = `/{repository-name}`
- `url` = `https://{pages-domain}`
- `description` = ask user for a one-line site description

### Step 2: Scaffold Directory Structure

Create the target directory and subdirectories:

```
{repo-name}/
├── _posts/
├── _drafts/
├── _capture/
├── _layouts/
├── _includes/
├── assets/css/
├── .github/workflows/
└── .claude/
    ├── context/
    ├── plans/
    └── skills/
```

### Step 3: Copy Source Files

Read files from the **source blog** (the repo containing this skill) and write them to the target. The source blog is the canonical reference — no separate templates are maintained.

**Adapted files** — read from source, substitute values:

| Source File | Substitutions |
|-------------|--------------|
| `_config.yml` | Replace `title`, `description`, `author`, `url`, `baseurl` values with collected metadata. Keep all other config (plugins, pagination, autopages, exclude list) as-is. |
| `.claude/CLAUDE.md` | Replace the hosted URL (`https://tailored-agentic-units.github.io/jaime/`) with the new URL. Replace baseurl `/jaime` references with new baseurl. Replace repo-specific references. |

**Verbatim files** — copy directly from the source blog:

- `_layouts/default.html` → `_layouts/`
- `_layouts/home.html` → `_layouts/`
- `_layouts/post.html` → `_layouts/`
- `_layouts/category.html` → `_layouts/`
- `_includes/head.html` → `_includes/`
- `_includes/nav.html` → `_includes/`
- `_includes/video.html` → `_includes/`
- `_includes/pagination.html` → `_includes/`
- `assets/css/index.css` → `assets/css/`
- `assets/css/reset.css` → `assets/css/`
- `assets/css/theme.css` → `assets/css/`
- `assets/css/layout.css` → `assets/css/`
- `assets/css/components.css` → `assets/css/`
- `.gitignore` → `.gitignore`
- `index.md` → `index.md`
- `.github/workflows/pages.yml` → `.github/workflows/`

### Step 4: Generate Themed CSS

Follow the [themes.md](../references/themes.md) workflow:

1. Validate the dark and light theme names against `rougify style list`
2. Generate `syntax.css` with both themes wrapped in `@layer syntax` and `prefers-color-scheme` media queries
3. Generate `tokens.css` — read the source blog's `assets/css/tokens.css`, replace the color values in the `prefers-color-scheme: dark` and `prefers-color-scheme: light` blocks with colors derived from the chosen Rouge themes. Keep the structural tokens (fonts, spacing, radius, widths) unchanged.

### Step 5: Generate Gemfile and Install Dependencies

Generate the Gemfile dynamically to ensure the latest versions of all gems:

```bash
cd {target-directory}
bundle init
bundle add jekyll
bundle add jekyll-feed jekyll-paginate-v2 jekyll-seo-tag jekyll-sitemap --group jekyll_plugins
```

### Step 6: Initialize Git and Build

```bash
git init
bundle exec jekyll build
```

Verify the build succeeds, then preview:

```bash
bundle exec jekyll serve
```

### Step 7: Create Settings

Write `.claude/settings.json`:

```json
{
  "plansDirectory": "./.claude/plans",
  "permissions": {
    "allow": [
      "Skill(dev-blog)"
    ]
  }
}
```

## Outcomes

- Fully functional Jekyll blog ready for `bundle exec jekyll serve`
- GitHub Actions workflow for automatic deployment on push to `main`
- CSS cascade layers with themed dark/light syntax highlighting
- Pagefind search with category and tag filtering
- Category pages auto-generated via jekyll-paginate-v2 autopages
- Claude Code project configuration with skill permissions
- Empty `_posts/`, `_drafts/`, and `_capture/` directories ready for content
