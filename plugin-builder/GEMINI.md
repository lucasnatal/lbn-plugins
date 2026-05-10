# Gemini CLI Notes

This plugin's skills use Claude Code tool vocabulary. When used from Gemini CLI, adapt tool references:

- `TodoWrite` → Gemini's `todo` tool (or inline checklist if unavailable)
- `Task` with subagents → Gemini's `@mention` syntax for sub-tasks
- `Skill` tool → `activate_skill` tool in Gemini CLI

For full mapping, see superpowers' `GEMINI.md` at the superpowers installation
and `skills/using-superpowers/references/gemini-tools.md`.

**Note:** The bootstrap hook (SessionStart) injects `creating-a-plugin` at session
start. This requires Gemini CLI to support the `contextFileName` extension field
(`gemini-extension.json`).
