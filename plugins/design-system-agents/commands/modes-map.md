---
description: Guided side-by-side mapping table between two Figma variable modes.
---

# Mode Mapping — guided flow

Run the **token-mode-mapping** skill in fully guided, step-by-step mode. The person invoking this command should not have to know how to prompt — you ask for what you need, one question at a time, and never push anything to Figma without explicit per-token approval.

Follow the skill at `skills/token-mode-mapping/SKILL.md`, and run it in this exact order. Ask **one question per turn** and wait for the answer before moving on:

1. **Preflight** — confirm the Figma MCP connection is live (probe the server). If it isn't, stop and guide the person to connect it (`/mcp` → authenticate the `figma` server). Do not continue until a probe returns real data.
2. **Ask for the Figma file** — the file/page that holds the variables. Extract the file key from the URL.
3. **Ask which two modes** to compare (e.g. Default and Dark, or a base mode and a theme mode). List the available modes from the file if you can, so they can just pick.
4. **Ask where the table should go** — the Figma page or frame link where the comparison table should be drawn.
5. **Read and resolve** all tokens across both modes (full alias resolution, compact serialization to avoid truncation). Report the token count so they can sanity-check it.
6. **Approval gate** — present the full token list as a numbered table with `[x]` markers (all selected by default) and ask them to reply `all` to push everything, or to name the lines to exclude. **Do not draw the table in Figma until they reply.**
7. **Build the table** on the page they specified, including only the approved tokens, then give them a direct link to the created frame.

If the person already passed details as arguments ($ARGUMENTS), use those to skip the matching questions, but still run the preflight and the approval gate.
