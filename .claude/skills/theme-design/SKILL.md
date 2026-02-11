---
name: theme-design
argument-hint: "[init <name> | load <spec-dir> [target-dir] | layer <name> | enhance [layer|all] | export-syntax <format> | spec]"
description: >
  Design themes built bottom-up through validated foundational layers.
  Each layer is applied and verified in isolation before the next is
  introduced. No layer references concepts from a later layer.
  Triggers: theme design, design layer, design init, design load,
  design enhance, design spec, export syntax, color palette,
  typography system.
allowed-tools:
  - "Bash(bundle *)"
  - "Bash(git *)"
  - "Bash(mkdir *)"
  - "Bash(pkill *)"
  - "Bash(rougify *)"
  - Edit
  - Glob
  - Grep
  - Read
  - Write
  - "mcp__playwright__*"
---

# Theme Design Skill

Build design themes bottom-up through validated foundational layers. Each layer is applied in isolation, verified visually, and committed before the next is introduced.

## When This Skill Applies

Use this skill when designing or modifying the blog's visual theme. This includes initializing a new theme, applying individual design layers, enhancing existing layers, exporting syntax highlighting themes, or viewing the design specification.

## Philosophy

Design themes are built bottom-up. Each layer is validated in isolation before the next is introduced. No layer should reference concepts from a later layer. Cohesion comes from the system, not from individual component decisions being clever.

**Key lesson (from Industrial Tech):** Do not jump from philosophy to component-level decisions. Do not make opinionated component choices before the foundational system is tested. Each layer must be visually validated before the next is introduced.

**Inheritance principle:** Layers inherit the foundation — only override when deviating. If the reset establishes a baseline value (e.g. `line-height: 1.6`), higher layers should not reassert it with a token just because a token exists. Tokens exist for when properties need to deviate from the default. Let the foundation set the standard.

## Commands

### `design init <name>`

Initialize a new theme. Establishes name, description, and reference materials. Creates the spec document scaffold under `themes/<name>/spec.yaml`. Strips all existing theme CSS back to empty `@layer` wrappers so foundational layers can be applied and observed incrementally.

Also copies the stress-test post template from `references/stress-test-post.md` into `_posts/` with `published: false` for visual validation.

### `design load <spec-dir> [target-dir]`

Load a theme from its spec into CSS assets — like loading a save file. Reads `spec.yaml` from the given theme directory and materializes validated decisions into the corresponding CSS layer files. Empty spec sections produce empty `@layer` wrappers. Non-destructive by default — warns before overwriting existing files.

See `functions/load.md` for full specification.

### `design layer <name>`

Apply a specific foundational layer to the blog. Generates the CSS for that layer based on the spec, creates/updates the appropriate cascade layer file. Layers must be applied in order. After applying, launch the site in Playwright for visual verification.

### `design enhance [layer|all]`

Refine an existing layer or the full composition. This is the iterative loop used across sessions to drive the design toward its final form. Enhancement sessions should be explicitly scoped.

### `design export-syntax <format>`

Generate syntax highlighting theme files from the Base16 palette. Supported formats: rouge (CSS), neovim (lua), vscode (JSON tokenColors).

### `design spec`

View or regenerate the current design specification document.

## Layer Sequence (strict order)

Each design layer maps 1:1 to a CSS cascade layer and its own file. Applied in order — each layer is visually validated before the next is introduced.

| # | Layer | CSS File | What It Addresses |
|---|-------|---------|-------------------|
| 1 | reset | `reset.css` | Box model, margin/padding strip, typography baseline, media defaults |
| 2 | tokens | `tokens.css` | CSS custom properties — populated incrementally as each layer is designed |
| 3 | typography | `typography.css` | Typeface stacks, size scale, weights, line-heights, letter-spacing |
| 4 | colors | `colors.css` | Surface colors, content foreground, semantic hues, Base16 syntax palette |
| 5 | layout | `layout.css` | Base unit, spacing scale, content width, vertical rhythm, breakpoints |
| 6 | borders | `borders.css` | Stroke weights, corner treatment, dividers, surface containment |
| 7 | transitions | `transitions.css` | Motion timing, easing, reduced-motion |
| 8 | components | `components.css` | All UI elements — nav, post cards, tags, code blocks, pagination |

**Reset** comes first to establish a clean baseline. **Tokens** follows immediately — it is not a standalone design phase but a living accumulator of CSS custom properties that gets populated as each subsequent layer is designed. Each design layer may introduce new tokens as decisions are validated. **Components** comes last and styles all include-level markup (`.nav-title`, `.post-card`, `.tag`, etc.) using tokens.

## CSS Cascade Layer Structure

Eight CSS cascade layers declared in `assets/css/index.css`, one file per layer:

```css
@layer reset, tokens, typography, colors, layout, borders, transitions, components;
```

## Visual Validation Workflow

**Design session server command:**

```bash
bundle exec jekyll serve --drafts --unpublished --port 4000
```

The `--unpublished` flag is required because the stress-test post uses `published: false` to stay off the production site. Without it, the post won't be served.

**For each layer:**

1. **Discuss the approach first** — present the design intent, key decisions, and tradeoffs before writing any CSS. This avoids wasted iteration on a rejected direction.
2. Write the CSS based on the agreed approach
3. Inspect via Playwright (screenshot / snapshot) — check both dark and light modes
4. Iterate until satisfied
5. Update tokens.css with any new CSS custom properties identified
6. Update the spec with validated decisions

## References

- `references/base16-schema.md` — Base16 slot schema and semantic roles
- `references/spec-schema.md` — Design spec YAML schema
- `references/syntax-export.md` — Format-specific export mappings
- `references/stress-test-post.md` — Canonical stress-test post template
