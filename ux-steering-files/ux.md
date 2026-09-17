---
inclusion: always
---

# UX Standards

**Owner:** Sara Pacheco — propose additions or changes via Slack or [sara.pacheco@nutrien.com](mailto:sara.pacheco@nutrien.com)

## Design Thinking

When making experience decisions, reviewing ideas, or generating UX recommendations, pull in the design thinking guide:

- #[[file:ux-design-thinking.md]] — user personas, agriculture context, expected AI behavior, usability heuristics, and operational reality checks.

## Rise Design System

When implementing any new UI, pull in the Rise build guide:

- #[[file:ux-rise-build-guide.md]] — entry point for building a new Hub UI feature; loads the system rules and pitfalls context and lays out the build workflow.
- Use Rise UI Kit (`@nutrien/rise-ui-kit`) for new components with "Current Documentation" status.
- If no Rise component exists, do not use a Bonsai (`@nutrien/bonsai-core`) component.
- When you can't find what you need, build bespoke based on Rise patterns and tokens, and flag in `#designsystem_chat` before promoting to shared use.

### Design System Principles

When building a new component or view, follow these principles in priority order:

1. Serve current needs — solve the problem at hand, don't over-engineer for hypothetical futures.
2. Flexibility over control — prefer unopinionated components; never overwrite styles or mutate Rise components.
3. Usability and stability over speed — no shortcuts; prioritize reliability and completeness.
4. Clarity over brevity — explain things fully in code comments and labels; don't assume prior knowledge.
5. Support innovation — bespoke components are fine for exploration, but flag in `#designsystem_chat` before promoting to shared use.

## Zeroheight MCP

When implementing UI, use the Zeroheight MCP to fetch the latest Rise documentation before writing code. The design system is in Alpha — specs evolve frequently.

- Use `list-pages` to find the relevant component page.
- Use `get-page` to read usage guidelines, anatomy, specs, and states.
- Check the "Status" of a component page — prefer "Current Documentation" pages; treat "Not Yet Developed" components as unavailable in Rise.

## Using Figma as Visual Source of Truth

When a Figma URL or node reference is provided, always use the Figma MCP to extract design context before writing UI code. The Figma file is the definitive source for layout, spacing, color, and visual hierarchy.

- Use `get_design_context` to pull reference code, screenshots, and metadata for a given node — this is the primary tool for design-to-code work.
- Use `get_screenshot` to visually verify a node when you need to confirm layout or appearance.
- Use `get_metadata` only for structural overview (layer names, positions, sizes) — prefer `get_design_context` for implementation work.
- If the Figma design conflicts with Zeroheight documentation, Figma wins for visual specifics (spacing, layout, color); Zeroheight wins for component API usage and interaction behavior.
- When implementing a screen from a design, ask for the Figma URL if one hasn't been provided.
