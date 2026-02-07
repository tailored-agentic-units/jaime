# Plan: Local Dev Blog Skill

## Context

The blog's capture/draft/publish workflow is documented in `.claude/context/dev-blog-skill.md` but has no automation yet — posts have been authored manually. This plan creates a local Claude Code skill at `.claude/skills/dev-blog/` that implements the full workflow as a `/dev-blog` slash command with sub-command routing.

The skill follows the patterns established by `tau:dev-workflow` — a SKILL.md with frontmatter routing, brief command descriptions, detailed workflows in `references/`, and template files for blog initialization.

---

## File Structure

```
.claude/skills/dev-blog/
├── SKILL.md
├── references/
│   ├── init.md
│   ├── themes.md
│   ├── calibrate.md
│   ├── capture.md
│   ├── draft.md
│   └── publish.md
└── template/
    ├── _config.yml
    ├── _layouts/
    │   ├── default.html
    │   ├── home.html
    │   └── post.html
    ├── _includes/
    │   ├── head.html
    │   ├── nav.html
    │   └── video.html
    ├── assets/css/
    │   ├── index.css
    │   ├── tokens.css
    │   ├── reset.css
    │   ├── theme.css
    │   ├── layout.css
    │   └── components.css
    ├── github-workflows/
    │   └── pages.yml
    ├── Gemfile
    ├── .gitignore
    ├── index.md
    └── CLAUDE.md
```

Total: 7 skill files + 18 template files = 25 files.

Note: `template/.github/` can't use that path directly because `.github` is hidden and may behave oddly in skill discovery. Use `template/github-workflows/pages.yml` and the init workflow places it at `.github/workflows/pages.yml`.

---

## SKILL.md

### Frontmatter

```yaml
---
name: dev-blog
argument-hint: "[init | calibrate | capture <slug> | draft <bucket> | publish <draft>]"
description: >
  REQUIRED for dev blog authoring workflow operations.
  Use when the user asks to "set up my blog", "initialize blog",
  "calibrate my writing style", "capture this", "draft from bucket",
  "draft a post", "publish the post", "publish draft",
  or any blog authoring activity.
  Triggers: init, calibrate, capture, draft, publish, blog post,
  writing style, style profile, capture bucket, blog workflow,
  initialize blog, set up blog.

  When this skill is invoked, identify the command from the argument
  (init, calibrate, capture, draft, or publish) and follow the
  corresponding workflow in the references/ directory. Load the style
  profile from .claude/context/style-profile.md for draft and publish
  commands.
allowed-tools:
  - "Bash(bundle *)"
  - "Bash(gem install *)"
  - "Bash(gh release *)"
  - "Bash(gh api *)"
  - "Bash(git *)"
  - "Bash(mkdir *)"
  - "Bash(pkill *)"
  - "Bash(ruby *)"
  - "Bash(rougify *)"
  - "Bash(mise *)"
  - "Bash(which *)"
---
```

### Body Sections

1. **# Dev Blog Workflow**

2. **## When This Skill Must Be Used** — bulleted trigger list

3. **## Commands** — invocation pattern table:

   | Pattern | Command | Description |
   |---------|---------|-------------|
   | `/dev-blog init` | Init | Scaffold a new blog with Jekyll, CSS, and GitHub Pages |
   | `/dev-blog calibrate` | Calibrate | Generate/update style profile from writing samples |
   | `/dev-blog capture <slug>` | Capture | Stage entry into a capture bucket |
   | `/dev-blog draft <bucket>` | Draft | Generate rough draft from captured entries |
   | `/dev-blog publish <draft>` | Publish | Finalize draft, release media, deploy |

4. **## Init Metadata** — table of required metadata collected during init:

   | Field | Required | Default | Description |
   |-------|----------|---------|-------------|
   | Repository name | Yes | — | Used for baseurl |
   | Repository owner | Yes | — | GitHub org or user |
   | Site title | Yes | — | Displayed in nav and page titles |
   | Author name | Yes | — | Footer copyright and meta tags |
   | Pages domain | No | `{owner}.github.io` | Custom domain or GitHub default |
   | Dark theme | Yes | — | Rouge-compatible theme name for dark mode (see [themes.md](references/themes.md)) |
   | Light theme | Yes | — | Rouge-compatible theme name for light mode (see [themes.md](references/themes.md)) |

