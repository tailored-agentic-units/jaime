# Design Specification Schema

Every theme uses this fixed YAML structure. The spec is both human-readable reference and machine-parseable contract. Sections are ordered to match the CSS cascade layer sequence.

```yaml
theme:
  name: ""
  description: ""
  references: []
  influences: []

reset:
  box_model: ""
  margin: ""
  padding: ""
  body:
    min_height: ""
    line_height: 0
    font_size: ""
    font_smoothing: ""
    text_rendering: ""
  form_inheritance: ""
  media_defaults: ""
  text_wrapping:
    paragraphs: ""
    headings: ""
  list_normalization: ""  # preserve browser defaults or strip — component layer handles exceptions
  anchor_normalization: ""
  table_normalization: ""
  scroll_behavior: ""
  explicit_exclusions: []  # what the reset intentionally does NOT do

tokens:
  font_stacks: {}    # keyed by category: sans-serif, monospace, etc.
  type_scale:
    ratio: ""        # e.g., "minor third (1.2)"
    steps: ""        # computed rem values
  line_heights: ""   # named steps: tight, snug, normal, relaxed
  font_weights: ""   # normal, bold (+ others if needed)
  code_size: ""      # mono in sans context typically needs reduction
  letter_spacing: "" # tight, wide

typography:
  voices: {}         # keyed by role: prose, structural
  heading_scale: {}  # keyed by h1-h4: size, weight, line-height
  chrome: {}         # keyed by element: nav, time, footer
  code:
    all: ""          # font, size, line-height
  table:
    th: ""           # alignment
  links: ""          # font family assignment
  unit_convention: ""

colors:
  approach: ""       # dark-first or light-first
  color_scheme: ""   # e.g., "dark light"
  temperature: ""    # overall neutral temperature
  architecture:
    principle: ""    # e.g., "theme colors are source of truth, base16 derives from them"
    flow: ""         # dependency direction
    benefit: ""      # why this architecture
  theme_colors:
    dark: {}         # chrome, interactive, emphasis — direct hex values
    light: {}        # chrome, interactive, emphasis — direct hex values
  semantic_surfaces: {}  # base, raised, overlay — mapped to base00-02
  semantic_text: {}      # primary, secondary, muted — mapped to base03-05
  dark_neutrals: {}      # base00-07 with hex + description
  dark_accents: {}       # base08-0F — hex or var(--color-*) + description
  light_neutrals: {}     # base00-07 with hex + description
  light_accents: {}      # base08-0F — may remap slots vs dark mode
  element_colors: {}     # per-element color assignments
  syntax_mapping: {}     # Rouge class → base16 slot mapping

layout:
  spacing_scale:
    system: ""       # e.g., "golden ratio (φ = 1.618)"
    baseline: ""     # which step = 1rem
    steps: {}        # space-N: value
  content_width: ""
  page_padding: ""
  vertical_rhythm:
    headings: {}     # per-heading margin-block
    blocks: ""       # sibling block spacing
    code_blocks: ""
    section_dividers: ""
  page_structure: {} # header, main, footer padding
  responsive: {}     # breakpoint rules

borders:
  tokens: {}         # border_width, border_width_thick, border_color, border_color_muted, radius
  elements: {}       # per-element border treatments

transitions:
  tokens: {}         # duration_N, ease_default
  elements: {}       # per-element transition assignments
  scope: ""          # what gets transitions

components: {}       # all UI components — populated after foundational layers are validated
```
