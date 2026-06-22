# Figma Design Tools

A Claude plugin marketplace for Figma design-system automation. Subscribe once, install the plugin, and use the tools through natural language in Claude Code or Cowork.

## For teammates — install

```
# 1. Add this marketplace (one time)
/plugin marketplace add kirillkeywork/figma-design-tools

# 2. Install the plugin
/plugin install design-system-agents@figma-design-tools
```

That's it. The easiest way to use the tools is the two guided commands — type one and Claude walks you through the rest, one question at a time:

| Command | What it does |
|---|---|
| `/modes-map` | Guided side-by-side mapping table between two Figma variable modes. Asks for the file, the two modes, and where to draw the table; shows you a numbered list to approve before drawing anything. |
| `/brand-upload` | Guided creation of a "Brand Foundations" variable collection from brand guidelines. Asks for your brand material and the file; shows you a numbered checklist to approve; then creates a new collection, leaving existing variables untouched. |
| `/component-updates` | Guided developer-handoff doc comparing a library component to a modified version. Classifies styling-only changes vs. an anatomy rebuild; shows a numbered checklist to approve; then creates a documentation section in Figma with code syntax for changed variable-bound values. |

Nothing is ever written to Figma until you approve the list. You can also just describe what you want in plain language if you prefer — see the [plugin README](./plugins/design-system-agents/README.md) for examples.

To get later updates:

```
/plugin marketplace update figma-design-tools
```

### One prerequisite: Figma

The tools talk to Figma through the Figma MCP server. You need it connected and authenticated to an account that can reach your files. The skills check this automatically and guide you if it's not ready — run `/mcp` to see the `figma` server status.

## What's in the marketplace

| Plugin | Commands | Skills | Purpose |
|---|---|---|---|
| `design-system-agents` | `/modes-map`, `/brand-upload`, `/component-updates` | `brand-token-update`, `token-mode-mapping`, `component-updates` | Map two variable modes into a comparison table; create a Brand Foundations collection from brand guidelines; document component changes for developer handoff. |

## For the maintainer — repo layout

```
figma-design-tools/
├── .claude-plugin/
│   └── marketplace.json          # the catalog teammates subscribe to
├── .gitignore
└── plugins/
    └── design-system-agents/
        ├── .claude-plugin/
        │   └── plugin.json        # plugin manifest + version
        ├── .mcp.json              # Figma MCP connection definition
        ├── commands/              # slash commands (the guided entry points)
        │   ├── modes-map.md       # /modes-map
        │   ├── brand-upload.md    # /brand-upload
        │   └── component-updates.md  # /component-updates
        ├── skills/
        │   ├── brand-token-update/SKILL.md
        │   ├── component-updates/SKILL.md
        │   └── token-mode-mapping/SKILL.md
        └── README.md
```

## For the maintainer — adding a feature later

Adding a new tool is intentionally small:

1. Create `plugins/design-system-agents/skills/<new-skill>/SKILL.md`. The `description` field is what makes Claude pick the skill up — describe both what it does and when to use it, and lean slightly "pushy" so it triggers when it should.
2. Bump `version` in **both** `plugin.json` and the plugin entry in `marketplace.json` (e.g. `0.1.0` → `0.2.0`).
3. Commit and push.
4. Teammates run `/plugin marketplace update figma-design-tools` to get it.

If you'd rather ship a feature as a *separate* plugin instead of a new skill, add another folder under `plugins/` and a second entry in `marketplace.json`.

To give a skill a guided slash-command entry point, add a matching file under `plugins/design-system-agents/commands/` — the filename becomes the command (e.g. `commands/my-tool.md` → `/my-tool`).
