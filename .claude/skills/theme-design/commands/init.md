# Command: init

Initialize a new theme with an empty spec scaffold and stripped CSS layers.

## Usage

```
design init <name>
```

## Workflow

1. **Create theme directory** — `.claude/skills/theme-design/themes/<name>/`
2. **Scaffold spec.yaml** — use the schema from `references/spec-schema.md` with empty values. Populate `theme.name` and `theme.description` from the user.
3. **Strip CSS layers** — replace all CSS layer files in `assets/css/` with empty `@layer` wrappers so foundational layers can be applied and observed incrementally. Preserve `index.css` cascade declaration.
4. **Copy stress-test post** — copy `references/stress-test-post.md` into `_posts/` with `published: false` for visual validation during design.
5. **Start dev server** — `bundle exec jekyll serve --drafts --unpublished --port 4000`

## After Init

Proceed through the layer sequence using `design layer <name>`. Each layer is discussed, applied, visually validated, and committed before the next is introduced.
