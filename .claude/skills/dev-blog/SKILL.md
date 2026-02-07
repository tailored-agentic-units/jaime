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
  corresponding workflow in the commands/ directory. Load the style
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

# Dev Blog Workflow

## When This Skill Must Be Used

**ALWAYS invoke this skill when the user's request involves ANY of these:**

- Initializing or setting up a new dev blog
- Calibrating or updating a writing style profile
- Capturing a development entry for a future blog post
- Drafting a blog post from captured entries
- Publishing a draft to the live blog
- Any operation involving the capture/draft/publish pipeline
- Questions about blog configuration, media strategy, or content types

## Commands

| Pattern | Command | Description |
|---------|---------|-------------|
| `/dev-blog init` | Init | Scaffold a new blog with Jekyll, CSS cascade layers, and GitHub Pages deployment |
| `/dev-blog calibrate` | Calibrate | Generate or update writing style profile from writing samples |
| `/dev-blog capture <slug>` | Capture | Stage an entry into a named capture bucket |
| `/dev-blog draft <bucket>` | Draft | Generate a rough draft from captured entries in a bucket |
| `/dev-blog publish <draft>` | Publish | Finalize a draft, release media, and deploy |

## Init Metadata

The `init` command collects the following metadata to scaffold the blog:

| Field | Required | Default | Description |
|-------|----------|---------|-------------|
| Repository name | Yes | — | GitHub repository name, used as baseurl (`/{repo}`) |
| Repository owner | Yes | — | GitHub organization or username |
| Site title | Yes | — | Displayed in nav header and page titles |
| Author name | Yes | — | Footer copyright and HTML meta tags |
| Pages domain | No | `{owner}.github.io` | Custom domain or GitHub Pages default |
| Dark theme | Yes | — | Rouge-compatible theme name for dark mode (see [themes.md](references/themes.md)) |
| Light theme | Yes | — | Rouge-compatible theme name for light mode (see [themes.md](references/themes.md)) |

## Blog Configuration

### Content Types

Both types use the same `post` layout. Differentiation is through frontmatter only.

| Type | Category | Purpose |
|------|----------|---------|
| **Update** | `update` | Development progress, new capabilities, strategic direction |
| **Concept** | `concept` | Technical architecture and design documents |

### Post Frontmatter Schema

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS
tags: [tag-1, tag-2, tag-3]
category: update | concept
excerpt: "One-sentence summary for the post listing."
---
```

- `layout` is always `post`
- `date` includes time component for same-day ordering (newest at top)
- `category` is singular (`update` or `concept`)
- `tags` is a YAML array
- `excerpt` truncated to 30 words on home page

### URL Structure

```
/{baseurl}/{category}/YYYY/MM/DD/{slug}.html
```

### Media Strategy

Git history bloats with binary files. Media is hosted via GitHub Releases:

- **Tag pattern**: `post/{slug}`
- **Download URL**: `https://github.com/{owner}/{repo}/releases/download/post/{slug}/{filename}`
- **Local staging**: `assets/media/` (gitignored) — used during drafting, rewritten to release URLs on publish

## Style Profile

The writing style profile at `.claude/context/style-profile.md` calibrates the voice for draft generation. It is produced by the `calibrate` command and consumed by `draft` and `publish`.

If the style profile does not exist when `draft` or `publish` is invoked, trigger calibration first.

## Commands

- [Init — Blog scaffolding](commands/init.md)
- [Calibrate — Style profile generation](commands/calibrate.md)
- [Capture — Entry staging](commands/capture.md)
- [Draft — Post generation from captures](commands/draft.md)
- [Publish — Finalization and deployment](commands/publish.md)

## References

- [Themes — Rouge theme validation and CSS generation](references/themes.md)
