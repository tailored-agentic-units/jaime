# Capture — Entry Staging

## Purpose

Stage a development entry into a named capture bucket for later drafting. Captures accumulate throughout a development period and provide the structured foundation for a blog post.

## Workflow

### Step 1: Determine Bucket

Derive the bucket name from the argument slug:

- Pattern: `YYYY-MM-DD-{slug}` (date of first capture, slug from argument)
- Location: `_capture/{bucket}/`
- Example: `_capture/2026-02-14-kernel-memory-subsystem/`

If the bucket does not exist, create it:

```bash
mkdir -p _capture/YYYY-MM-DD-{slug}
```

If the bucket already exists (slug matches an existing bucket regardless of date prefix), add to the existing bucket.

### Step 2: Determine Sequence Number

Count existing `.md` files in the bucket directory. The next entry gets the sequence number `max + 1`, zero-padded to 2 digits.

Example: if `01-scaffold.md` and `02-css-layers.md` exist, the next entry is `03-`.

### Step 3: Create Entry

Ask the user what they want to capture. Collect:
- **Short title** — descriptive slug for the filename (e.g., `hub-messaging`)
- **Description** — what was accomplished and why it matters
- **Media** — optional screenshots, videos, or other files

Write the entry to `_capture/{bucket}/NN-{descriptor}.md`:

```markdown
## [Short title]

[Description of what was done and why it matters]

[Optional: media references]
```

### Step 4: Stage Media

If the user provides media files (screenshots, videos, diagrams):

1. Create the media subdirectory if needed:
   ```bash
   mkdir -p _capture/{bucket}/media
   ```
2. Copy or move media files into `_capture/{bucket}/media/`
3. Reference media in the entry using relative paths:
   ```markdown
   ![description](media/filename.png)
   ```

## Entry Format

Each capture entry is a standalone markdown file with minimal structure:

```markdown
## [Short title]

[1-3 paragraphs describing what was accomplished, why it matters,
and any relevant technical details or links]

[Optional: ![screenshot](media/screenshot.png)]
[Optional: links to PRs, issues, or files]
```

Entries should be brief — they are raw material for drafting, not finished prose. Capture the "what" and "why" while it's fresh; the `draft` command handles voice and structure.

## Bucket Naming

| Convention | Example |
|-----------|---------|
| Date prefix | `2026-02-14` (date of first capture in the bucket) |
| Slug | Descriptive, kebab-case (e.g., `kernel-memory-subsystem`) |
| Full bucket | `2026-02-14-kernel-memory-subsystem` |

## Outcomes

- New entry staged in the capture bucket
- Optional media files stored alongside the entry
- Bucket is tracked in git (the `_capture/` directory is in the repo but excluded from the Jekyll build)
