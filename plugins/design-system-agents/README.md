# Design System Agents

Two Figma design-system tools in one Claude plugin. Once installed, you just describe what you want in plain language inside Claude Code or Cowork — the right skill activates automatically.

## What's inside

**Brand Foundations** — Turn a brand document into a new Figma variable collection. Give it a brand PDF (or pasted values), point it at your Figma file, and it pulls out the explicit brand elements (colors, typography, spacing), shows you a checklist to approve, then creates a new collection named "Brand Foundations" (or a name you choose) holding those variables. It never touches your existing variables — the brand values live in their own collection.

**Component Updates** — Document what changed between a library component and a modified version, as a developer-facing handoff section in Figma. Give it the original and the modified component (or just the modified one for a plain spec), and it captures the differences. If only styling changed (color, typography, spacing, sizing, corner radii) it lists those property changes with the code syntax of any variable-bound value. If the anatomy changed, it flags that this is a different custom component, not a modification, and documents its spec instead.

**Token Mode Mapping** — Compare two modes of a variable collection (e.g. Default vs. a brand theme) and generate a native Figma auto-layout table mapping them side by side, with color swatches, resolved aliases, typography samples, and code syntax.

## Requirements

- Claude Code or Cowork (desktop or terminal).
- The **Figma MCP** connection, authenticated to an account that can access your files. Each skill checks this connection before doing any work and will walk you through connecting if it isn't ready — run `/mcp` to check the `figma` server status.

## Using it

The simplest way is the two guided commands — they walk you through everything, one question at a time, and always show you a checklist for approval before changing anything in Figma:

- **`/modes-map`** — guided mode comparison. Asks for the Figma file, which two modes to compare, and where to draw the table; resolves all tokens; shows you a numbered list to approve; then builds the table (with per-token code syntax) and links you to it.
- **`/brand-upload`** — guided brand foundations setup. Asks for your brand material (PDF or values) and the Figma file; shows you a numbered checklist of the brand elements to approve; then creates a new "Brand Foundations" collection with them, leaving existing variables untouched.
- **`/component-updates`** — guided component handoff doc. Asks for the original component (optional) and the modified one; classifies the change; shows you a numbered checklist to approve; then creates a documentation section in Figma explaining what a developer needs to implement, with code syntax for changed variable-bound values.

You can also just talk to Claude in plain language if you prefer — the skills trigger on their own. Examples:

- "Create a Brand Foundations collection from this brand PDF — here's the Figma link."
- "Document what changed between this library button and our modified version for the developers."
- "Compare the Default and Dark modes in our color collection and build a mapping table in Figma."

Either way, nothing is written to Figma until you approve the numbered list.

## Notes

- Neither skill modifies your file without showing you a numbered checklist first and getting your approval. The mode-mapping skill only adds a table; the brand skill only creates a new collection — existing variables are never changed.
- For large files, reads are handled in a way that avoids truncation — you'll get the full set, not a partial one.
