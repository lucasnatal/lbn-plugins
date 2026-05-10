# Plugin Bootstrap

> SessionStart eager-injection recipe. Consulted by execution-stage subagents implementing the bootstrap task. Pattern reference: `superpowers/hooks/session-start` and `superpowers/hooks/run-hook.cmd`.

## Pattern Explanation

By default, plugin skills are **lazy**. Only the YAML frontmatter (`name` + `description`) is injected into the system prompt at session start. The body of `SKILL.md` is read only when the model invokes the skill via the Skill tool. Lazy skills can be ignored — if the model never invokes, the skill never acts.

The **bootstrap pattern** flips this for one specific skill: a `SessionStart` hook reads the SKILL.md file and injects its full content into the system prompt as `additionalContext`. The skill becomes **eager** — present without invocation.

**Cost:** approximately ~600 tokens permanently per session (depends on SKILL.md size). The cost compounds across `/clear` and `/compact` because the hook fires on `startup|clear|compact` matchers.

**Benefit:** guaranteed activation. The model cannot ignore an instruction it has already read in the system prompt.

## When to Use Bootstrap

Use bootstrap only when **all** of these are true:

- The skill is an **orientation** or **discipline** skill — the model needs to apply it before deciding what else to do.
- The skill's trigger conditions are too broad or context-dependent for reliable lazy invocation.
- Missing the skill produces measurably worse outcomes (verified via pressure testing).
- The token cost is justified (you can articulate the trade-off to the user).

Do **not** use bootstrap when:

- The skill is narrow and technical (lazy invocation works fine).
- The skill is rarely needed.
- You haven't measured the cost vs. benefit.
- "Bootstrap is good practice" is your only justification — it isn't; it's a deliberate trade-off.

## hooks.json Structure

The `hooks.json` file (at `plugin/hooks/hooks.json`) declares which events trigger which scripts. For a SessionStart bootstrap:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
        "async": false
      }]
    }]
  }
}
```

Key points:
- `matcher: "startup|clear|compact"` — fires on initial load AND after `/clear` AND after compaction. All three need the SKILL.md re-injected.
- `command` quotes `${CLAUDE_PLUGIN_ROOT}` so paths with spaces work.
- `command` calls `run-hook.cmd <script-name>` (not the script directly) — this is the polyglot wrapper.
- `async: false` — block the harness until the hook completes (synchronous context injection).

A separate `hooks-cursor.json` exists for Cursor (different env var, possibly different output format expectations).

## Polyglot Wrapper

`run-hook.cmd` is a single file that serves as **bash on Unix** and **batch on Windows** simultaneously. The trick is `: << 'CMDBLOCK'` — `:` is a no-op in bash, so bash sees the heredoc as a comment block and falls through to the bash code below. cmd.exe ignores the leading `:` and executes the batch portion.

```cmd
: << 'CMDBLOCK'
@echo off
if "%~1"=="" (echo missing script name >&2 & exit /b 1)
set "HOOK_DIR=%~dp0"
if exist "C:\Program Files\Git\bin\bash.exe" (
    "C:\Program Files\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
where bash >nul 2>nul && (bash "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9 & exit /b %ERRORLEVEL%)
exit /b 0
CMDBLOCK

# Unix path: run the named script directly
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

Abridged for clarity — see `templates/hooks-polyglot.cmd.tmpl` for the full version with all bash search paths (Program Files, Program Files (x86), and PATH) and the comment explaining why scripts use extensionless filenames.

Hook scripts (e.g., `session-start`) are **extensionless** — Claude Code's Windows auto-detection prepends `bash` to any `.sh` filename, which conflicts with the wrapper.

## Session-Start Script

The hook script (`hooks/session-start`) is bash. It must:

1. Resolve `PLUGIN_ROOT` from its own location.
2. Read the target SKILL.md content.
3. Escape the content for embedding in JSON.
4. Detect the active harness via env vars.
5. Emit a JSON object on stdout in the format the harness expects.

Pattern (adapted from `superpowers/hooks/session-start`):

```bash
#!/usr/bin/env bash
set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"

skill_content=$(cat "${PLUGIN_ROOT}/skills/<skill-name>/SKILL.md" 2>&1 || echo "Error reading skill")

escape_for_json() {
    local s="$1"
    s="${s//\\/\\\\}"
    s="${s//\"/\\\"}"
    s="${s//$'\n'/\\n}"
    s="${s//$'\r'/\\r}"
    s="${s//$'\t'/\\t}"
    printf '%s' "$s"
}

skill_escaped=$(escape_for_json "$skill_content")
session_context="<EXTREMELY_IMPORTANT>\nYou have <plugin-name>.\n\n${skill_escaped}\n</EXTREMELY_IMPORTANT>"

# Emit per-harness format (see next section)
```

## Output JSON per Harness

Different harnesses consume different JSON shapes. Detect via env vars and emit only the format the active harness reads (some harnesses read multiple keys without deduplication, so emit only one).

| Harness | Detection | Output shape |
|---|---|---|
| Cursor | `CURSOR_PLUGIN_ROOT` set | `{"additional_context": "<content>"}` (snake_case, top-level) |
| Claude Code | `CLAUDE_PLUGIN_ROOT` set, `COPILOT_CLI` unset | `{"hookSpecificOutput": {"hookEventName": "SessionStart", "additionalContext": "<content>"}}` |
| Copilot CLI / SDK | `COPILOT_CLI` set, or unknown | `{"additionalContext": "<content>"}` (camelCase, top-level) |

Dispatch logic:

```bash
if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "additional_context": "%s"\n}\n' "$session_context"
elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$session_context"
else
  printf '{\n  "additionalContext": "%s"\n}\n' "$session_context"
fi
```

Use `printf` (not heredoc) — bash 5.3+ has a heredoc hang bug that affects this exact pattern.

## Variable Injection

Environment variables the harness sets, available inside the hook:

| Variable | Set by | Holds |
|---|---|---|
| `${CLAUDE_PLUGIN_ROOT}` | Claude Code | Absolute path to the plugin's root directory |
| `${CURSOR_PLUGIN_ROOT}` | Cursor | Absolute path to the plugin's root (Cursor's analog) |
| `${COPILOT_CLI}` | Copilot CLI v1.0.11+ | `1` if running under Copilot CLI |
| `${HOME}` | OS | User home (used to check for sibling-plugin install) |

Reference `${CLAUDE_PLUGIN_ROOT}` in `hooks.json` `command` strings; resolve it inside the hook script via `$(cd "$(dirname "$0")/.." && pwd)` for portability (some harnesses don't set the env var early enough).

## Cost Analysis

To estimate the per-session cost of a bootstrap:

1. Word-count the SKILL.md being injected: `wc -w skills/<name>/SKILL.md`.
2. Token estimate: `~1.3 × word count` (English prose; code blocks tokenize denser).
3. Add overhead for the wrapping `<EXTREMELY_IMPORTANT>` tag and any warning messages.

Typical orientation skill: 600-900 words → ~800-1200 tokens. The injection happens on `startup|clear|compact` — so every fresh context costs that amount.

**Compaction caveat:** the hook fires after `/compact`, which means the SKILL.md is re-injected into the compacted context. This is intentional — the skill's content survives compaction — but it also means the cost is permanent across long sessions, not just at startup.

To reduce cost without dropping bootstrap: shrink the SKILL.md (move detail into lazy reference docs), or split the skill into a small bootstrap-injected shell + lazy-loaded body. Aim for SKILL.md < 200 lines if you bootstrap it.
