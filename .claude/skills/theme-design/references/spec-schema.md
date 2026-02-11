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
  margin_padding: ""
  typography_baseline:
    root_font: ""
    root_line_height: 0
    root_font_size: ""
    inheritance: ""
  media_defaults: ""
  list_normalization: ""
  anchor_normalization: ""
  table_normalization: ""
  scroll_behavior: ""
  text_rendering: ""
  explicit_exclusions: []  # what the reset intentionally does NOT do

tokens: {}  # CSS custom properties — populated incrementally as each layer is designed

typography:
  stacks: {}   # open-ended map keyed by Google Fonts category (sans-serif, serif, monospace, display, handwriting)
  usage: {}    # parallel map — same keys as stacks, values explain design role
  scale:
    ratio: 0     # e.g., 1.2 (minor third)
    steps: []    # computed rem values
  weights: {}   # same keys as stacks — e.g., monospace: [400, 700], sans-serif: [400, 600, 700]
  line_height:
    chrome: { min: 0, max: 0 }  # tight — short-run labels, nav
    body: 0                      # generous — sustained reading
    code: 0                      # between chrome and body
  letter_spacing:
    uppercase_labels: ""  # e.g., "0.05em to 0.1em"
    body: ""              # e.g., "0"

colors:
  color_scheme: ""  # e.g., "dark light" — tells browser how to render UA elements
  surfaces:
    light: ""
    dark: ""
    note: ""
  content:
    dark_surface:
      primary: ""
      secondary: ""
      tertiary: ""
    light_surface:
      primary: ""
      secondary: ""
      tertiary: ""
    note: ""
  semantic_hues:
    - name: ""
      role: ""
      dark: ""
      light: ""
      reference: ""
  syntax:
    variant: "base16"
    derivation_arc: ""
    derivation_principle: ""
    dark:
      base00: "" # through base0F
    light:
      base00: "" # through base0F
    known_issue: ""

layout:
  base_unit: ""
  scale: []
  content_max_width: ""
  vertical_rhythm: ""
  breakpoints:
    tablet: ""
    desktop: ""
    ultra: ""
  mobile_strategy: ""
  chrome_density: ""
  body_density: ""

borders:
  stroke_weights: []
  corner_radius: ""
  divider_styles: []
  containment_model: ""

transitions:
  duration:
    fast: ""
    normal: ""
    slow: ""
  easing: ""
  reduced_motion: ""

components: []  # defined after all foundational layers
```