5. **## Blog Configuration** — key project details: content types (update/concept), frontmatter schema, media strategy (GitHub Releases), URL structure. Condensed from dev-blog-skill.md.

6. **## Style Profile** — `.claude/context/style-profile.md`, role in draft/publish, calibrate generates it.

7. **## References** — links to each workflow file.

Target: ~200 lines.

---

## references/init.md

### Purpose
Scaffold a new dev blog with Jekyll, CSS cascade layers, and GitHub Pages deployment.

### Workflow

**Step 1: Validate Ruby**
- Run `which ruby` to check for Ruby installation
- If not found, recommend mise:
  ```
  mise install ruby
  mise use -g ruby
  ```
- Verify Ruby version is 4.x+ (`ruby --version`)

**Step 2: Collect metadata**
- Repository name, owner, site title, author name
- Pages domain (default: `{owner}.github.io`)
- Dark theme preset, light theme preset (show available presets from `templates/presets/`)

**Step 3: Scaffold Jekyll site**
- `gem install jekyll bundler` (if not installed)
- Create target directory
- Copy template files from `templates/` to the target, substituting placeholders:
  - `_config.yml`: `{{SITE_TITLE}}`, `{{SITE_DESCRIPTION}}`, `{{AUTHOR}}`, `{{URL}}`, `{{BASEURL}}`
  - `CLAUDE.md`: `{{SITE_TITLE}}`, `{{BASEURL}}`, `{{URL}}`
  - `github-workflows/pages.yml` → `.github/workflows/pages.yml`
- All other template files are static — copy verbatim

**Step 4: Generate themed CSS**
- Follow the theme validation workflow in [themes.md](references/themes.md)
- Validate user-supplied dark and light theme names against `rougify style list`
- If a theme name is invalid, show the incompatible name, list available options, and ask user to pick again
- Generate `syntax.css` using `rougify style <theme>` for each mode, wrapped in `@layer syntax` with `prefers-color-scheme` media queries
- Write `tokens.css` using the template with color values derived from the Rouge theme output (extract background/foreground from generated CSS)

**Step 5: Initialize git and install dependencies**
- `git init`, create `.gitignore`
- `bundle install`
- `bundle exec jekyll build` to verify
- `bundle exec jekyll serve` to preview

**Step 6: Create supporting directories**
- `_posts/`, `_drafts/`, `_capture/`
- `.claude/context/`, `.claude/plans/`, `.claude/settings.json`

---

## Template Files

### Parameterized (have `{{PLACEHOLDER}}` variables)

| File | Placeholders |
|------|-------------|
| `template/_config.yml` | `{{SITE_TITLE}}`, `{{SITE_DESCRIPTION}}`, `{{AUTHOR}}`, `{{URL}}`, `{{BASEURL}}` |
| `template/CLAUDE.md` | `{{SITE_TITLE}}`, `{{BASEURL}}`, `{{URL}}`, `{{REPO_OWNER}}`, `{{REPO_NAME}}` |
| `template/assets/css/tokens.css` | Color tokens — init generates these from the Rouge theme output |

### Static (verbatim copy)

All layouts, includes, CSS (except tokens.css), Gemfile, .gitignore, index.md, pages.yml — copied directly from the existing blog. These are identical for every blog instance.

---

## references/themes.md

### Purpose
Theme validation and syntax CSS generation reference for the init command.

### Available Themes
Run `rougify style list` to get the current list of Rouge-compatible themes. If the user does not supply a dark or light theme, run this command and present the available options.

### Validation Workflow
1. Run `rougify style list` to get available theme names
2. Check if the user-supplied dark theme name is in the list
3. Check if the user-supplied light theme name is in the list
4. If either is invalid, display the incompatible name and the full list of available options, then ask the user to select a valid theme

### Syntax CSS Generation
Once both themes are validated:
```bash
rougify style <dark-theme> > /tmp/syntax-dark.css
rougify style <light-theme> > /tmp/syntax-light.css
```

Wrap the output into a single `syntax.css` file:
```css
@layer syntax {
  @media (prefers-color-scheme: dark) {
    /* contents of syntax-dark.css */
  }

  @media (prefers-color-scheme: light) {
    /* contents of syntax-light.css */
  }
}
```

### Token Derivation
Extract the primary background and foreground colors from the generated Rouge CSS (`.highlight` background-color and color properties) to inform the `tokens.css` color palette. The user can customize the full palette after init.

