# Dev Blog Skill — Context Document

Reference document for designing and implementing the `dev-blog` Claude Code skill plugin. Captures all decisions, conventions, and implementation details from the blog's concept and build sessions.

---

## Blog Implementation (Complete)

### Site Details

- **Repository**: `tailored-agentic-units/jaime`
- **Live URL**: `https://tailored-agentic-units.github.io/jaime/`
- **baseurl**: `/jaime` — all asset references use `| relative_url` Liquid filter
- **Stack**: Jekyll 4.x, kramdown (GFM), Rouge syntax highlighting, vanilla CSS
- **Deployment**: Push to `main` triggers `.github/workflows/pages.yml` (GitHub Actions)
- **Ruby**: 4.0, Bundler 4.x
- **gh CLI**: Authenticated as JaimeStill, org is `tailored-agentic-units`

### Content Types

Both types use the same `post` layout. Differentiation is expressed through frontmatter only.

| Type | Category | Purpose |
|------|----------|---------|
| **Update** | `update` | Weekly development progress, new capabilities, strategic direction |
| **Concept** | `concept` | Technical architecture and design documents |

### Post Frontmatter Schema

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD
tags: [tag-1, tag-2, tag-3]
category: update | concept
excerpt: "One-sentence summary for the post listing."
---
```

- `layout` is always `post`
- `date` determines sort order and URL path
- `category` is singular (`update` or `concept`), not plural
- `tags` is a YAML array
- `excerpt` displays on the home page post listing (truncated to 30 words)

### URL Structure

Jekyll generates URLs using the category permalink pattern:

```
/jaime/{category}/YYYY/MM/DD/{slug}.html
```

Examples:
- `/jaime/update/2026/02/07/introducing-the-dev-blog.html`
- `/jaime/concept/2026/02/06/dev-blog-architecture.html`

### Layout Hierarchy

```
default.html          # HTML5 shell (head include, nav include, {{ content }}, footer)
├── home.html         # Post listing (iterates site.posts with date, title, category, excerpt)
└── post.html         # Article (h1, time, tags, {{ content }})
```

### Includes

| Include | Purpose | Parameters |
|---------|---------|------------|
| `head.html` | Meta, single CSS link (`index.css`), `{% seo %}`, `{% feed_meta %}` | None |
| `nav.html` | Site title linking to home | None |
| `video.html` | Reusable `<video>` tag | `src` (URL to video file) |

Usage: `{% include video.html src="https://github.com/.../demo.mp4" %}`

### CSS Architecture

Entry point: `assets/css/index.css`

```css
@layer tokens, reset, theme, layout, components, syntax;

@import url("./tokens.css");
@import url("./reset.css");
@import url("./theme.css");
@import url("./layout.css");
@import url("./components.css");
@import url("./syntax.css");
```

Each file declares its own `@layer`. The layer order is established in `index.css`.

| File | Layer | Contents |
|------|-------|----------|
| `tokens.css` | tokens | CSS custom properties — colors (GitHub palette, light/dark), spacing scale, type scale, font stacks, radius, content-width |
| `reset.css` | reset | Modern CSS reset (box-sizing, margin removal, media defaults, text-wrap, interpolate-size) |
| `theme.css` | theme | Body defaults from tokens (font-family, background, foreground, link colors) |
| `layout.css` | layout | Page structure — content max-width (`65ch`), nav, main, footer |
| `components.css` | components | Post list, article prose, tags, tables, code blocks, blockquotes, images |
| `syntax.css` | syntax | Rouge-generated GitHub dark/light syntax highlighting |

Color tokens toggle via `prefers-color-scheme` media queries. Key values:

| Token | Dark | Light |
|-------|------|-------|
| `--bg` | `#0d1117` | `#ffffff` |
| `--bg-soft` | `#161b22` | `#f6f8fa` |
| `--bg-raised` | `#21262d` | `#eaeef2` |
| `--fg` | `#c9d1d9` | `#24292f` |
| `--fg-dim` | `#8b949e` | `#57606a` |
| `--fg-muted` | `#6e7681` | `#6e7781` |
| `--border` | `#30363d` | `#d0d7de` |
| `--accent` | `#58a6ff` | `#0969da` |
| `--link` | `#58a6ff` | `#0969da` |
| `--link-visited` | `#bc8cff` | `#8250df` |

### Directory Structure

```
jaime/
├── .github/workflows/pages.yml
├── .claude/
│   ├── CLAUDE.md                      # Project-level Claude Code instructions
│   ├── context/
│   │   ├── style-profile.md           # Generated writing voice calibration
│   │   └── dev-blog-skill.md          # This document
│   ├── plans/
│   └── settings.json
├── _config.yml
├── _layouts/
│   ├── default.html
│   ├── home.html
│   └── post.html
├── _includes/
│   ├── head.html
│   ├── nav.html
│   └── video.html
├── _posts/                            # Published markdown (date-prefixed)
├── _drafts/                           # Authoring target, excluded from production
├── _capture/                          # Capture buckets (excluded from Jekyll build)
├── assets/
│   ├── css/                           # Cascade layer files
│   └── media/                         # Gitignored — local staging only
├── .gitignore
├── .mcp.json                          # Playwright MCP config (Firefox)
├── Gemfile
├── Gemfile.lock
└── index.md
```

### Jekyll Build Excludes

The following are excluded from the Jekyll build via `_config.yml`:

