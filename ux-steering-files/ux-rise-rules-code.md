---
inclusion: manual
---

# Rise Hub — System Rules (Code)

How the Rise UI Kit fits together when you build Hub UI in code (React Native, `@nutrien/rise-ui-kit`). This is the reasoning layer: not "which import" (see `rise-code-connect`) but why the system is shaped the way it is, so implementation choices stay coherent with it. The rules below are the ones a component author has to honor; each says what to do in code and why it exists.

Token notation here is the code form off the theme, not the Figma form. Import once: `import theme from '@nutrien/rise-ui-kit'`. Figma writes `rise/hub/color/sem/bg-hover-error`; code writes `theme.colors.bgHoverError`. The mapping is not always mechanical — see the token rules below.

## 1. Two type weights, hierarchy from size

Hub uses two font weights — Regular (400) and SemiBold (600) — for effectively all UI text. Bold/ExtraBold exist only for the Antonio display faces. You almost never set weight by hand: render text through the kit's `Text` component and pick a `variant`, which carries the right family, size, and line height together.

```tsx
import { Text } from '@nutrien/rise-ui-kit';

<Text variant="headerSm1" color="textDefault">Field summary</Text>
<Text variant="bodyDefault" color="textSecondary">12 fields</Text>
```

- Do: create hierarchy by choosing a larger/smaller `variant` (`displayXl1` → `headerLg1` → `headerMd1` → `bodyDefault` → `captionDefault`), not by bumping weight.
- Don't: hand-roll `fontWeight`/`fontFamily` in a `StyleSheet`, or introduce Medium (500). It's not in the system.
- Why: in a data-dense app, a two-weight system keeps dense tables and forms scannable. A third weight muddies the hierarchy. Binding to `variant` also means text reflows correctly if the type scale is revised.

Families are already baked into the variants: Open Sans for body/headers, Antonio for `display*`, Roboto Condensed for the `*Condensed` variants (chart labels, tight table text).

## 2. Semantic color tokens only

Color comes off `theme.colors` using the camelCase semantic name. Never reach into `theme.colors.prim.*` (the primitive palette) for UI, and never inline a hex string.

```ts
import { StyleSheet } from 'react-native';
import theme from '@nutrien/rise-ui-kit';

const styles = StyleSheet.create({
  card: { backgroundColor: theme.colors.white },
  errorText: { color: theme.colors.textError },
  divider: { backgroundColor: theme.colors.outlineDefault },
});
```

For text and icons, prefer the component's own color prop (`<Text color="textError">`, Icon color props) over restyling.

- Do: use semantic tokens — `bgErrorTertiary`, `textHoverError`, `outlineStatusSuccess`, `bgSelected`.
- Don't: use `theme.colors.prim.corn` (or any primitive) for UI, and don't hardcode `#cc280a`.
- Why: semantic tokens can be remapped per product line (Hub vs. Agrible) or mode; primitives and hex literals can't. Binding to the semantic name is what makes a color "correct" rather than "happens to match today."

The semantic names don't follow one clean pattern (e.g. `bgError` has no `default` suffix but `bgWarningDefault` does; error text appears as `textError`, `textHoverError`, and `textStatusError` for three different jobs). Don't infer a name — confirm it against `theme.colors` before using it.

## 3. Spacing, radius, shadows, borders — off the theme, in code form

These are the accessors that differ most from Figma. Get them right or the values look right but aren't bound.

- Spacing: `theme.spacing.space8`, `theme.spacing.space16`, … The key is the pixel value.
- Radius: `theme.borderRadius.radius400` (4px). Note it's `theme.borderRadius`, NOT `theme.br` — the Figma path is `br`, the code accessor is not.
- Shadows: reserved for overlays only (see below). When you do use one, `theme.shadows.shadow200` is a full RN shadow style object — spread it, don't assign it. `{ ...theme.shadows.shadow200 }`.
- Borders: `theme.border.border100S` is likewise a spread object (`{ ...theme.border.border100S }`).

```ts
const styles = StyleSheet.create({
  row: { paddingVertical: theme.spacing.space8, paddingHorizontal: theme.spacing.space16 },
  card: { borderRadius: theme.borderRadius.radius400, backgroundColor: theme.colors.white },
  input: { ...theme.border.border100S, borderColor: theme.colors.outlineDefault },
});
```

### 8px is the default gap; 16px is for forms

The default spacing between elements is **8px** (`space8`). Use it for the gap between stacked or tiled elements — including the gap **between dashboard cards** (cards sit 8px apart, always). Reach for a larger value only with a specific reason.

The main exception is **forms**: fields within a form are spaced **16px** (`space16`) apart. So 16px reads as "this is form field rhythm"; 8px reads as "default layout spacing." Don't default to 16px for card grids, list gaps, or general layout — that's the form spacing leaking where it doesn't belong.

