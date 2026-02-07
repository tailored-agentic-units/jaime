# Draft — Post Generation from Captures

## Purpose

Generate a rough draft from the captured entries in a bucket, calibrated to the author's writing voice.

## Prerequisites

- **Style profile** must exist at `.claude/context/style-profile.md`. If missing, trigger the `calibrate` command first.
- **Capture bucket** must exist at `_capture/{bucket}/` with at least one entry.

## Workflow

### Step 1: Read the Bucket

List all `.md` files in `_capture/{bucket}/` ordered by sequence number prefix (01-, 02-, etc.). Read each entry to understand the scope of content captured.

### Step 2: Determine Content Type

Ask the user which content type this post should be:

| Type | Category | Structural Pattern |
|------|----------|--------------------|
| **Update** | `update` | Context-setting intro, topic sections (what/why/evidence), forward-looking close |
| **Concept** | `concept` | Title — Qualifier, problem/current state, solution, design principles, architecture, key principles |

The content type sets the `category` frontmatter value and determines which structural pattern from the style profile to follow.

### Step 3: Load Style Profile

Read `.claude/context/style-profile.md` and apply the voice calibration:

- **Tone and register** — match the author's natural voice
- **Sentence structure** — follow the cadence patterns (short/long/medium)
- **Vocabulary** — use favored words, avoid avoided words
- **Formatting conventions** — match the conventions for the selected content type
- **Structural patterns** — follow the section ordering for updates or concepts

### Step 4: Generate Draft

Compose the post from the captured entries. This is a synthesis, not a concatenation — the entries provide raw material that gets shaped into a cohesive post.

**For updates:**
1. Opening paragraph — set the context for what this period was about
2. Topic sections — organize captured entries into logical sections, each with what was done, why it matters, and links to evidence
3. Closing paragraph — forward-looking statement

**For concepts:**
1. Opening paragraph — one-sentence summary of the concept
2. Problem statement or current state
3. Target state or proposed solution
4. Design principles and philosophy
5. Architecture details (tables, diagrams, hierarchies)
6. Key principles summary

### Step 5: Write Frontmatter

Generate validated frontmatter:

```yaml
---
layout: post
title: "[Generated title]"
date: YYYY-MM-DD HH:MM:SS
tags: [tag-1, tag-2, tag-3]
category: update | concept
excerpt: "[One-sentence summary]"
---
```

- **title**: For updates, a declarative statement. For concepts, follow the `Subject — Qualifier` pattern.
- **date**: Use a placeholder date (today) with time component. The publish command sets the final datetime.
- **tags**: 3-4 lowercase, kebab-case tags relevant to the content.
- **excerpt**: Under 30 words (the home page truncation limit).

### Step 6: Copy Media

If the capture bucket has a `media/` subdirectory:

1. Create the local staging directory:
   ```bash
   mkdir -p assets/media
   ```
2. Copy media files from `_capture/{bucket}/media/` to `assets/media/`
3. Update media references in the draft to use Jekyll's `relative_url` filter:
   ```markdown
   ![description]({{ "/assets/media/filename.png" | relative_url }})
   ```

### Step 7: Write Draft

Save the draft to `_drafts/{slug}.md`. Jekyll's `_drafts/` directory is excluded from production builds but visible with `--drafts` flag.

### Step 8: Preview

Build and serve with drafts enabled:

```bash
bundle exec jekyll serve --drafts
```

Take a Playwright screenshot of the draft post for visual review. Save to `.playwright-mcp/draft-preview.png`.

Present the draft to the user for feedback. Iterate on content, structure, or voice as needed before publishing.

## Outcomes

- Draft written to `_drafts/{slug}.md` with validated frontmatter
- Media staged to `assets/media/` with correct references
- Local preview available with `--drafts` flag
- Draft ready for user review and iteration before publish
