# Command: enhance

Refine an existing layer or the full composition.

## Usage

```
design enhance [layer|all]
```

- `design enhance colors` — refine just the colors layer
- `design enhance all` — refine the full composition holistically

## Workflow

1. **Scope the session** — clarify what is being enhanced and what the goals are.
2. **Review current state** — read the relevant CSS files and spec sections.
3. **Make targeted changes** — apply refinements to the scoped layer(s).
4. **Visual validation** — inspect via Playwright. Check both dark and light modes.
5. **Iterate** — refine based on visual inspection until satisfied.
6. **Update spec** — record any changed decisions in `spec.yaml`.

## Guidelines

- Enhancement sessions should be explicitly scoped — avoid scope creep into unrelated layers.
- When enhancing a lower layer, verify that higher layers still look correct after the change.
- Component overrides are valid when the semantic context requires deviation from foundational layers.
