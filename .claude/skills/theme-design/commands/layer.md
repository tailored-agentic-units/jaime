# Command: layer

Apply a specific foundational layer to the blog theme.

## Usage

```
design layer <name>
```

Where `<name>` is one of: `reset`, `tokens`, `typography`, `colors`, `layout`, `borders`, `transitions`, `components`.

## Workflow

1. **Verify order** — layers must be applied in sequence. Check that all preceding layers have been completed in the spec.
2. **Discuss approach** — present the design intent, key decisions, and tradeoffs before writing any CSS. This avoids wasted iteration on a rejected direction.
3. **Write CSS** — generate the layer's CSS file based on the agreed approach.
4. **Visual validation** — inspect via Playwright (snapshot preferred, screenshot when needed). Check both dark and light modes using `prefers-color-scheme`.
5. **Iterate** — refine based on visual inspection until satisfied.
6. **Update tokens** — add any new CSS custom properties identified during this layer to `tokens.css`.
7. **Update spec** — record validated decisions in `spec.yaml`.

## Key Principles

- Each layer is validated in isolation before the next is introduced.
- No layer should reference concepts from a later layer.
- Layers inherit the foundation — only override when deviating from the baseline.
- Tokens are a living accumulator — each layer may introduce new tokens as decisions are validated.
