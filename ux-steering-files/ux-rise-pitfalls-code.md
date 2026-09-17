---
inclusion: manual
---

# Rise Hub — Pitfalls (Code)

Common ways Hub UI code drifts from the Rise system, and how to avoid them. Each is a mistake that compiles and often looks right, but breaks theming, accessibility, density, or consistency. Pairs with `ux-rise-rules-code` (the reasoning) and `rise-code-connect` (the lookup). Token notation is the code form off `import theme from '@nutrien/rise-ui-kit'`.

## Tokens and color

### Don't hardcode color; don't use primitives
Bind to semantic tokens on `theme.colors` (e.g. `theme.colors.bgErrorTertiary`, `theme.colors.textHoverError`, `theme.colors.outlineStatusSuccess`). Never inline a hex string, and never reach into `theme.colors.prim.*` for UI color — primitives are the palette, not usage instructions. A hex literal that matches today won't remap when the theme changes per product line or mode.

### Don't infer a semantic token name — confirm it
The semantic names aren't a clean pattern. `bgError` has no `default` suffix but `bgWarningDefault` does; error text exists as `textError`, `textHoverError`, and `textStatusError` for three different jobs. Autocompleting a plausible name gets you a wrong or undefined token. Confirm against `theme.colors` before using.

### Radius is `theme.borderRadius`, not `theme.br`
The Figma variable path is `br`, but the code accessor is `theme.borderRadius.radius400`. Writing `theme.br.radius400` won't resolve. Same trap in reverse for anyone porting from the token names.

### Shadows and borders are objects — spread them
`theme.shadows.*` and `theme.border.border100S` are full RN style objects, not scalars. Assigning them to a single property is wrong; spread them:

```ts
const styles = StyleSheet.create({
  popover: { ...theme.shadows.shadow200, borderRadius: theme.borderRadius.radius400 },
  input: { ...theme.border.border100S, borderColor: theme.colors.outlineDefault },
});
```

### A card is just white — no shadow AND no resting border
A resting card is a plain white surface with `radius400` corners: no shadow, and **no border**. Separation comes entirely from the white-on-grey contrast (white card on the grey `bgDefault`/`bgTertiary` page). Don't add `borderWidth`/`borderColor` to define a card, and don't reach for `theme.shadows.*`. A border appears only on the **hover/focus** state of a *clickable* card (card-as-button) — hover uses `outlineDefault`, selected uses `primary` — never on a resting or non-interactive content card. Shadows themselves are reserved for overlays only (popovers, modals); inputs, tables, and the Blade get no shadow either. Don't invent custom shadow values, and don't use elevation to signal interaction state.

### DAN tokens don't auto-switch
`dan*` tokens (`danPrimary`, `danBgDefault`, …) are a separate set, not a mode applied over the base tokens. In a DAN surface you must use the `dan*` tokens explicitly — the base tokens won't "become" DAN on their own.

## Component selection

### Pick the right small-indicator component
The chip/tag/status family overlaps. Read-only status → Status Chip. Removable/selectable filter → Filter Chip. Metadata/category label → Tag. Interactive trigger → Action Chip. Reaching for the wrong one produces something that looks similar but behaves and reads wrong. Confirm the component and its import via `rise-code-connect`.

### Don't build tables by hand
Don't assemble rows from `View` + `Text`. Use the kit's table building blocks so cell padding (8/16), alignment rules, header/footer behavior, and responsive format-switching come for free. A hand-built table misses all of that and drifts on every screen.

### Don't reach for drawers in new work
Drawers are deprecated. Use a Blade for a focused task, a Sheet for a compact native selection. Drawer components remain only for backward compatibility — if you're about to use one, a Blade or Sheet is almost certainly the right call.

### Don't default to a DAN fork
DAN-prefixed components are forks of the base components, not extensions — base fixes don't propagate to them. Before using a DAN fork, check whether the base component styled with `dan*` tokens covers the need.

## Layout, spacing, density

### Don't re-pad table cells
Cell rhythm is 8 vertical / 16 horizontal (`space8` / `space16`), owned by the table components; compact/nested rows use 4/8. Adding your own uniform 16px padding breaks alignment with adjacent rows.