```
_capture, .artifacts, .claude, .github, .playwright-mcp,
samples, .mcp.json, CLAUDE.md, Gemfile, Gemfile.lock,
README.md, node_modules, vendor
```

### Gitignored Paths

```
.artifacts, _site/, .playwright-mcp/, .sass-cache/,
.jekyll-cache/, .jekyll-metadata, vendor/, .bundle/, assets/media/
```

---

## Media Strategy

### Problem

Git history bloats with binary files. GitHub Pages does not serve Git LFS-tracked files.

### Solution

GitHub Releases as a per-post CDN.

- **Tag pattern**: `post/{slug}` (e.g., `post/introducing-the-dev-blog`)
- All media for a post uploaded as release assets
- **Stable download URLs**: `https://github.com/tailored-agentic-units/jaime/releases/download/post/{slug}/{filename}`

### Local Staging

`assets/media/` is gitignored entirely. It serves only as a local staging area during authoring. On publish, media is uploaded to a GitHub Release and the directory contents can be discarded.

### Video Include

```liquid
{% include video.html src="https://github.com/.../demo.mp4" %}
```

---

## Capture / Draft / Publish Workflow

A three-stage authoring pipeline designed around how development actually happens.

### Stage 1 — Capture

As noteworthy work is completed, a brief entry is staged into a named bucket:

```
_capture/
└── 2026-02-07-introducing-the-dev-blog/
    ├── 01-blog-scaffold.md
    ├── 02-css-architecture.md
    └── media/
        └── screenshot.png
```

- Bucket naming convention: `[YYYY-MM-DD]-[slug]`
- Entries within are ordered by sequence number prefix (`01-`, `02-`, etc.)
- Each entry includes a description with optional media references
- The `media/` subdirectory holds local media files associated with the bucket
- `_capture/` is excluded from the Jekyll build but tracked in git for traceability

### Stage 2 — Draft

The `draft` command reads all entries from a specified bucket and generates a rough draft in `_drafts/` with validated frontmatter. The style profile (`.claude/context/style-profile.md`) calibrates the writing voice. Media references point to local staging paths (`assets/media/`).

### Stage 3 — Publish

The `publish` command finalizes the draft:

1. Promotes the post from `_drafts/` to `_posts/` with the current date prefix
2. Creates a GitHub Release tagged `post/{slug}`
3. Uploads all associated media as release assets
4. Rewrites media paths in the post to release download URLs
5. Commits and pushes

---

## Skill Design (From Concept Session)

### Architecture

Single Claude Code skill with sub-command routing. The skill is a plugin — blog-specific content (posts, captures, style profile) is generated at runtime, not shipped with the skill.

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

The style profile lives at the **project level**, not within the skill. Each user generates their own via the `calibrate` command. If the profile doesn't exist when `draft` or `publish` runs, the skill should trigger calibration first.

### Key Design Principles

1. **Deterministic operations separated from non-deterministic** — style inference (calibrate) runs once, not per-post
2. **Capture incrementally, publish periodically** — matches how development actually happens
3. **Skill automates, user owns** — the plugin manages process; content, voice, and style belong to the author
4. **Media outside git** — GitHub Releases serve binary assets without bloating history
5. **Push markdown, get HTML** — the entire publishing pipeline is git-native

---

## Writing Voice

The style profile at `.claude/context/style-profile.md` was generated from analysis of 5 weekly update emails and 3 technical concept documents. Key characteristics:

- **Tone**: Professional, confident, conversational warmth over technical substance
- **Register**: Formal enough for leadership, informal enough to feel human
- **Cadence**: Short grounding statement, longer explanatory sentence, medium impact sentence
- **Vocabulary**: Favors "establish, foundational, standardize, reusable, modular"; avoids "utilize, synergy, paradigm"
- **Structure (updates)**: Context-setting intro → topic sections (what/why/links) → forward-looking close
- **Structure (concepts)**: Title → problem/current state → solution/target → design principles → architecture → implementation → key principles

See the full profile for detailed patterns, formatting conventions, and audience awareness guidance.

---

## Open Questions for Skill Planning

These items were deferred during the blog implementation session and need resolution during skill design:

1. **`init` command scope** — How much of the blog scaffold should `init` handle vs. assuming the blog already exists? The current blog was built manually. Should the skill's `init` command be able to reproduce it from scratch, or is it primarily for new users adopting the skill?

2. **Capture entry format** — What is the expected structure of a single capture entry markdown file? Just prose? Structured frontmatter? Should the skill enforce a template?

3. **Draft generation** — How should entries be ordered and combined? By sequence number prefix? Should the draft include all entries verbatim or synthesize them? What role does the style profile play in transforming captures into draft prose?

4. **Publish command — git workflow** — Should `publish` commit and push automatically, or stage changes for the user to review? The current blog deploys on push to `main`, so an auto-push would trigger deployment immediately.

5. **`calibrate` command — input sources** — The current style profile was generated from email samples in `.artifacts/`. Should `calibrate` accept a directory of samples? Specific files? How should it handle re-calibration (overwrite vs. merge)?

6. **Error recovery** — What happens if `publish` fails mid-way (e.g., release created but media upload fails)? Should there be a rollback mechanism?

7. **Bucket lifecycle** — After publishing, what happens to the `_capture/` bucket? Archive it? Delete it? Leave it for traceability?

8. **Multiple drafts from same bucket** — Can you re-draft from a bucket that already has a draft in `_drafts/`? Overwrite or create a new version?
