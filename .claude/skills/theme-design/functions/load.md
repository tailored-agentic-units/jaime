# Function: load

Load a theme spec into CSS assets — equivalent to loading a save file.

## Usage

```
design load <spec-dir> [target-dir]
```

- `<spec-dir>` — path to the theme directory containing `spec.yaml` (e.g., `.claude/skills/theme-design/themes/tau-ceti/`)
- `[target-dir]` — optional path to the CSS assets directory (defaults to `assets/css/`)

## What It Does

1. **Read the spec** — parse `spec.yaml` from the given theme directory
2. **Resolve the layer structure** — identify which layers have populated sections (non-empty values beyond `{}`)
3. **Generate CSS files** — for each populated layer, materialize the spec decisions into the corresponding CSS file:
   - `reset.css` — from `reset:` section
   - `tokens.css` — from `tokens:` section
   - `typography.css` — from `typography:` section
   - `colors.css` — from `colors:` section
   - `layout.css` — from `layout:` section
   - `borders.css` — from `borders:` section
   - `transitions.css` — from `transitions:` section
   - `components.css` — from `components:` section
4. **Empty layers** — layers with `{}` in the spec get empty `@layer` wrappers
5. **Write index.css** — ensure the cascade layer declaration and imports match the layer structure
6. **Copy stress test** — if not already present, copy the stress-test post template from `references/stress-test-post.md` into `_posts/` with `published: false`

## Spec Interpretation

The spec is declarative — it describes *what* was decided, not the literal CSS. The load function translates spec decisions into CSS:

- `tokens:` entries become CSS custom properties in `:root {}`
- `typography.voices:` determines `font-family` assignments on elements
- `typography.heading_scale:` becomes heading size/weight/line-height rules
- `typography.chrome:` becomes nav/footer/time styling rules
- `typography.code:` becomes pre/code styling rules
- `colors:` entries become color custom properties and `prefers-color-scheme` media queries
- Layout, borders, transitions, and components follow similar patterns

## Behavior

- **Non-destructive by default** — if CSS files already exist with content, warn and ask before overwriting
- **Partial load** — only generates CSS for layers that have spec content; empty spec sections produce empty `@layer` wrappers
- **Validates build** — after writing files, run `bundle exec jekyll build` to verify the result compiles

## Example

```
# Load the graphic realism theme into the default assets directory
design load .claude/skills/theme-design/themes/tau-ceti/

# Load into a different directory (e.g., for comparison)
design load .claude/skills/theme-design/themes/tau-ceti/ assets/css-preview/
```
