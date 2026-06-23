---
description: Guided documentation of what changed between a library component and a modified version, for developer handoff.
---

# Component Updates — guided flow

Run the **component-updates** skill in fully guided, step-by-step mode. The person invoking this command should not have to know how to prompt — you ask for what you need, one question at a time, and never write anything to Figma without explicit approval.

The goal is a clear, developer-facing doc section in Figma explaining exactly what was customized on a component against its library original — including the code syntax of any changed variable-bound values.

Follow the skill at `skills/component-updates/SKILL.md`, and run it in this exact order. Ask **one question per turn** and wait for the answer before moving on:

1. **Preflight** — confirm the Figma MCP connection is live (probe the server). If it isn't, stop and guide the person to connect it (`/mcp` → authenticate the `figma` server). Do not continue until the probe succeeds.
2. **Ask for the initial (library) component** — OPTIONAL. Tell them they can paste a link to the original component, or skip it if they just want a spec of the component. Accept a Figma node link (use "Copy link to selection" so it includes the node id).
3. **Ask for the modified / target component** — REQUIRED. The component to document. Accept a Figma node link.
4. **Ask where the doc should go** — a page or frame link. Default to a frame titled "Component Updates".
5. **Read both components** (handle same-file or cross-file), capture anatomy, styling, bound variables with any code syntax, and **detect variant properties / states** (e.g. State, Size, Type) for the example matrix.
6. **Classify** — anatomy unchanged & only styling differs = Modified; anatomy changed = Different/custom component (say so plainly, with the anatomical reasons); no original given = Spec only.
7. **Approval gate** — present the findings as a numbered list with `[x]` markers (all selected by default), grouped by category. Ask them to reply `all` or to name lines to exclude. **Do not write anything until they reply.**
8. **Create the documentation section** with only the approved items, including code syntax for changed variable-bound values, then give them a link to the doc frame.
9. **What's next** — after confirming the result, show the shared "What's next" footer from `commands/_whats-next.md` (showing all three, with `/component-updates` marked "(run again)") so the user discovers the other tools.

If the person already passed details as arguments ($ARGUMENTS), use those to skip the matching questions, but still run the preflight and the approval gate.
