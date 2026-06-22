---
name: component-updates
description: Document what changed between an initial library component and a modified version of it, as a developer-facing handoff section in Figma. Use this whenever the user wants to compare two components, document component overrides or modifications, explain to developers what was customized against a library component, or produce a spec for a component. Trigger on phrases like "what changed in this component", "document the differences from the library version", "create a handoff spec for this component", or "compare these two components". Asks for the initial component (optional) and the modified one, classifies the change, and after approval creates a documentation section in Figma. If no initial component is given, it produces a standalone spec of the component instead.
---

# Component Updates Mapping

Produce a clear, developer-facing documentation section in Figma that communicates exactly what was changed on a component relative to its original library version — so a developer can implement the customization without guessing. Wherever a changed value is bound to a variable that defines code syntax, surface that syntax so the developer sees the real token, not just a raw value.

## The core classification rule

Decide which of three modes you are in. This decision drives everything in the doc.

1. **Modified component** — the anatomy is the same, and only **styling** differs: color, typography, spacing, sizing, corner radii. This is a true modification. Document the styling changes property by property.

2. **Rebuilt / different component** — the **anatomy** changed: layers added or removed, structure or hierarchy reorganized, elements replaced or re-nested. When anatomy differs, this is **no longer the same component**. Do not present it as a diff. Instead, state plainly in the doc that this is **not a modified version of the original — it is a different, custom component**, and then describe its spec. Briefly note what anatomical differences led to that conclusion.

3. **Spec only** — the user gave no initial component. There is nothing to diff against, so simply describe the spec of the component provided.

Anatomy = the set, count, nesting, and roles of layers/elements. Styling = the visual property values on those layers. If you are unsure whether a change is anatomical or stylistic, treat a change in layer structure as anatomical and a change in a property value as stylistic.

## Preflight: confirm the Figma connection before any work

This skill depends entirely on the Figma MCP server. Before anything else, confirm it is connected:

1. Probe with `get_metadata` (or `get_design_context`) against the target file key. If it returns real data, proceed.
2. If it errors or is empty, stop and guide the user: in Claude Code / Cowork, run `/mcp` to check the `figma` server and authenticate if disconnected. Confirm the file(s) are reachable by the authenticated account.
3. Do not proceed until the probe succeeds.

## Inputs to gather

When launched via the `/component-updates` command, gather these **one question at a time**, conversationally. Confirm each before asking the next:

- **Initial (library) component** — OPTIONAL. Ask for a link to the original component. Make clear they can skip it ("leave blank if you just want a spec of the component"). Accept a Figma node URL; extract both the file key and the node id.
- **Modified / target component** — REQUIRED. The component to document. Accept a Figma node URL; extract file key and node id.
- **Documentation destination** — where to create the doc section (a page or frame link). Default to creating a frame titled "Component Updates" if they have no preference.

The two components may live in the **same file or two different files** — handle both. When they are in different files, read each from its own file key; do not assume a shared key.

## Reading the components

Use **`get_design_context`** (and `get_metadata` for structure) on each node to retrieve its layer tree and properties. This is the right tool for node-level structure and styling; `get_variable_defs` remains the way to resolve any bound variables.

For each component capture:

- **Anatomy** — the layer tree: names, types, nesting, and count. This is what you compare to classify modified vs. rebuilt.
- **Styling per layer** — fills/colors, typography (family, size, weight, line height), spacing (padding, gap), sizing (width, height, constraints), corner radii.
- **Bound variables** — where a property is bound to a variable rather than a raw value, capture the variable and resolve it. If that variable defines **code syntax** (Web / iOS / Android), capture each syntax that exists — only the ones actually defined, never invented.

Colors arrive as RGBA `{ r, g, b, a }` in the 0–1 range; convert to hex for display.

## Comparing (modified mode)

When an initial component is provided and the anatomy matches:

- Walk both layer trees in parallel and collect only the **styling properties that differ**.
- For each difference record: layer/element, property (e.g. "corner radius", "fill", "heading font size"), original value, new value, and — if the new value is bound to a variable with code syntax — the variable name and its defined syntaxes.
- Group differences by category: Color, Typography, Spacing, Sizing, Corner Radius.
- Report only real differences. An unchanged property is not listed.

If the anatomy does **not** match, switch to rebuilt mode: do not produce a property diff; capture the spec of the modified component and the anatomical reasons it is considered different.

## Approval gate — required before writing anything

Present what you found and wait for the user. Show it as a **numbered list with checkbox markers**, all selected by default. The content depends on mode:

- **Modified**: list each styling change.
  ```
  Mode: Modified component (anatomy unchanged)

  Corner Radius
  1. [x] Button / container   radius 4 → 8   (var: radius/md · web: var(--radius-md))

  Color
  2. [x] Button / container   fill #003087 → #171717   (var: color/primary)
  ```
- **Rebuilt**: show the "different component" verdict, the anatomical reasons, and the spec lines.
  ```
  Mode: Different component (anatomy changed)
  Reason: 3 layers added, icon group re-nested under a new frame.

  Spec
  1. [x] container   radius 8 · fill #171717 (var: color/primary)
  ...
  ```
- **Spec only**: just the spec lines.

Then ask: **"Reply `all` to document everything, or tell me which line numbers to exclude (e.g. `skip 2`)."** Do not write anything to Figma until they reply.

## Creating the documentation in Figma

After approval, create a native auto-layout documentation section on the destination (default a frame titled "Component Updates"). Include:

- A **header** with the component name and the mode verdict — for rebuilt components, lead with a clear callout: "This is a different custom component, not a modified version of <original>." with the anatomical reasons.
- The approved changes (or spec) grouped by category, each row showing original → new where applicable.
- For any changed value bound to a variable with code syntax, show the **variable name and its code syntax** (only the platforms that exist) so developers get the real token reference.
- Keep it scannable: a developer should grasp what to implement at a glance.

When the tool is `use_figma` with a `fileKey`, call it directly with a precise description of the section. After creating, give the user a direct link to the documentation frame.

## Failure modes to handle gracefully

- **Node URL with no node id**: ask the user to right-click the component in Figma and "Copy link to selection" so the node id is included.
- **Original and modified are identical**: say so plainly — there are no changes to document.
- **Anatomy ambiguous**: default to calling a structural change anatomical (rebuilt), and state the reasoning so the user can correct you.
- **Cross-file variable references**: resolve the bound variable from whichever file defines it; if unreachable, show the value and note the variable could not be resolved.

## What's next

After the flow completes and you've confirmed the result, show a brief "What's next" list so the user discovers the plugin's other tools. List the *other* commands (omit the one just used), one line each:

- **`/modes-map`** — side-by-side mapping table between two variable modes.
- **`/brand-upload`** — create a "Brand Foundations" variable collection from brand guidelines.
- **`/component-updates`** — document component changes for developer handoff.
