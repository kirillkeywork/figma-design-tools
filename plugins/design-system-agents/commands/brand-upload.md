---
description: Guided creation of a "Brand Foundations" variable collection from brand guidelines.
---

# Brand Foundations — guided flow

Run the **brand-token-update** skill in fully guided, step-by-step mode. The person invoking this command should not have to know how to prompt — you ask for what you need, one question at a time, and never create anything in Figma without explicit per-element approval.

This tool **creates a new variable collection** (named "Brand Foundations" by default) from the brand material. It must **never modify existing variables** — they are entirely out of scope.

Follow the skill at `skills/brand-token-update/SKILL.md`, and run it in this exact order. Ask **one question per turn** and wait for the answer before moving on:

1. **Preflight** — confirm the Figma MCP connection is live (probe the server). If it isn't, stop and guide the person to connect it (`/mcp` → authenticate the `figma` server). Do not continue until the probe succeeds.
2. **Ask for the brand material** — a brand PDF they can attach, pasted brand-guideline text, or a short list of values (hex codes, fonts, spacing). Wait until you have something concrete.
3. **Ask for the Figma file** — where the new collection will be created. Extract the file key from the URL.
4. **Confirm the collection name** — default "Brand Foundations", or let them set a different one.
5. **Extract the brand elements** from the material (colors, typography, spacing, other) — only explicit values, never invented ones.
6. **Approval gate** — present every proposed element as a numbered list with `[x]` markers (all selected by default), grouped by category. State plainly that it will create N variables in a new collection and that no existing variables will be touched. Ask them to reply `all` or to name the lines to exclude. **Do not create anything until they reply.**
7. **Create the new collection** with only the approved elements (convert hex → RGBA 0–1 for colors), leaving all existing collections and variables unchanged. Then confirm what was made and give them a link to the file.
8. **What's next** — after confirming the result, show the shared "What's next" footer from `commands/_whats-next.md` (omitting `/brand-upload` since it was just run) so the user discovers the other tools.

If the person already passed details as arguments ($ARGUMENTS), use those to skip the matching questions, but still run the preflight and the approval gate.
