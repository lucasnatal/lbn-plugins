# Installing Plugin Builder for OpenCode

## Prerequisites

- [OpenCode.ai](https://opencode.ai) installed

## Installation

Add plugin-builder to the `plugin` array in your `opencode.json` (global or project-level):

```json
{
  "plugin": ["plugin-builder@git+https://github.com/lucasnatal/lbn-plugins.git#main:plugin-builder"]
}
```

Restart OpenCode. The plugin installs through OpenCode's plugin manager and
registers all skills.

Verify by asking: "Help me create a plugin"

OpenCode uses its own plugin install. If you also use Claude Code, Codex, or
another harness, install Plugin Builder separately for each one.

## Usage

Use OpenCode's native `skill` tool:

```
use skill tool to list skills
use skill tool to load plugin-builder/creating-a-plugin
```

## Updating

OpenCode installs Plugin Builder through a git-backed package spec. Some OpenCode
and Bun versions pin that resolved git dependency in a lockfile or cache, so a
restart may not pick up the newest commit. If updates do not appear, clear
OpenCode's package cache or reinstall the plugin.

To pin a specific version:

```json
{
  "plugin": ["plugin-builder@git+https://github.com/lucasnatal/lbn-plugins.git#v0.1.0:plugin-builder"]
}
```

## Troubleshooting

### Plugin not loading

1. Check logs: `opencode run --print-logs "hello" 2>&1 | grep -i plugin-builder`
2. Verify the plugin line in your `opencode.json`
3. Make sure you're running a recent version of OpenCode

### Windows install issues

Some Windows OpenCode builds have upstream installer issues with git-backed
plugin specs, including cache paths for `git+https` URLs and Bun not finding
`git.exe` even when it works in a normal terminal. If OpenCode cannot install
the plugin, try installing with system npm and pointing OpenCode at the local
package:

```powershell
npm install plugin-builder@git+https://github.com/lucasnatal/lbn-plugins.git --prefix "$HOME\.config\opencode"
```

Then use the installed package path in `opencode.json`:

```json
{
  "plugin": ["~/.config/opencode/node_modules/plugin-builder"]
}
```

### Skills not found

1. Use `skill` tool to list what's discovered
2. Check that the plugin is loading (see above)

### Tool mapping

When skills reference Claude Code tools:
- `TodoWrite` → `todowrite`
- `Task` with subagents → `@mention` syntax
- `Skill` tool → OpenCode's native `skill` tool
- File operations → your native tools

## Getting Help

- Report issues: https://github.com/lucasnatal/lbn-plugins/issues
