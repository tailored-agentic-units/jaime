# Themes — Rouge Theme Validation and CSS Generation

## Purpose

Validate user-supplied Rouge theme names and generate the `syntax.css` file for the blog's dark and light modes. Used by the **Quick Start** theme strategy during `init`.

## Available Themes

Run `rougify style list` to get the current list of Rouge-compatible theme names:

```bash
rougify style list
```

If the user does not supply a dark or light theme during init, run this command and present the available options for them to choose from.

## Validation Workflow

1. Run `rougify style list` to get the list of valid theme names
2. Check if the user-supplied **dark theme** name is in the list
3. Check if the user-supplied **light theme** name is in the list
4. If either theme name is **not** in the list:
   - Display the incompatible theme name
   - Show the full list of available options from `rougify style list`
   - Ask the user to select a valid theme name
   - Repeat validation until both themes are valid

## Syntax CSS Generation

Once both theme names are validated, generate the syntax stylesheet:

```bash
rougify style <dark-theme> > /tmp/syntax-dark.css
rougify style <light-theme> > /tmp/syntax-light.css
```

Combine both into a single `syntax.css` file wrapped in cascade layer and media queries:

```css
@layer syntax {
  @media (prefers-color-scheme: dark) {
    /* contents of /tmp/syntax-dark.css */
  }

  @media (prefers-color-scheme: light) {
    /* contents of /tmp/syntax-light.css */
  }
}
```

Write the result to `assets/css/syntax.css`.

## Token Derivation

The Rouge theme output includes background and foreground colors for the `.highlight` class. Extract these values to inform the `tokens.css` color palette:

- **--surface-base** — derived from `.highlight { background: ... }` (or a slightly darker variant)
- **--surface-raised** — the `.highlight { background: ... }` value itself
- **--text-primary** — derived from `.highlight { color: ... }`

These provide a starting point for the token palette. Read the source blog's `assets/css/tokens.css` and replace the color values with colors derived from the chosen Rouge themes. Keep the structural tokens (fonts, spacing, radius, widths) unchanged.

The remaining semantic tokens should be derived as complementary values from the Rouge palette:

- **--surface-overlay** — slightly lighter than `--surface-raised`
- **--text-secondary** — midpoint between `--text-primary` and `--text-muted`
- **--text-muted** — subdued foreground for comments, hints
- **--border-color** — matches `--surface-overlay`
- **--border-color-muted** — matches `--surface-raised`
- **--color-chrome** — a distinctive accent from the Rouge palette for structural elements (nav, headings, code)
- **--color-interactive** — an accent for links and interactive states
- **--color-emphasis** — an accent for highlights, tags, and callouts

The theme colors (`--color-chrome`, `--color-interactive`, `--color-emphasis`) are defined as direct hex values in `:root`. Base16 accent slots that correspond to a theme color should reference `var(--color-*)` rather than duplicating the hex value. See `base16-schema.md` for the full architecture.

The user can customize the full token palette after init by editing `assets/css/tokens.css`.

## Alternative: Custom Theme

For full creative control over the color system, use the **Custom Theme** strategy (Option B in init) instead. This invokes the `theme-design` skill for a bottom-up layer design process with visual validation at each step.
