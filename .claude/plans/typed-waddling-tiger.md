# Graphic Realism — Session Continuation Plan

## Status: Typography layer complete. Colors layer next.

---

## Completed Layers

### 1. Reset (validated)
- Clean baseline: border-box, zero margin/padding, 100dvh body, 1.6 line-height
- `*:not(dialog)` for margin (preserves native dialog centering)
- Smooth scroll with reduced-motion respect
- Spec section populated in `spec.yaml`

### 2. Tokens (accumulated through typography)
Current tokens in `assets/css/tokens.css`:
- Font stacks: `--font-sans` (system-ui stack), `--font-mono` (ui-monospace stack)
- Type scale (minor third 1.2): `--text-xs` through `--text-3xl`
- Line heights: `--leading-tight` (1.1), `--leading-snug` (1.25), `--leading-normal` (1.45), `--leading-relaxed` (1.6)
- Font weights: `--weight-normal` (400), `--weight-bold` (700)
- Code size: `--code-size` (0.9rem)
- Letter spacing: `--tracking-tight` (0), `--tracking-wide` (0.05rem)
- **Unit convention**: rem for all sizing tokens — no em to avoid cascade compounding

### 3. Typography (validated)
Voice assignments after iterative evaluation:
- **Sans-serif** = prose voice (body text only — the reading voice)
- **Monospace** = structural voice (headings, chrome, code, links — the system voice)

Specifics:
- Headings (h1-h4): mono, minor third scale (3xl/2xl/xl/lg), bold, tight/snug line-height
- Chrome (nav, footer): mono, text-sm, uppercase, tracking-wide
- Time: mono, text-sm
- Links (`a`): mono — interactive elements use structural voice
- Code (inline + block): mono, `--code-size` (0.9rem), leading-normal
- Body inherits line-height 1.6 from reset — typography doesn't reassert it

**Inheritance principle**: layers inherit the foundation, only override when deviating. If reset establishes a baseline value, higher layers don't reassert it with a token just because a token exists.

---

## Next: Colors Layer

The bootstrap spec calls for:
- Surface colors: dark and light modes via `prefers-color-scheme`
- Three semantic hues: green (chrome), blue (interactive), pink (emphasis)
- Base16 syntax palette: 16-slot color system for code highlighting
- Content foreground: text colors at different emphasis levels

Open questions to discuss before implementing:
1. Dark-first or light-first? (Marathon leans dark)
2. Color temperature — cool/neutral grays vs warm
3. Confirm the three semantic hues (green/blue/pink)
4. Base16 approach — adapt existing scheme or build from scratch

### Remaining layers after colors:
5. **Layout** — base unit, spacing scale, content width, vertical rhythm, breakpoints
6. **Borders** — stroke weights, corner treatment, dividers, containment
7. **Transitions** — motion timing, easing, reduced-motion
8. **Components** — nav, post cards, tags, code blocks, pagination

---

## Infrastructure Done

- **Branch**: `theming`
- **CSS structure**: 8 cascade layers, one file each, declared in `assets/css/index.css`
- **Skill**: `.claude/skills/theme-design/` with SKILL.md, references, and spec
- **Spec**: `.claude/skills/theme-design/themes/graphic-realism/spec.yaml` — reset and typography sections populated
- **Stress test**: `_posts/2026-02-10-design-stress-test.md` (`published: false`) — exercises all content primitives
- **Serve command**: `bundle exec jekyll serve --drafts --unpublished --port 4000`

## Design Principles
- Bottom-up validation: each layer validated in isolation before the next
- Discuss approach first: present design intent before writing CSS
- Inheritance principle: layers inherit foundation, only override when deviating
- Spec as source of truth: populated only after visual evaluation and alignment
- rem over em for all sizing tokens
- No proactive screenshots — use `browser_snapshot` for routine checks
