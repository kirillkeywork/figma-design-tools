---
name: brand-token-update
description: Update a Figma UI kit's design-token variables to match new brand guidelines. Use this whenever the user wants to rebrand, re-skin, or re-theme a Figma file, apply brand colors/typography/spacing to existing variables, map a brand document (PDF, brand-guideline text, or hex/font values) onto a UI kit, or push updated token values into Figma. Trigger even when the user does not say "token" explicitly — phrases like "apply our new brand to the UI kit", "update the colors in our Figma file", or "make the design system match this brand doc" should all use this skill. Reads existing variables via the Figma MCP, maps brand values onto them, shows a diff for approval, then writes the changes back.
---

# Brand Token Update

Map values from a brand document onto the existing variables in a Figma UI kit, and push the approved changes back into Figma. The guiding principle is **never invent**: only change a variable when the brand material gives an explicit value for it. A four-color brand doc produces a handful of changes, not eighty.

## Preflight: confirm the Figma connection before any work

This skill depends entirely on the Figma MCP server. Before doing anything else, confirm it is connected and working, because failing early with a clear message is far better than producing hallucinated variable data.

1. Tell the user you need the Figma connection and ask them to confirm the file is open/shared with the connector.
2. Make one cheap probe call — `get_variable_defs` (or `get_metadata`) against the target file key. If it returns real data, proceed. If it errors or returns nothing, stop and guide them:
   - In Claude Code / Cowork, the Figma MCP must be authenticated. Point them to run `/mcp` to check the `figma` server status and authenticate if it shows disconnected.
   - Remind them the file must be accessible to the Figma account they authenticated with.
3. Do not proceed to mapping until a probe call returns actual variables. If you cannot read variables, say so plainly rather than guessing values.

## Inputs to gather

When launched via the `/brand-upload` command, gather these **one question at a time**, conversationally — the user should never need to know how to prompt. Confirm each answer before asking the next:

- **Brand material** — any of: a PDF the user attaches, pasted brand-guideline text, or a short list of values (hex codes, font families, spacing). Read what they give; do not fetch from the web unless they explicitly ask and provide a URL. Ask for this first — there is no point fetching variables you have nothing to map onto.
- **Figma file** — the UI kit. Accept a full Figma URL and extract the file key (the segment after `/design/` or `/file/`, e.g. `https://figma.com/design/ABC123/Name` → `ABC123`).
- Optionally, a specific collection name if they only want to touch one collection.

## Reading variables from Figma

Use **`get_variable_defs`** with the file key. This is the correct read tool. Do **not** use `get_local_variables` — it returns no results on this MCP server.

The response arrives as nested content blocks. Variable data lives inside `mcp_tool_result` blocks, not plain text — when parsing, read both `text` blocks and `mcp_tool_result` blocks, and for tool-result blocks extract the nested content text.

The data shape is a nested object keyed by variable ID, not a flat array:

```
{ "meta": { "variables": {
  "VariableID:123": { "name": "...", "resolvedType": "COLOR", "valuesByMode": { ... } }
} } }
```

Color values come as RGBA objects `{ r, g, b, a }` with each channel in the **0–1 range**. Convert to hex for display and for showing the diff to the user (`r*255` rounded, etc.).

## Mapping brand values onto variables

Match each explicit brand value to the variable(s) it belongs on, by name and meaning. Examples: a brand "Primary" color maps onto variables named like `color/brand/primary` or `colors/primary`; a heading font maps onto `font/family/heading`.

Hard rules, because fidelity is the whole point:

- Only produce a change when the brand material contains an explicit value for that variable. If the brand doc lists four colors and one font, you produce changes for those, and nothing else.
- Never generate default or placeholder values to "fill out" a collection.
- Keep `name` and `id` exactly equal to what `get_variable_defs` returned — never rename or invent IDs.
- Mark a variable changed only when the new value actually differs from the current value.

Build a change set where each entry records: variable id, exact name, collection, category (Color / Typography / Spacing / etc.), current value (hex for colors), and the new brand value.

## Approval gate — required before writing anything

Present the change set for review and wait for the user. This is the verification step teammates rely on, so never skip it. Show every proposed change as a **numbered list with checkbox markers**, all selected by default, grouped by category, with current → new values (hex for colors):

```
Colors
1. [x] color/brand/primary      #003087 → #171717
2. [x] color/brand/accent       #2563EB → #FF5A1F

Typography
3. [x] font/family/heading      Inter → Söhne
```

State the count plainly: "3 of 47 variables will change." Then ask: **"Reply `all` to apply every change, or tell me which line numbers to exclude (e.g. `skip 2`)."**

**Do not write to Figma until the user replies.** Writing variable values is a side-effecting action on their real file. Apply only the entries still marked `[x]` after their reply.

## Writing changes back to Figma

After approval, push only the approved entries. When the underlying tool is `use_figma` with a `fileKey`, call it directly with a precise instruction describing exactly which variables to set to which values. For COLOR variables, convert the hex brand value **back** to RGBA in the 0–1 range before writing (the inverse of the read conversion).

After writing, confirm what changed and offer the user a direct link back to the file so they can verify in Figma.

## Failure modes to handle gracefully

- **Truncated reads** on large files: variable payloads can get cut off. If the parsed set looks incomplete, fetch in batches or request a compact serialization, then continue.
- **No matching variable** for a brand value: report it as "no variable found for X" rather than forcing it onto an unrelated token.
- **Ambiguous match** (two plausible variables): ask the user which one, don't guess.
