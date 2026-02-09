# Publish — Finalization and Deployment

## Purpose

Finalize a draft and prepare it for deployment to the live blog. Handles date stamping, media release creation, path rewriting, and build verification.

## Prerequisites

- Draft must exist at `_drafts/{slug}.md`
- `gh` CLI must be authenticated for media release creation
- Style profile should exist (for any final voice adjustments)

## Workflow

### Step 1: Select Draft

Identify the draft from the command argument. If no argument is provided, list available drafts in `_drafts/` and ask the user to select one.

Read the draft content for review.

### Step 2: Review

Display the draft content and build a preview:

```bash
bundle exec jekyll serve --drafts
```

Take a Playwright screenshot for visual confirmation. Save to `_capture/.playwright/publish-preview.png`.

Ask the user to confirm the draft is ready for publication. If changes are needed, make them and re-preview.

### Step 3: Set Publish Date

Update the frontmatter `date` field to the publish datetime:

```yaml
date: YYYY-MM-DD HH:MM:SS
```

The time component controls ordering among same-day posts (newest at top on the home page). Use the current time, or ask the user if a specific time is preferred.

### Step 4: Promote to Posts

Move the draft to `_posts/` with the date prefix:

```bash
mv _drafts/{slug}.md _posts/YYYY-MM-DD-{slug}.md
```

### Step 5: Create Media Release

If the post references media files in `assets/media/`:

1. **Create a GitHub Release** tagged `post/{slug}`:
   ```bash
   gh release create "post/{slug}" --title "{post-title}" --notes "Media assets for blog post: {title}"
   ```

2. **Upload media as release assets**:
   ```bash
   gh release upload "post/{slug}" assets/media/{file1} assets/media/{file2} ...
   ```

3. **Rewrite media paths** in the post from local staging to release download URLs:
   ```
   assets/media/{filename}
   ```
   becomes:
   ```
   https://github.com/{owner}/{repo}/releases/download/post/{slug}/{filename}
   ```

   Also rewrite any Liquid `relative_url` filter references:
   ```
   {{ "/assets/media/{filename}" | relative_url }}
   ```
   becomes the full release download URL.

If the post has no media references, skip this step entirely.

### Step 6: Build and Verify

Stop any running Jekyll server, then build and serve:

```bash
pkill -f jekyll 2>/dev/null
bundle exec jekyll build
bundle exec jekyll serve --detach
```

Verify:
- Post appears on the home page at the correct position
- Post renders correctly at its URL
- All links resolve (cross-references via `post_url`, external links)
- Media loads from release URLs (if applicable)

Take Playwright screenshots of the home page and the published post. Save to `_capture/.playwright/`.

### Step 7: Stage for Review

Stage the post file for git but do **NOT** commit or push automatically:

```bash
git add _posts/YYYY-MM-DD-{slug}.md
```

Display the staged diff:

```bash
git diff --cached
```

Ask the user to confirm before committing. Push to `main` triggers the GitHub Actions deployment pipeline, so the commit/push decision belongs to the user.

## Git Workflow

The publish command stages changes but defers commit and push to the user. This is deliberate:

- Push to `main` triggers `.github/workflows/pages.yml` (automatic deployment)
- The user should review the staged changes before triggering deployment
- If additional edits are needed after staging, unstage and modify

## Outcomes

- Post promoted from `_drafts/` to `_posts/` with publish datetime
- Media uploaded to GitHub Release (if applicable)
- Media paths rewritten to stable release download URLs
- Build verified locally with Playwright screenshots
- Changes staged in git, awaiting user confirmation to commit and push