---

## references/calibrate.md

### Purpose
Generate or update the writing style profile at `.claude/context/style-profile.md`.

### Workflow
1. Collect writing samples — user provides a directory path or specific files (default: `.artifacts/` if it exists)
2. Analyze samples for: tone, register, sentence structure, cadence, vocabulary fingerprint, formatting conventions, structural patterns, audience awareness
3. Generate the style profile following the established section schema:
   - Voice Characteristics (Tone, Register)
   - Sentence Structure (Patterns, Paragraph Construction, Cadence)
   - Vocabulary Fingerprint (Technical-to-Conversational Ratio, Characteristic Phrases, Words to Favor, Words to Avoid)
   - Formatting Conventions (Update Posts, Concept Documents)
   - Structural Patterns (Update Posts, Concept Documents)
   - Audience Awareness
4. Write to `.claude/context/style-profile.md`

---

## references/capture.md

### Purpose
Stage a development entry into a named capture bucket for later drafting.

### Workflow

1. **Determine bucket** — derive from argument slug. If bucket `_capture/YYYY-MM-DD-{slug}/` doesn't exist, create it (date = today).
2. **Determine sequence number** — count existing `.md` files in bucket, next = max + 1, zero-padded to 2 digits.
3. **Create entry** — `_capture/{bucket}/NN-{descriptor}.md`:
   ```markdown
   ## [Short title]

   [Description of what was done and why it matters]

   [Optional: media references like ![screenshot](media/filename.png)]
   ```
4. **Stage media** — if user provides screenshots/videos, copy to `_capture/{bucket}/media/`.

### Bucket naming
- Pattern: `YYYY-MM-DD-slug` (date of first capture, slug from argument)
- Example: `2026-02-14-kernel-memory-subsystem`

---

## references/draft.md

### Purpose
Generate a rough draft from captured entries in a bucket.

### Prerequisites
- Style profile must exist at `.claude/context/style-profile.md`. If missing, trigger calibrate first.

### Workflow

1. **Read the bucket** — list all entries in `_capture/{bucket}/` ordered by sequence number prefix.
2. **Determine content type** — ask user: update or concept. Sets category and structural pattern.
3. **Load style profile** — read `.claude/context/style-profile.md` for voice calibration.
4. **Generate draft** — compose the post following the style profile's structural patterns:
   - **Update**: Context-setting intro → topic sections (what/why/evidence) → forward-looking close
   - **Concept**: Title — Qualifier → problem/current state → solution → design principles → architecture → key principles
5. **Write frontmatter** — layout, title, date (placeholder), tags, category, excerpt.
6. **Copy media** — copy `_capture/{bucket}/media/*` to `assets/media/`. Update references in draft to `{{ "/assets/media/filename" | relative_url }}`.
7. **Write draft** — save to `_drafts/{slug}.md`.
8. **Preview** — `bundle exec jekyll serve --drafts`, take Playwright screenshot.

---

## references/publish.md

### Purpose
Finalize a draft and deploy it to the live blog.

### Workflow

1. **Select draft** — identify from argument. Read `_drafts/{slug}.md`.
2. **Review** — display content for user confirmation. Preview with `bundle exec jekyll serve --drafts`.
3. **Set date** — update frontmatter `date` to publish datetime (using HH:MM:SS for ordering).
4. **Promote** — move from `_drafts/{slug}.md` to `_posts/YYYY-MM-DD-{slug}.md`.
5. **Create release** (if media exists):
   - Tag: `post/{slug}`
   - Upload media from `assets/media/` as release assets via `gh release create`
   - Rewrite media paths: `assets/media/{file}` → `https://github.com/{owner}/{repo}/releases/download/post/{slug}/{file}`
6. **Build and verify** — `bundle exec jekyll build`, serve, Playwright screenshot.
7. **Stage for review** — `git add` the post file. Do NOT commit or push. Display diff and ask user to confirm before committing.

### Git workflow
Publish stages changes but defers commit/push to the user. Push to `main` triggers GitHub Actions deployment.

---

## Settings Update

Add `"Skill(dev-blog)"` to `.claude/settings.json` permissions array (alphabetical order).

---

## Verification

1. Confirm skill appears in `/` autocomplete menu
2. `/dev-blog` with no argument — skill loads, shows available commands
3. Verify template files are readable and contain valid content
4. Spot-check a preset file has all required token values