- Do: `gap: theme.spacing.space8` between dashboard cards and for general element spacing; `space16` between form fields.
- Don't: put 16px between cards or tiles; don't put 8px between form fields.
- (Page-level outer padding and a card's own interior padding are separate from this element-gap rule and can be larger.)

Radius scale: `radius200` (2), `radius400` (4, the default), `radius650` (6.5), `radius800` (8), `radius1000` (10), `radiusCircle` (50), `radiusPill` (100).

- Do: default to `radius400` for cards, inputs, buttons, containers; use `radiusCircle`/`radiusPill` for chips and pills.
- Don't: use 8px because other systems default to it; don't mix radii on one surface; don't invent shadow values.
- Why: a single default radius keeps a dense UI tight and professional.

### Shadows are for overlays only

Shadows are reserved for **popovers and modals** — the surfaces that genuinely float above the page. Nothing else gets a shadow: not cards, not inputs, not tables, not the Blade, not "main components."

- Do: apply a shadow only to a popover or a modal/dialog surface, by spreading a `theme.shadows.*` token.
- Don't: put a shadow on a card. Never reach for `shadow*` to make a card "pop."
- Don't: invent custom shadow values or use a shadow to signal interaction state — use color/border tokens for that.
- Why: on a data-dense screen, elevation is meaningful only if it's rare. Reserving shadows for true overlays keeps "floats above everything" a reliable, unambiguous signal; shadowing in-page surfaces makes the z-axis noisy and the interface heavier than it should be.

### A card is just a white surface — no border, no shadow at rest

A resting card is a plain white (`theme.colors.white`) surface with `radius400` corners and nothing else — no border, no shadow. Its separation from the page comes entirely from the white-on-grey contrast (white card on the grey `bgDefault`/`bgTertiary` page). Don't add a resting `borderWidth`/`borderColor` to a card to "define" it.

The only time a card shows a border is the **hover and focus** state of a *clickable* card (a card acting as a button): hover binds `outlineDefault`, selected binds `primary`. That border is an interaction affordance, not a resting style — a non-interactive content card never gets one.

```ts
const styles = StyleSheet.create({
  card: {
    backgroundColor: theme.colors.white,
    borderRadius: theme.borderRadius.radius400,
    padding: theme.spacing.space16,
  }, // no borderWidth, no shadow
});
```

## 4. Data-table conventions

Tables are a system, not a layout you assemble. Use the kit's table building blocks (see `rise-code-connect` / the data-display reference for the exact component names and props) rather than composing rows from `View`s and `Text`.

- Cell padding is 8 vertical / 16 horizontal (`space8` / `space16`); compact/nested rows use 4/8 (`space4` / `space8`). This comes from the table components — don't re-pad cells by hand.
- Tables render on a grey surface with white rows and no zebra striping. The grey page background (`bgTertiary` / `bgDefault` family) behind white rows gives the contrast; alternating row colors add noise.
- Pagination is numbered on desktop only; tablet and mobile use lazy load / infinite scroll.
- Alignment: left for text and non-quantified numbers (IDs, phone), right for quantified numbers (currency, decimals), center for icons; headers match their column.
- Why: the app is fundamentally data-heavy. The 8/16 rhythm keeps vertical density high while staying touchable, and the grey/white/no-stripe treatment maximizes scannability.

## 5. Overlays are unified, responsive components

The kit doesn't ship separate "Dropdown", "Bottom Sheet", and "Select" widgets. Selection menus and action lists come from one Menu/Sheet component that renders as a menu at ≥ medium and a sheet below medium; contextual/quick-task overlays come from one Popover/Sheet component the same way. (Names and imports: `rise-code-connect`.)

- Do: use the unified overlay component and let it switch by breakpoint.
- Don't: build a custom overlay, or hunt for a standalone dropdown/select. A hand-built overlay misses the elevation levels, transitions, and dismissal behavior.
- Why: one component guarantees a desktop menu becomes a mobile sheet automatically, with consistent scrim, elevation, and dismissal.

## 6. Blade is a rail that shares the viewport

The Blade is a persistent, collapsible side rail (a narrow icon rail that expands to a panel). It sits beside content and shares the viewport — it does not overlay content the way a drawer would. On small breakpoints it becomes a full-screen panel with a proper header and larger touch targets, not a shrunk-down rail.

- Do: place the Blade as a sibling of the content region so content resizes when it expands/collapses; drive which tools appear via the component's props rather than manually hiding children.
- Don't: implement it as a slide-over that floats above a map/table; don't hand-space it (its outer spacing is owned by the layout wrapper, not the Blade).
- Why: if the Blade overlays content, a map or table underneath can't resize and the user loses spatial context. The rail keeps tools reachable without dismissing anything. Small icons are unusable in field conditions (gloves, sunlight), which is why the mobile form is a full panel.

## 7. Drawers are deprecated — Blade or Sheet

Don't reach for drawer components in new work. Use a Blade for a focused task that needs full attention, a Sheet for a compact selection on native. Drawers remain only for backward compatibility.

- Why: Blade takeovers give full focus for complex flows; sheets resolve quickly for simple choices. Drawers sit awkwardly between the two.

## 8. DAN is a separate theme, not a mode switch

DAN (the AI surface) has its own tokens (`danPrimary`, `danBgDefault`, …) and its own forked components (`DAN Button`, `DAN Table`, etc.). These are parallel to the base tokens/components, not mode-switched versions of them — base changes do not flow into the DAN forks automatically.

- Do: use the `dan*` tokens explicitly when building a DAN feature; check whether a base component styled with DAN tokens is enough before defaulting to a DAN-prefixed fork.
- Don't: expect base tokens to "become" DAN in a DAN context; don't swap DAN's branded orb affordance for a generic AI glyph.

## 9. Data-visualization color discipline

Chart colors are categorical assignments, not aesthetic picks. The data-viz ramps live under `theme.colors` too (e.g. `dataVizBlue700`, `dataVizGreen800`, and the categorical set).

- Single-series charts: use the default categorical color rather than an arbitrary one.
- Categorical palettes: cap at 6 colors; keep the same category → color mapping across every chart in a view.
- Sequential ramps: darker = larger values. Don't use a categorical palette for ordered/interval/ratio data.
- Why: consistent category color across charts is what lets a user read several charts as one story; picking "whichever green looks good" breaks that.

## 10. Working in a package that still uses Bonsai

Rise is the target; Bonsai is the legacy system being sunset. But much of the app — including whole micro-frontends like the dashboard — is still built on `@nutrien/bonsai-core` today. You will routinely open a file whose neighbors import `Card`, `List`, `EmptyState`, `Divider`, etc. from Bonsai. This section is about what to do then, because "Rise only" alone doesn't tell you how to behave inside a Bonsai-still package.

The rule: **new UI is built in Rise, even when its neighbors are Bonsai.** Don't add new Bonsai usage to match the surroundings, and don't mix Bonsai and Rise inside a single new component. Following this will make your new component diverge from the file next to it — that divergence is expected and correct, it's the migration happening one component at a time.

- Do: build the new component entirely in Rise (`@nutrien/rise-ui-kit`), even if `OtherCard.tsx` beside it is Bonsai.
- Do: leave a brief note (code comment) when your new Rise component sits among Bonsai siblings, so the divergence reads as intentional, not accidental.
- Don't: convert the neighboring Bonsai components as a side effect of your task — a Bonsai→Rise migration is its own scoped change (imports, props, tests all shift), not drive-by work. If a screen genuinely needs the old and new to interoperate, that's a `#designsystem_chat` question, not a silent call.
- Don't: reach for a Bonsai component because "that's what this package uses." The package's current state is not the standard; the steering is.
- Note the API gap: Rise and Bonsai components with the same name are not the same. Bonsai's `Card` has slots (`Card.Content`, `Card.Header`); Rise's `Card` is a minimal padded surface you compose yourself. Don't assume a Bonsai prop/slot exists on the Rise equivalent — verify against the Rise source or `rise-code-connect`.
- Why: if new work keeps landing in Bonsai to stay locally consistent, the sunset never finishes. Local inconsistency during migration is the acceptable cost of moving forward; the alternative is never moving.

## Non-negotiables (quick reference)

These hold regardless of context:

- One primary action per view; secondary buttons support; text-only and icon buttons are tertiary. Icon-only buttons need an accessible label/tooltip.
- Never disable a form submit button — keep submission enabled and surface validation errors after the attempt. Disabled controls are allowed only outside forms where every element is optional.
- Card footers: at most 3 actions (consolidate beyond that into a "More" menu); at most 2 footer icon buttons.
- Tables on grey, white rows, no zebra striping. Numbered pagination desktop-only; lazy load on tablet/mobile.
- Semantic tokens only (`theme.colors.*`, never `prim`, never hex). Two type weights via `Text` `variant`.
- Default radius `radius400`. Shadows on overlays only (popovers, modals) — never on cards or other in-page surfaces.
- Default element gap is `space8` (8px), including between dashboard cards; `space16` is for form-field spacing only.
- Overlays via the unified Menu/Sheet and Popover/Sheet components; no custom overlays.
- Blade shares the viewport; drawers are deprecated → Blade or Sheet.
- All user-facing text is translated (`t()`), never hardcoded.
- Accessibility is pass/fail: keyboard-reachable in logical order, a programmatic label on every field, 44×44 minimum touch targets, WCAG AA contrast, alt/aria-labels on meaningful imagery.
