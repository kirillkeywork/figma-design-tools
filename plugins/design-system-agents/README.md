# Design System Agents

Two Figma design-system tools in one Claude plugin. Once installed, you just describe what you want in plain language inside Claude Code or Cowork — the right skill activates automatically.

## What's inside

**Brand Token Update** — Update your UI kit's variables to match new brand guidelines. Give it a brand PDF (or pasted values), point it at your Figma file, and it reads your existing variables, maps the brand's explicit values onto them, shows you a before → after diff, and writes the approved changes back. It only changes variables the brand material actually specifies — no invented defaults.

**Token Mode Mapping** — Compare two modes of a variable collection (e.g. Default vs. a brand theme) and generate a native Figma auto-layout table mapping them side by side, with color swatches, resolved aliases, typography samples, and code syntax.

## Requirements

- Claude Code or Cowork (desktop or terminal).
- The **Figma MCP** connection, authenticated to an account that can access your files. Each skill checks this connection before doing any work and will walk you through connecting if it isn't ready — run `/mcp` to check the `figma` server status.

## Using it

Just talk to Claude. Examples:

- "Apply our new brand guidelines to the UI kit — here's the brand PDF and the Figma link."
- "Update the colors in this Figma file to match these hex values: …"
- "Compare the Default and Bonnell modes in our color collection and build a mapping table in Figma."
- "Map our two variable modes side by side so I can document the differences."

The skill will confirm the Figma connection, gather what it needs (file link, brand material or the two mode names), show you what it plans to change, and only write to Figma after you approve.

## Notes

- Both skills read variables with `get_variable_defs` and write through the Figma MCP. They never modify your file without showing you the change first and getting approval.
- For large files, reads are handled in a way that avoids truncation — you'll get the full set, not a partial one.