### Don't use 16px as the default gap — 8px is the default
The default gap between elements is `space8` (8px), including the gap **between dashboard cards** (always 8px apart). 16px (`space16`) is specifically **form-field** spacing — using it for card grids, tile gaps, or general layout is the form rhythm leaking where it doesn't belong. Default to `gap: space8`; reach for 16px only between form fields (or with a deliberate reason). Page-level outer padding and a card's own interior padding are a separate concern and can be larger.

### Don't use 8px radius by reflex
The default is `radius400` (4px). Other design systems default to 8px; Hub doesn't. Don't mix radii on a single surface either.

### Don't hand-space the Blade
The Blade's outer spacing comes from its layout wrapper, not from padding/margins on the Blade itself. Setting padding on the Blade or manual margins between it and content fights the layout. Compose it as a sibling of the content region and let the wrapper handle the gaps.

### Keep action buttons together, right-aligned
Action buttons live together in a footer, right-aligned — not split to opposite screen edges (Cancel far left, Save far right). Max 2 side by side; beyond that, consolidate into a menu. On small breakpoints they go full-width within the content region, never edge-to-edge across rails.

### Don't zebra-stripe tables
Rows are solid white on a grey surface. The grey behind white rows is the contrast; alternating row backgrounds add noise in a dense UI.

## Typography

### Don't hand-set weight or family
Render text through `Text` with a `variant` (`headerSm1`, `bodyDefault`, `captionSemibold`, …). Don't set `fontWeight`/`fontFamily` in a StyleSheet. There are two weights (Regular 400, SemiBold 600); Medium (500) isn't in the system, and Bold/ExtraBold are Antonio display only. Hand-set text won't reflow if the scale is revised.

### Create hierarchy with size, not weight
Reach for a larger/smaller `variant`, not a heavier weight, to signal importance.

## Interaction and behavior

### Never disable a form submit button
Keep submission enabled and surface validation errors after the attempt. Disabled controls are allowed only outside forms where every element is optional. A disabled submit leaves the user stuck with no feedback about why.

### Don't overload a card footer
Max 3 actions in a card footer; beyond that use a "More" menu. Footer icon buttons cap at 2 (typically edit/delete).

### Tables must be on a grey background
Never place a table on white. This is a hard rule, not a preference.

### Text-only button placement restrictions
Text-only primary (ghost) buttons don't go on grey backgrounds. Text-only secondary buttons are restricted to inside Banner components.

### "Apply" vs "Done" aren't interchangeable
"Apply" for selection tools that change page content (filters, sorting). "Done" for viewing/reviewing without changing anything. All user-facing labels go through `t()` — never hardcode the string.

### Mobile pagination is lazy load
Numbered pagination is desktop-only. Tablet and mobile use lazy load / infinite scroll.

### Order state checks error-before-loading
When more than one state flag can be true at once, check `isError` before `isLoading`. A failed fetch that also happens to be "loading" should surface the error + retry, not an indefinite spinner. Putting the loading branch first is an easy mistake that unit tests pass over unless you assert the precedence explicitly.

## Charts and other layout-measured components need a browser check

Charts render through Victory (`Charting.LineChart`, `BarChart`, `DonutChart`) and are layout-dependent — they measure their container and compute positions at render time. Unit tests run in jsdom/jest where that layout never happens, so a chart can pass typecheck, pass every unit test, and pass lint, then throw at runtime in a real browser. Green tests are not sufficient evidence a chart works.

- Do: load any screen with a chart in a browser (or on device) before calling it done. This is the one component class where the steering verification step (lint + tests) genuinely isn't enough.
- Watch the tooltip/voronoi path specifically: Victory's hover-label placement can throw on web for larger series even when the tooltip config is shaped exactly like the working examples. If a chart crashes in `VictorySharedEvents` / `getLabelPlacement`, drop the `tooltip` prop — the trend line still renders — and revisit the hover separately.
- Keep the chart in a container with an explicit height; the component renders an empty box until it has measured a non-zero width and height.

## Before building custom

If a pattern seems missing, confirm it really is before hand-rolling it — the kit's coverage is broad and components sometimes live under a name you didn't expect. Check `rise-code-connect` / the component reference first. Building a custom version of something the kit already ships creates a competing implementation that misses the system's states, tokens, and accessibility handling.
