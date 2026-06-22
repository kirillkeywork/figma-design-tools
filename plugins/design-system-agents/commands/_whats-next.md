# What's next — shared footer

At the very end of any completed flow (after confirming the result and giving the Figma link), show this "What's next" block so the user can pick their next action. Show **all three commands**: the two the user has not just run, plus the command they *did* just run as a "run it again" option (label it clearly, e.g. "(run again)").

Render it as a short list, one line per command, no extra preamble. Mark the just-completed command with "(run again)" so it's clear it repeats the current flow.

Example, shown after a `/component-updates` run:

---

**What's next?** You can run:

- **`/modes-map`** — build a side-by-side mapping table between two variable modes in Figma.
- **`/brand-upload`** — create a "Brand Foundations" variable collection from brand guidelines.
- **`/component-updates`** (run again) — document changes for another component pair.

---

Keep it to this list. When a new command is added to the plugin, add one line here and every flow picks it up automatically.
