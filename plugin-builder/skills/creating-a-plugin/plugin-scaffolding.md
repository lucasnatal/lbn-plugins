# Plugin Scaffolding

> Recipe for generating a plugin's physical structure. Consulted by `superpowers:writing-plans` during plan creation, and by subagents during scaffolding tasks.

## Standard Root Layout

The directory tree observed across mature multi-harness plugins:

```
<plugin-name>/
├── .claude-plugin/
│   └── plugin.json              # Claude Code manifest
├── .codex-plugin/
│   └── plugin.json              # Codex manifest (with interface block)
├── .cursor-plugin/
│   └── plugin.json              # Cursor manifest
├── .opencode/
│   ├── INSTALL.md               # Install instructions for OpenCode users
│   └── plugins/
│       └── <plugin-name>.js     # OpenCode ESM entry point
├── gemini-extension.json        # Gemini CLI manifest (root-level)
├── package.json                 # For npm/OpenCode distribution
│
├── hooks/                       # Only if plugin uses hooks
│   ├── hooks.json               # Claude Code + Copilot CLI
│   ├── hooks-cursor.json        # Cursor variant
│   ├── run-hook.cmd             # Polyglot bash + batch wrapper
│   └── <event>                  # Hook scripts (extensionless)
│
├── skills/
│   └── <skill-name>/
│       ├── SKILL.md
│       ├── <reference-doc>.md   # Lazy refs (referenced as prose)
│       ├── <role>-prompt.md     # Subagent prompt templates
│       └── templates/           # File templates for execution tasks
│
├── evals/                       # Pressure scenarios for discipline skills
│   └── <category>/
│       └── scenario-*.md
│
├── assets/                      # Icons, branding (referenced in Codex manifest)
│   ├── <plugin>-small.svg
│   └── app-icon.png
│
├── AGENTS.md → CLAUDE.md        # Symlink
├── CLAUDE.md                    # Contributor guidelines
├── GEMINI.md                    # Gemini CLI override notes
├── README.md                    # Public README
├── LICENSE
├── .gitignore
└── .version-bump.json           # Optional: single source of version
```

## Per-Harness Manifest Paths

| Harness | Path | Required fields | Optional | Template |
|---|---|---|---|---|
| Claude Code | `.claude-plugin/plugin.json` | `name`, `version`, `description` | `author`, `homepage`, `repository`, `license`, `keywords` | `claude-plugin.json.tmpl` |
| Codex | `.codex-plugin/plugin.json` | `name`, `version`, `description`, `interface` | `author`, `keywords`, `skills` | `codex-plugin.json.tmpl` |
| Cursor | `.cursor-plugin/plugin.json` | `name`, `version`, `description`, `skills` | `displayName`, `hooks`, `keywords` | `cursor-plugin.json.tmpl` |
| OpenCode | `.opencode/plugins/<name>.js` | ESM `export default` | `package.json`'s `main` field points here | `opencode-plugin.js.tmpl` |
| Gemini CLI | `gemini-extension.json` | `name`, `version`, `description`, `geminiMdFiles` | `skills` | `gemini-extension.json.tmpl` |

`name` is the same string across all manifests. `version` should be kept in sync via `.version-bump.json` or a manual sync script.

## File Creation Order

Order matters because later files reference earlier ones, and verification at each step prevents drift:

1. **Manifests first** — establish the plugin identity. JSON parses are fastest to verify.
2. **Skills** — SKILL.md per declared skill, then reference docs and prompt templates.
3. **Hooks** (if declared) — `hooks.json`, `hooks-cursor.json`, polyglot wrapper, hook scripts. Make scripts executable (`chmod +x`).
4. **Documentation** — `CLAUDE.md`, `AGENTS.md` symlink, `GEMINI.md`, full `README.md` (after everything else exists so descriptions are accurate).
5. **Evals** — pressure scenarios for each discipline skill.
6. **Assets** — placeholder icons; refine in Phase 2.
7. **`.gitignore`** last — only add when there are real artifacts to ignore.

