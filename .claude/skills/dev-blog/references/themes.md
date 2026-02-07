# Themes — Rouge Theme Validation and CSS Generation

## Purpose

Validate user-supplied Rouge theme names and generate the `syntax.css` file for the blog's dark and light modes.

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

- **--bg** — derived from `.highlight { background: ... }`
- **--fg** — derived from `.highlight { color: ... }`

These provide a starting point for the token palette. The remaining tokens (--bg-soft, --bg-raised, --fg-dim, --fg-muted, --border, --accent, --link, --link-visited) should be derived as complementary values or set to reasonable defaults that pair well with the Rouge theme's palette.

The user can customize the full token palette after init by editing `assets/css/tokens.css`.
