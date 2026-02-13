# Syntax Theme Export Mappings

Each Base16 slot translates mechanically to token selectors per format.

## Rouge (CSS)

Rouge syntax classes mapped to Base16 slots:

| Base16 Slot | Rouge Classes | Token Type |
|-------------|--------------|------------|
| base00 | `.highlight` background | Background |
| base03 | `.c, .ch, .cd, .cm, .cp, .cpf, .c1, .cs` | Comments |
| base04 | `.gl, .gt` | Secondary foreground |
| base05 | `.highlight, .w, .ni, .si` | Default foreground |
| base08 | `.nv, .vc, .vg, .vi, .vm` | Variables |
| base08 | `.gd` | Diff deleted |
| base08 | `.err` (with base01 bg) | Errors |
| base09 | `.l, .ld, .m, .mb, .mf, .mh, .mi, .il, .mo, .mx` | Numbers/constants |
| base09 | `.nb, .nc, .no, .nn` | Builtins, classes, constants, namespaces |
| base0A | `.kt` (in some schemes) | Types |
| base0B | `.s, .sa, .sc, .dl, .sd, .s2, .se, .sh, .sx, .s1, .ss` | Strings |
| base0B | `.gi` | Diff inserted |
| base0B | `.sr, .na, .nt` | Regex, attributes, tags |
| base0C | `.sb, .bp` | String backtick, builtin pseudo |
| base0D | `.nf, .fm` | Functions |
| base0D | `.gh, .gu` | Headings |
| base0E | `.k, .kd, .kn, .kp, .kr, .kt, .kv` | Keywords |
| base0E | `.nd` | Decorators |
| base0E | `.o, .ow` | Operators |
| base0F | `.ne, .nl, .py` | Exceptions, labels, properties |

**Note:** The exact mapping varies by theme. Some themes assign base0C to regex/escape and base0B to strings. The table above reflects the most common convention. Adjust per-theme as needed.

## Neovim (Lua)

```lua
-- Example: applying Base16 to Neovim highlight groups
local function apply(colors)
  local hl = vim.api.nvim_set_hl
  hl(0, "Normal",        { fg = colors.base05, bg = colors.base00 })
  hl(0, "Comment",       { fg = colors.base03, italic = true })
  hl(0, "String",        { fg = colors.base0B })
  hl(0, "Number",        { fg = colors.base09 })
  hl(0, "Function",      { fg = colors.base0D })
  hl(0, "Keyword",       { fg = colors.base0E })
  hl(0, "@variable",     { fg = colors.base08 })
  hl(0, "@type",         { fg = colors.base0A })
  hl(0, "@constant",     { fg = colors.base09 })
  hl(0, "@string.regex", { fg = colors.base0C })
end
```

## VS Code (JSON tokenColors)

```json
{
  "tokenColors": [
    { "scope": "comment", "settings": { "foreground": "base03" } },
    { "scope": "string", "settings": { "foreground": "base0B" } },
    { "scope": "constant.numeric", "settings": { "foreground": "base09" } },
    { "scope": "entity.name.function", "settings": { "foreground": "base0D" } },
    { "scope": "keyword", "settings": { "foreground": "base0E" } },
    { "scope": "variable", "settings": { "foreground": "base08" } },
    { "scope": "entity.name.type", "settings": { "foreground": "base0A" } },
    { "scope": "string.regexp", "settings": { "foreground": "base0C" } }
  ]
}
```
