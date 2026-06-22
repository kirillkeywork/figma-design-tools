---
description: Guided update of Figma UI-kit variables from brand guidelines.
---

# Brand Token Update — guided flow

Run the **brand-token-update** skill in fully guided, step-by-step mode. The person invoking this command should not have to know how to prompt — you ask for what you need, one question at a time, and never write anything to Figma without explicit per-token approval.

Follow the skill at `skills/brand-token-update/SKILL.md`, and run it in this exact order. Ask **one question per turn** and wait for the answer before moving on:

1. **Preflight** — confirm the Figma MCP connection is live (probe the server). If it isn't, stop and guide the person to connect it (`/mcp` → authenticate the `figma` server). Do not continue until a probe returns real data.
2. **Ask for the brand material** — a brand PDF they can attach, pasted brand-guideline text, or a short list of values (hex codes, fonts, spacing). Wait until you have something concrete.
3. **Ask for the Figma file** — the UI kit whose variables should be updated. Extract the file key from the URL.
4. **Read the existing variables**, then **map** only the explicit brand values onto matching variables. Never invent values to fill out a collection.
5. **Approval gate** — present every proposed change as a numbered before → after table with `[x]` markers (all selected by default), grouped by category, showing current value → new brand value. State the count plainly (e.g. "3 of 47 variables will change"). Ask them to reply `all` to apply everything, or to name the lines to exclude. **Do not write to Figma until they reply.**
6. **Write the approved changes** back to Figma (convert hex → RGBA 0–1 for colors), then confirm what changed and give them a link to the file.

If the person already passed details as arguments ($ARGUMENTS), use those to skip the matching questions, but still run the preflight and the approval gate.
