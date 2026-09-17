---
inclusion: manual
---

# Rise Hub — Build a New UI (Code)

Entry point for building a new Hub UI feature in code (React Native, `@nutrien/rise-ui-kit`). Pull this in when you're implementing a screen, view, or component from scratch. It loads the Rise reasoning and pitfall context and lays out the build workflow.

## Context loaded with this guide

Read and apply both before writing UI:

1. How the Rise system fits together — principles and the "why" behind them — #[[file:ux-rise-rules-code.md]]
2. Common ways code drifts from the system, and how to avoid them — #[[file:ux-rise-pitfalls-code.md]]

Two more references are already in context on every turn (always-on steering), use them directly, no need to pull them in:

- `rise-ui-kit` — the binding gate: non-negotiables and which detail docs to load for a given surface.
- `rise-code-connect` — the lookup layer: resolve any component's import path, Figma node, and Storybook page via `rise/rise-ui-kit/src/components/figma-constants.ts`.

When you need per-component detail (props, variants, states) beyond the reasoning docs, `rise-ui-kit` names the detail docs to load; `rise-code-connect` resolves the exact import.

### zeroheight is the source of truth for usage guidance

Before you reach for a component, check its zeroheight page via the MCP (`mcp_zeroheight_search_pages` to find it, `mcp_zeroheight_get_page` to read it). This is where the *when to use* and *why to use* guidance lives — the intended use cases, when to pick this component over an alternative, variant and prop recommendations, and the do's and don'ts that the detail docs and code mapping don't carry. Don't infer a component's purpose from its name or props; confirm it against the documented guidance. When zeroheight and a detail doc conflict, zeroheight wins.

## Build workflow

Follow this order when implementing a new UI.

### 1. Map the UI to Rise components first
Before writing markup, identify which `@nutrien/rise-ui-kit` component covers each element (surface, inputs, actions, feedback, data display). For each candidate, check its zeroheight page via the MCP (`mcp_zeroheight_search_pages`, `mcp_zeroheight_get_page`) to confirm *when to use* and *why to use* it — that it's the right component for the job and not a near-neighbor that looks similar. Resolve the import for each via `rise-code-connect`. Only consider building something custom after confirming the kit doesn't already ship it — and if you do go custom, flag it rather than silently filling the gap.

### 2. Compose from kit components, don't rebuild
Use the kit's building blocks as-is — tables from the table components, overlays from the unified Menu/Sheet and Popover/Sheet, cards from Card. Don't reassemble these from `View`/`Text`, and don't override or mutate a component's styling to force a different appearance.

### 3. Bind everything to the theme
Import once: `import theme from '@nutrien/rise-ui-kit'`. Colors from `theme.colors.<semanticToken>` (never `prim`, never hex), spacing from `theme.spacing.space*`, radius from `theme.borderRadius.radius*` (not `theme.br`), shadows/borders spread from `theme.shadows.*` / `theme.border.*`. Typography via `<Text variant="…">`, not hand-set font properties. See `ux-rise-rules-code` for the accessor details and the non-obvious ones.

### 4. Handle every state
For each interactive element and data surface, implement: empty, loading (kit activity indicator / skeleton), error (specific message), success, and disabled (only where allowed — never a form submit). A blank space is not an empty state.

### 5. Wire text and data correctly
All user-facing strings go through `t()` (never hardcoded). Follow the repo's data conventions — data hooks wrap queries, server-side pagination, no client-side "fetch all". (These are covered by the always-on dev and data steering.)

### 6. Meet the accessibility floor
Keyboard-reachable in a logical order, a programmatic label on every field, 44×44 minimum touch targets, WCAG AA contrast, alt/aria-labels on meaningful imagery. This is pass/fail.

### 7. Verify before you call it done
Run the package's lint and tests scoped to the files you changed (see the dev standards steering for the exact scoped commands). Fix issues before presenting the result.

## Non-negotiables (from ux-rise-rules-code)

- Rise UI Kit only, never Bonsai. Import from `@nutrien/rise-ui-kit`.
- Semantic color tokens only — never primitives, never hardcoded hex.
- Two type weights (Regular / SemiBold), via `Text` `variant`; hierarchy from size, not weight.
- One primary action per view; never disable a form submit button.
- Tables on grey, white rows, no zebra striping; lazy load on mobile.
- Overlays via the unified Menu/Sheet and Popover/Sheet; drawers deprecated → Blade or Sheet.
- Accessibility is pass/fail.
