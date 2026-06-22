---
name: brand-token-update
description: Create a new Figma variable collection holding a brand's foundation elements (colors, typography, spacing, and other values from brand guidelines). Use this whenever the user wants to set up brand foundations in Figma, turn a brand document into Figma variables, create a "Brand Foundations" collection, or capture brand colors/fonts/spacing as a new set of variables. Trigger even when the user does not say "token" or "variable" explicitly — phrases like "add our brand to Figma", "create brand foundations from this brand doc", or "make variables for our brand colors" should all use this skill. Reads the brand material, proposes the brand elements as a checklist, and after approval creates a NEW collection — it never modifies existing variables.
---

# Brand Foundations Collection

Turn a brand document into a **new** Figma variable collection that holds the brand's foundation elements. The collection is named **"Brand Foundations"** by default; the user may choose a different name.

The single most important rule: **existing variables are out of scope.** This skill only ever *creates* a new collection. It must never read-then-overwrite, rename, or otherwise modify any variable that already exists in the file. If the brand material overlaps with existing tokens, that is fine — the brand values live in the new collection, side by side with whatever was already there.

The second rule is **never invent.** Only create a variable when the brand material gives an explicit value for it. A four-element brand doc produces four variables, not forty. No placeholder or default values to "fill out" the collection.

## Preflight: confirm the Figma connection before any work

This skill depends entirely on the Figma MCP server. Before doing anything else, confirm it is connected and working, because failing early with a clear message is far better than producing hallucinated data.

1. Tell the user you need the Figma connection and ask them to confirm the file is open/shared with the connector.
2. Make one cheap probe call — `get_metadata` (or `get_variable_defs`) against the target file key. If it returns real data, the connection works; proceed.
3. If it errors or returns nothing, stop and guide them:
   - In Claude Code / Cowork, the Figma MCP must be authenticated. Point them to run `/mcp` to check the `figma` server status and authenticate if it shows disconnected.
   - Remind them the file must be accessible to the Figma account they authenticated with.
4. Do not proceed until the probe succeeds. Note: the probe only confirms the connection — you are not reading existing variables in order to change them.

## Inputs to gather

When launched via the `/brand-upload` command, gather these **one question at a time**, conversationally — the user should never need to know how to prompt. Confirm each answer before asking the next:

- **Brand material** — any of: a PDF the user attaches, pasted brand-guideline text, or a short list of values (hex codes, font families, spacing). Read what they give; do not fetch from the web unless they explicitly ask and provide a URL. Ask for this first.
- **Figma file** — where the new collection will be created. Accept a full Figma URL and extract the file key (the segment after `/design/` or `/file/`, e.g. `https://figma.com/design/ABC123/Name` → `ABC123`).
- **Collection name** — confirm the default "Brand Foundations", or let them set a different name.

## Extracting brand elements from the material

Read the brand material and pull out every **explicit** foundation value. Group them by category as you go. Cover whatever the brand material actually defines — typically:

- **Colors** — each named brand color and its hex (e.g. Primary `#003087`). Convert to figure out the RGBA later.
- **Typography** — font families, and any explicitly given weights or sizes (e.g. heading family "Söhne").
- **Spacing** — any explicit spacing/scale values the brand defines.
- **Other** — radii, elevation, or any other foundation values the brand material specifies.

For each element, decide a clean variable name within the new collection using a sensible path-style convention, e.g. `color/primary`, `color/accent`, `font/family/heading`, `spacing/base`. Do not copy names from existing variables in the file — this is a fresh collection with its own naming.

Build a proposed-element set where each entry records: proposed variable name, category (Color / Typography / Spacing / Other), Figma variable type (COLOR / STRING / FLOAT), and the brand value (hex for colors).

## Approval gate — required before creating anything

Present the proposed elements for review and wait for the user. This is the verification step teammates rely on, so never skip it. Show every proposed element as a **numbered list with checkbox markers**, all selected by default, grouped by category:

```
New collection: "Brand Foundations"

Colors
1. [x] color/primary        #003087
2. [x] color/accent         #FF5A1F

Typography
3. [x] font/family/heading  Söhne
4. [x] font/family/body     Inter
```

State the count plainly: "This will create 4 variables in a new collection. No existing variables will be touched." Then ask: **"Reply `all` to create every element, or tell me which line numbers to exclude (e.g. `skip 2`)."**

**Do not create anything in Figma until the user replies.** Create only the entries still marked `[x]` after their reply.

## Creating the collection in Figma

After approval:

1. Create a **new** variable collection with the confirmed name (default "Brand Foundations"). Do not add elements into an existing collection.
2. Add the approved elements as variables in that collection, each with the correct type. For COLOR variables, convert the hex brand value to RGBA in the **0–1 range** before writing.
3. When the underlying tool is `use_figma` with a `fileKey`, call it directly with a precise instruction: create a collection named X, then add these named variables of these types with these values. Be explicit that existing collections and variables must be left unchanged.

After creating, confirm what was made ("Created 'Brand Foundations' with 4 variables") and give the user a direct link back to the file so they can verify in Figma.

## Failure modes to handle gracefully

- **A collection with that name already exists**: do not merge into it silently. Tell the user and ask whether to pick a different name (e.g. "Brand Foundations 2") or add to the existing one — let them decide.
- **Ambiguous brand value** (e.g. a color given with no clear role): ask the user what to name it rather than guessing.
- **Brand material is vague or has no explicit values**: say so and ask for concrete values, rather than inventing any.

## What's next

After the flow completes and you've confirmed the result, show a brief "What's next" list. Show all three commands — the two others, plus the command just completed marked "(run again)" so the user can repeat the current flow. One line each:

- **`/modes-map`** — side-by-side mapping table between two variable modes.
- **`/brand-upload`** (run again) — create a "Brand Foundations" variable collection from brand guidelines.
- **`/component-updates`** — document component changes for developer handoff.
