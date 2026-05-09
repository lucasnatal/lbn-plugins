# Plugin Distribution

> Marketplace setup. Consulted in the finishing stage when the user chooses Option 2 (PR) and the plugin needs to be installable via `/plugin install`.

## marketplace.json Structure

A marketplace declares one or more plugins. The file lives at the marketplace repo's root: `<repo>/.claude-plugin/marketplace.json`.

Required fields: `name`, `owner`, `plugins[]`. Each entry in `plugins[]` requires `name`, `description`, `version`, `source`.

```json
{
  "name": "lbn-plugins",
  "description": "Lucas Bonetti Natal's personal Claude Code plugins.",
  "owner": {
    "name": "Lucas Bonetti Natal",
    "email": "bonettinatal@gmail.com"
  },
  "plugins": [
    {
      "name": "plugin-builder",
      "description": "Build Claude Code plugins using the canonical superpowers pipeline.",
      "version": "0.1.0",
      "source": "./plugin-builder",
      "author": {
        "name": "Lucas Bonetti Natal",
        "email": "bonettinatal@gmail.com"
      }
    }
  ]
}
```

`source` is a relative path from the marketplace repo root to the plugin directory. The `name` here must match the `name` field in the plugin's own `.claude-plugin/plugin.json`.

## Marketplace Modes

**Standalone.** No `marketplace.json`. The plugin lives at the repo root. Users install via `/plugin marketplace add <user>/<repo>` followed by `/plugin install <plugin-name>@<repo-name>`. Simplest setup; appropriate for one-off plugins or single-author small projects.

**Private marketplace.** A repo containing multiple plugins, with `marketplace.json` listing them all. Each plugin is in its own subdirectory. Users add the marketplace once, then install any plugin from it. This is the right pattern for multi-plugin authors (e.g., `lbn-plugins` lists multiple personal plugins).

**Public marketplace.** A community registry. Higher metadata bar — sponsorship, community guidelines, possibly review process. Consider only after the plugin has stabilized and proven useful beyond the author's own use.

## Install Path Semantics

Two-step install for marketplace-hosted plugins:

```bash
/plugin marketplace add <github-user>/<repo-name>
/plugin install <plugin-name>@<marketplace-name>
```

- `<github-user>/<repo-name>` is a GitHub coordinate (e.g., `lucasnatal/lbn-plugins`).
- `<marketplace-name>` is the `name` field from `marketplace.json` (often the repo name, but can differ).
- `<plugin-name>` is the plugin's `name` field (matching across `marketplace.json` and `.claude-plugin/plugin.json`).

Standalone plugin install (no separate marketplace.json): same two-step flow — `/plugin marketplace add <user>/<repo>` then `/plugin install <plugin-name>@<repo-name>`. The repo identifier doubles as the marketplace name.

For OpenCode users, distribution is via `git+https://github.com/<user>/<repo>.git` referenced in the consumer's `opencode.json`.

For Gemini CLI users, the extension is installed via `gemini-extension.json` symlinking or copying into the user's extensions directory.

## Version Bumping

`.version-bump.json` is an optional file at the plugin root that names a single source of truth for the plugin version. A small build/release script reads this file and rewrites every manifest's `version` field in lockstep:

- `package.json`
- `.claude-plugin/plugin.json`
- `.codex-plugin/plugin.json`
- `.cursor-plugin/plugin.json`
- `gemini-extension.json`
- The plugin's entry in the parent `marketplace.json` (if applicable)

Without `.version-bump.json`, version drift is the failure mode — one manifest gets bumped, others lag, and users get inconsistent metadata. Use it for any plugin shipping to 3+ harnesses.

## README Requirements

A public-facing `README.md` at the plugin root must contain:

1. **Title + one-line description** — the plugin's `description` field, verbatim.
2. **Quickstart** — the simplest install command for the most common harness (typically Claude Code: `/plugin install <name>@<marketplace>`).
3. **Installation per harness** — explicit instructions for each declared harness (Claude Code, Cursor, Codex, OpenCode, Gemini CLI). One subsection per harness.
4. **What's Inside** — list of skills, hooks, MCP servers, agents (if any). One paragraph each: name, purpose, when it activates.
5. **Philosophy** (optional but recommended) — the design principles the plugin follows. For plugins built on superpowers patterns, point to the upstream conventions.
6. **Contributing** — pointer to `CLAUDE.md` and the contributor pipeline.
7. **License** — name (e.g., MIT) and pointer to the LICENSE file.

Optional sections: Sponsorship, Community, Roadmap, Changelog, Acknowledgments. Add when relevant; omit when not.

The README is the first thing users see — keep it concrete and action-oriented. Examples and copy-paste install commands beat narrative essays.
