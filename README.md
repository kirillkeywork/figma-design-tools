# Figma Design Tools

A Claude plugin marketplace for Figma design-system automation. Subscribe once, install the plugin, and use the tools through natural language in Claude Code or Cowork.

## For teammates — install

```
# 1. Add this marketplace (one time)
/plugin marketplace add kirillkeywork/figma-design-tools

# 2. Install the plugin
/plugin install design-system-agents@figma-design-tools
```

That's it. Then just describe what you want — see the [plugin README](./plugins/design-system-agents/README.md) for examples.

To get later updates:

```
/plugin marketplace update figma-design-tools
```

### One prerequisite: Figma

The tools talk to Figma through the Figma MCP server. You need it connected and authenticated to an account that can reach your files. The skills check this automatically and guide you if it's not ready — run `/mcp` to see the `figma` server status.

## What's in the marketplace

| Plugin | Skills | Purpose |
|---|---|---|
| `design-system-agents` | `brand-token-update`, `token-mode-mapping` | Update UI-kit variables from brand guidelines; map two variable modes into a comparison table. |

## For the maintainer — repo layout

```
figma-design-tools/
├── .claude-plugin/
│   └── marketplace.json          # the catalog teammates subscribe to
└── plugins/
    └── design-system-agents/
        ├── .claude-plugin/
        │   └── plugin.json        # plugin manifest + version
        ├── .mcp.json              # Figma MCP connection definition
        ├── skills/
        │   ├── brand-token-update/SKILL.md
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
