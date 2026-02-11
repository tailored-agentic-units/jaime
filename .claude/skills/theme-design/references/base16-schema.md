# Base16 Slot Schema

Base16 defines 16 color slots: 8 neutrals (base00-07) forming the background/foreground gradient, and 8 accents (base08-0F) with fixed semantic assignments.

## Neutral Slots

| Slot | Role |
|------|------|
| base00 | Default background |
| base01 | Raised surface (lighter bg) |
| base02 | Selection background |
| base03 | Comments, invisibles, line highlighting |
| base04 | Dark foreground (status bars) |
| base05 | Default foreground, caret, delimiters |
| base06 | Light foreground (rarely used) |
| base07 | Brightest foreground (rarely used) |

## Accent Slots

| Slot | Semantic Role | Cognitive Group |
|------|--------------|-----------------|
| base08 | Variables, diff deleted, errors | Data lane |
| base09 | Constants, numbers, booleans | Data lane |
| base0A | Types, classes, search highlight | Identity lane |
| base0B | Strings, diff inserted, success | Data lane |
| base0C | Regex, escape sequences, support | Structure lane |
| base0D | Functions, methods, headings | Identity lane |
| base0E | Keywords, storage, selectors | Structure lane |
| base0F | Embedded language tags, deprecated | Low-priority |

## Derivation Rules

Given a theme's core hues (typically 2-4), expand into 8 accent slots by:

1. Treat core hues as anchor points on the spectral arc
2. Interpolate intermediate slots between anchors via hue rotation
3. Maintain cognitive grouping: "data" tokens share one temperature, "structure" tokens another, "identity" tokens the middle
4. Contrast hierarchy maps to cognitive priority: keywords/strings most saturated, punctuation/operators least
5. Hue count stays between 6-8 distinct hues; differentiation beyond that uses lightness/saturation shifts
6. Dark variant: accents lighter and more saturated for contrast on dark background
7. Light variant: accents darker, same hue; lightness shifted to maintain >= 4.5:1 contrast ratio