Manifests written first because every other file may need to be referenced from them (e.g., Codex manifest's `composerIcon` field points to `assets/<plugin>-small.svg`). Writing the manifest first means the references can be verified as files are added.

## Path Conventions

- **Lowercase only.** Plugin names, skill names, file names. No camelCase, no PascalCase.
- **Hyphens, not underscores.** `creating-a-plugin`, not `creating_a_plugin`.
- **Forward slashes always.** Even on Windows, manifests and JSON config use `/`.
- **No spaces in paths.** Never. Polyglot wrappers and hook scripts assume safe path resolution.
- **`skills/<name>/SKILL.md`** is the only correct location for a skill. Never `commands/<name>.md` (deprecated).
- **No `agents/` directory** unless the subagent is genuinely reusable across many contexts. Default to prompt templates inside skills.
- **Hooks are extensionless.** `hooks/session-start`, not `hooks/session-start.sh`. Claude Code's Windows auto-detection prepends `bash` to any `.sh` filename, which conflicts with the polyglot wrapper.

## Auxiliary Files

**`package.json`** — required for OpenCode distribution. Minimal form:

```json
{
  "name": "<plugin-name>",
  "version": "0.1.0",
  "type": "module",
  "main": ".opencode/plugins/<plugin-name>.js"
}
```

`type: "module"` ensures `.js` files are treated as ESM. `main` points the npm/OpenCode resolver at the entry script.

**`AGENTS.md` symlink to `CLAUDE.md`** — Codex looks for `AGENTS.md`; Claude Code looks for `CLAUDE.md`. A symlink keeps the content single-source. Create with:

```bash
cd <plugin-name>
ln -s CLAUDE.md AGENTS.md
```

Verify: `ls -la AGENTS.md` should show `AGENTS.md -> CLAUDE.md`.

**`GEMINI.md`** — Gemini CLI uses different tool names than Claude Code. This file documents the mapping for users invoking the plugin from Gemini. Keep short — point to per-skill `references/gemini-tools.md` if more detail is needed.

**`.version-bump.json`** — optional; declares the single source of truth for the plugin version. A small script reads this and rewrites every manifest's `version` field. Useful when the plugin ships to 5+ harnesses.

## Plan Task Template: Implement a Skill

When `superpowers:writing-plans` creates a task for implementing a skill, the task template differs by skill type (see `plugin-anatomy.md` — "Skill Types").

**Reference / technique skill task:**
```
Task: Implement skill <name>
  Step 1: Invoke superpowers:brainstorming with stub from spec
  Step 2: Invoke superpowers:writing-skills → write SKILL.md
  Step 3: Verify structure (line count, H2 sections, no TBD placeholders)
  Step 4: Commit
```

**Discipline-enforcing skill task** (has Iron Law / Red Flags / Rationalization tables):
```
Task: Implement discipline skill <name>
  Step 0 (RED): Dispatch adversarial subagent WITHOUT the SKILL.md.
                Use pressure scenario from plugin-anatomy.md Skill Types section.
                Document verbatim rationalizations used to justify bypass.
  Step 1: Invoke superpowers:brainstorming with stub + RED findings as input
  Step 2: Invoke superpowers:writing-skills → write SKILL.md addressing
          the specific rationalizations found in Step 0 (not hypothetical ones)
  Step 3 (GREEN): Re-dispatch same pressure scenario WITH skill.
                  If agent bypasses → update Red Flags/Rationalizations → re-test.
                  Repeat until agent complies under maximum pressure.
  Step 4: Commit
```

If the plan has "write SKILL.md with these sections" without the RED phase — it was written without plugin-anatomy.md context. Push back and add the missing steps before execution.

## Templates Available

Templates live in `<skill>/templates/` and are consumed by execution-stage subagents. Each template uses `{{PLACEHOLDER}}` tokens that the controller fills before writing the real file:

| Template | Used by task | Placeholders include |
|---|---|---|
| `claude-plugin.json.tmpl` | Manifest task (Claude Code) | `{{NAME}}`, `{{DESCRIPTION}}`, `{{VERSION}}`, `{{AUTHOR_NAME}}`, `{{AUTHOR_EMAIL}}` |
| `codex-plugin.json.tmpl` | Manifest task (Codex) | All of above + `{{DISPLAY_NAME}}`, `{{BRAND_COLOR}}`, `{{COMPOSER_ICON}}`, `{{LOGO}}`, `{{DEFAULT_PROMPTS_JSON}}` |
| `cursor-plugin.json.tmpl` | Manifest task (Cursor) | All of above + `{{HAS_HOOKS}}` (bool) |
| `opencode-plugin.js.tmpl` | Manifest task (OpenCode) | `{{NAME}}`, `{{VERSION}}` |
| `gemini-extension.json.tmpl` | Manifest task (Gemini CLI) | `{{NAME}}`, `{{VERSION}}`, `{{DESCRIPTION}}` |
| `hooks-polyglot.cmd.tmpl` | Hook task (if bootstrap declared) | none — file is plugin-agnostic |
| `session-start-hook.tmpl` | Hook task (if bootstrap declared) | `{{SKILL_FILE_PATH}}`, `{{ACTIVATION_MESSAGE}}`, `{{PLUGIN_NAME}}`, `{{SKILL_NAME}}`, `{{LEGACY_CHECK_BLOCK}}` (optional) |
| `README.md.tmpl` | Documentation task | `{{NAME}}`, `{{ONE_LINE_DESCRIPTION}}`, `{{HOW_IT_WORKS_PARAGRAPH}}`, `{{MARKETPLACE}}`, install instructions per harness, `{{SKILLS_LIST}}`, `{{PHILOSOPHY_PARAGRAPH}}`, `{{LICENSE}}` |

When dispatching a subagent for a manifest task, pass the template path and the placeholder values as part of the prompt. The subagent fills, writes the real file, and verifies JSON validity (`python3 -m json.tool`).
