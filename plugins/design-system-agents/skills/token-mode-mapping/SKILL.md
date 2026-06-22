---
name: token-mode-mapping
description: Compare two modes of a Figma variable collection and generate a native Figma auto-layout table mapping them side by side. Use this whenever the user wants to compare, map, audit, or document the differences between two variable modes (e.g. Default vs. Dark, Light vs. a brand theme, Mode 1 vs. Mode 2), build a token comparison or mapping table in Figma, or see how each token resolves across modes including aliases and code syntax. Trigger on phrases like "map our two modes", "compare Default and Dark tokens", "build a token mapping table", or "show me how tokens differ between modes" — even without the word "token". Reads variables across both modes via the Figma MCP, resolves aliases, then pushes a formatted comparison table into the file.
---

# Token Mode Mapping

Read every variable in a collection across two named modes, resolve each value (including alias chains and external library references), and generate a native Figma auto-layout table that maps the two modes side by side.

## Preflight: confirm the Figma connection before any work

This skill depends entirely on the Figma MCP server. Confirm it works before reading anything, so you never produce hallucinated token data.

1. Ask the user to confirm the file is open/shared with the Figma connector.
2. Probe with `get_variable_defs` (or `get_metadata`) against the file key. Proceed only if real data comes back.
3. If it errors or is empty, stop and guide them: in Claude Code / Cowork, run `/mcp` to check the `figma` server and authenticate if disconnected; confirm the file is reachable by the authenticated account. Do not guess token values.

## Inputs to gather

When launched via the `/modes-map` command, gather these **one question at a time**, conversationally — the user should never need to know how to prompt. Confirm each answer before asking the next:

- **Figma file / page** — accept a URL and extract the file key (segment after `/design/` or `/file/`).
- **Two mode names** — exactly which two modes to compare (e.g. "Default" and "Dark", or a base mode and a theme mode). The collection may have more than two modes; the user picks two. If you can read the available mode names from the file, list them so the user just picks.
- **Table destination** — the Figma page or frame link where the comparison table should be drawn.
- Optionally a specific collection name to limit scope.

## Selecting collections

A collection qualifies if it **contains both** requested modes. Filter with a "contains" check across the collection's modes — `modes.some(m => m.name === modeName)` for each of the two names — **not** a strict two-mode count. Collections frequently have three or more modes (e.g. a base mode plus several theme modes), and a count check wrongly rejects them.

## Reading and resolving values

Use **`get_variable_defs`** to read variables. Do not use `get_local_variables`.

Resolving a value to its real hex/string is the hard part, because a mode's value often **aliases into another variable**, sometimes in an external library, and chains can be two to three levels deep:

- Local resolution alone is insufficient — a value can reference an external library variable whose ID carries a hash prefix (a long hex string followed by `/` and a node id).
- Resolve by following the reference: get the referenced variable by ID, get its collection, walk that collection's modes to find the one matching the target mode name, and read its value. If the target mode name does not exist in the referenced collection, fall back to that collection's first mode.
- Repeat until you reach a concrete value (color, number, string), capturing both the resolved value and the alias name to display.

Colors arrive as RGBA `{ r, g, b, a }` in the 0–1 range; convert to hex for display.

## Avoiding truncation on large collections

Large collections (e.g. ~291 color tokens) overflow the MCP response and get truncated partway (often around token ~57) if you request verbose named objects per token. Use a **compact tuple serialization** so the full dataset fits in one response. Carry each code syntax separately so the table can later show whichever ones exist:

```
[id, name, mode1Hex, mode1Alias, mode2Hex, mode2Alias, webSyntax, iosSyntax, androidSyntax]
```

Use an empty string for any syntax a token does not define — this is how the table knows which syntaxes to render per row. If a read still truncates, batch the fetch and stitch the batches together before building the table.

## Approval gate — required before drawing anything

After resolving all tokens and before touching Figma, present the full list for review and wait for the user. This is the verification step teammates rely on, so never skip it.

- Report the token count first so they can sanity-check it (e.g. "Resolved 291 tokens across both modes").
- Present the tokens as a **numbered list with checkbox markers**, all selected by default:

  ```
  1. [x] color/brand/primary        Mode A #003087 → Mode B #1A1A1A
  2. [x] color/brand/secondary      Mode A #6699CC → Mode B #444444
  3. [x] color/surface/background   Mode A #FFFFFF → Mode B #0E0E0E
  ...
  ```

- Then ask plainly: **"Reply `all` to include every token, or tell me which line numbers to exclude (e.g. `skip 4, 17, 22`)."**
- Build the table only with the tokens still marked `[x]` after their reply. Do not draw the table until they answer.

## Table specification

Generate a native Figma auto-layout frame. Unless the user asks otherwise, follow this spec:

- **Section headers** grouping rows by variable type, then by collection.
- **Color rows**: a 40×40 swatch for each mode, the resolved hex, and the alias name.
- **Typography rows**: an "Aa" sample plus the font family.
- **Code syntax** — this is required and was dropped in an earlier version, so treat it as non-optional. For each token, include the platform code syntaxes (Web / iOS / Android) **stacked within the row**, but include **only the syntaxes that token actually defines**. If a token has Web and iOS but no Android, show Web and iOS and omit Android — never invent or pad a missing syntax with a placeholder. A token with no code syntax at all simply shows none.
- One column per mode, clearly labeled with the mode names, mapped side by side.

## Generating the table in Figma

Pre-resolve all token data first (including the per-token code syntaxes from the compact tuple), then push the table in one operation. When the tool is `use_figma` with a `fileKey`, call it directly with a precise description of the frame: the rows, the two mode columns, swatch sizes, grouping, and the stacked code-syntax cells showing only the syntaxes each token defines.

Building the table is a side-effecting write. **Do not draw it until the user has approved the token list** (see the approval gate below). After creation, give the user a direct link to the created frame so they can find it in the file.

## Failure modes to handle gracefully

- **Alias dead-ends**: if a chain cannot resolve, show the last known alias name and mark the value unresolved rather than inventing a hex.
- **Mode missing in a referenced collection**: fall back to the first mode and note it.
- **Truncated read**: detect an incomplete set and re-fetch in batches before drawing the table.

## What's next

After the flow completes and you've confirmed the result, show a brief "What's next" list so the user discovers the plugin's other tools. List the *other* commands (omit the one just used), one line each:

- **`/modes-map`** — side-by-side mapping table between two variable modes.
- **`/brand-upload`** — create a "Brand Foundations" variable collection from brand guidelines.
- **`/component-updates`** — document component changes for developer handoff.
