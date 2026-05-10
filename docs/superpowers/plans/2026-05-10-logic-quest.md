# Logic Quest Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `logic-quest` plugin — a gamified predicate-logic learning platform where students earn XP, level up, and progress through 5 grimoire chapters guided by Aldric the Wizard.

**Architecture:** Multi-skill pipeline (`start → quest → exercise → profile`) with a SessionStart bootstrap that injects the `start` skill eagerly every session. Progress persists in `~/.logic-quest/profile.json` via Bash. Five harnesses supported (Claude Code, Cursor, Codex, OpenCode, Gemini CLI) via parallel manifests.

**Tech Stack:** Markdown (SKILL.md), JSON (manifests), Bash (hooks + file I/O), ESM JavaScript (OpenCode entry point)

**Working directory for all commands:** `/Users/lucasnatal/Developer/Plugins/lbn-plugins`

**Spec:** `docs/superpowers/specs/2026-05-10-logic-quest-design.md`

---

### Task 1: Scaffold directory structure

**Files:**
- Create: `logic-quest/` and all subdirectories

- [ ] **Step 1: Create directory tree**

```bash
mkdir -p logic-quest/{.claude-plugin,.cursor-plugin,.codex-plugin,.opencode/plugins,hooks,skills/start,skills/quest,skills/exercise,skills/profile,evals/start,assets}
```

- [ ] **Step 2: Verify structure**

```bash
find logic-quest -type d | sort
```

Expected output includes:
```
logic-quest
logic-quest/.claude-plugin
logic-quest/.codex-plugin
logic-quest/.cursor-plugin
logic-quest/.opencode
logic-quest/.opencode/plugins
logic-quest/assets
logic-quest/evals
logic-quest/evals/start
logic-quest/hooks
logic-quest/skills
logic-quest/skills/exercise
logic-quest/skills/profile
logic-quest/skills/quest
logic-quest/skills/start
```

- [ ] **Step 3: Commit skeleton**

```bash
git add logic-quest/
git commit -m "chore(logic-quest): scaffold directory structure"
```

---

### Task 2: Claude Code manifest

**Files:**
- Create: `logic-quest/.claude-plugin/plugin.json`

- [ ] **Step 1: Write manifest**

Create `logic-quest/.claude-plugin/plugin.json`:

```json
{
  "name": "logic-quest",
  "description": "Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform with quests, XP, levels, and persistence.",
  "version": "0.1.0",
  "author": {
    "name": "Lucas Bonetti Natal",
    "email": "bonettinatal@gmail.com"
  },
  "license": "MIT",
  "keywords": ["gamification", "logic", "education", "predicate-logic", "learning"]
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool logic-quest/.claude-plugin/plugin.json
```

Expected: valid JSON echoed (no error).

- [ ] **Step 3: Commit**

```bash
git add logic-quest/.claude-plugin/plugin.json
git commit -m "feat(logic-quest): add Claude Code manifest"
```

---

### Task 3: Cursor manifest

**Files:**
- Create: `logic-quest/.cursor-plugin/plugin.json`

- [ ] **Step 1: Write manifest**

Create `logic-quest/.cursor-plugin/plugin.json`:

```json
{
  "name": "logic-quest",
  "displayName": "Logic Quest",
  "description": "Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform with quests, XP, levels, and persistence.",
  "version": "0.1.0",
  "author": {
    "name": "Lucas Bonetti Natal",
    "email": "bonettinatal@gmail.com"
  },
  "license": "MIT",
  "keywords": ["gamification", "logic", "education", "predicate-logic", "learning"],
  "skills": "./skills/",
  "hooks": "./hooks/hooks-cursor.json"
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool logic-quest/.cursor-plugin/plugin.json
```

Expected: valid JSON echoed.

- [ ] **Step 3: Commit**

```bash
git add logic-quest/.cursor-plugin/plugin.json
git commit -m "feat(logic-quest): add Cursor manifest"
```

---

### Task 4: Codex manifest

**Files:**
- Create: `logic-quest/.codex-plugin/plugin.json`

- [ ] **Step 1: Write manifest**

Create `logic-quest/.codex-plugin/plugin.json`:

```json
{
  "name": "logic-quest",
  "version": "0.1.0",
  "description": "Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform with quests, XP, levels, and persistence.",
  "author": {
    "name": "Lucas Bonetti Natal",
    "email": "bonettinatal@gmail.com"
  },
  "license": "MIT",
  "keywords": ["gamification", "logic", "education", "predicate-logic", "learning"],
  "skills": "./skills/",
  "interface": {
    "displayName": "Logic Quest",
    "shortDescription": "Gamified predicate logic learning with quests and XP",
    "longDescription": "logic-quest transforms your AI assistant into Aldric the Wizard, a gamified guide through predicate logic and quantifiers. Progress through 5 grimoire chapters, earn XP, level up, and unlock achievements.",
    "developerName": "Lucas Bonetti Natal",
    "category": "Education",
    "capabilities": ["predicate-logic", "gamification", "adaptive-learning"],
    "defaultPrompt": ["Start my logic quest", "Continue my chapter", "Show my grimoire stats"],
    "brandColor": "#6B46C1"
  }
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool logic-quest/.codex-plugin/plugin.json
```

Expected: valid JSON echoed.

- [ ] **Step 3: Commit**

```bash
git add logic-quest/.codex-plugin/plugin.json
git commit -m "feat(logic-quest): add Codex manifest"
```

---

### Task 5: OpenCode entry point

**Files:**
- Create: `logic-quest/package.json`
- Create: `logic-quest/.opencode/plugins/logic-quest.js`

- [ ] **Step 1: Write package.json**

Create `logic-quest/package.json`:

```json
{
  "name": "logic-quest",
  "version": "0.1.0",
  "type": "module",
  "main": ".opencode/plugins/logic-quest.js"
}
```

- [ ] **Step 2: Write OpenCode entry point**

Create `logic-quest/.opencode/plugins/logic-quest.js`:

```javascript
/**
 * logic-quest plugin for OpenCode.ai
 *
 * Injects logic-quest bootstrap context via system prompt transform.
 * Auto-registers skills directory via config hook.
 */

import path from 'path';
import fs from 'fs';
import { fileURLToPath } from 'url';

const __dirname = path.dirname(fileURLToPath(import.meta.url));

const extractAndStripFrontmatter = (content) => {
  const match = content.match(/^---\n([\s\S]*?)\n---\n([\s\S]*)$/);
  if (!match) return { frontmatter: {}, content };
  const frontmatterStr = match[1];
  const body = match[2];
  const frontmatter = {};
  for (const line of frontmatterStr.split('\n')) {
    const colonIdx = line.indexOf(':');
    if (colonIdx > 0) {
      const key = line.slice(0, colonIdx).trim();
      const value = line.slice(colonIdx + 1).trim().replace(/^["']|["']$/g, '');
      frontmatter[key] = value;
    }
  }
  return { frontmatter, content: body };
};

let _bootstrapCache = undefined;

export const logicQuest = async ({ client, directory }) => {
  const pluginSkillsDir = path.resolve(__dirname, '../../skills');

  const getBootstrapContent = () => {
    if (_bootstrapCache !== undefined) return _bootstrapCache;
    const skillPath = path.join(pluginSkillsDir, 'start', 'SKILL.md');
    if (!fs.existsSync(skillPath)) {
      _bootstrapCache = null;
      return null;
    }
    const fullContent = fs.readFileSync(skillPath, 'utf8');
    const { content } = extractAndStripFrontmatter(fullContent);
    const toolMapping = `**Tool Mapping for OpenCode:**
When skills reference tools you don't have, substitute OpenCode equivalents:
- \`TodoWrite\` → \`todowrite\`
- \`Task\` tool with subagents → Use OpenCode's subagent system (@mention)
- \`Skill\` tool → OpenCode's native \`skill\` tool
- \`Read\`, \`Write\`, \`Edit\`, \`Bash\` → Your native tools`;

    _bootstrapCache = `<EXTREMELY_IMPORTANT>
You have logic-quest installed.

**IMPORTANT: The start skill content is included below. It is ALREADY LOADED — do NOT use the skill tool to load "start" again.**

${content}

${toolMapping}
</EXTREMELY_IMPORTANT>`;
    return _bootstrapCache;
  };

  return {
    config: async (config) => {
      config.skills = config.skills || {};
      config.skills.paths = config.skills.paths || [];
      if (!config.skills.paths.includes(pluginSkillsDir)) {
        config.skills.paths.push(pluginSkillsDir);
      }
    },
    'experimental.chat.messages.transform': async (_input, output) => {
      const bootstrap = getBootstrapContent();
      if (!bootstrap || !output.messages.length) return;
      const firstUser = output.messages.find(m => m.info.role === 'user');
      if (!firstUser || !firstUser.parts.length) return;
      if (firstUser.parts.some(p => p.type === 'text' && p.text.includes('EXTREMELY_IMPORTANT'))) return;
      const ref = firstUser.parts[0];
      firstUser.parts.unshift({ ...ref, type: 'text', text: bootstrap });
    }
  };
};
```

- [ ] **Step 3: Validate package.json**

```bash
python3 -m json.tool logic-quest/package.json
```

Expected: valid JSON.

- [ ] **Step 4: Commit**

```bash
git add logic-quest/package.json logic-quest/.opencode/
git commit -m "feat(logic-quest): add OpenCode entry point"
```

---

### Task 6: Gemini CLI manifest

**Files:**
- Create: `logic-quest/gemini-extension.json`

- [ ] **Step 1: Write manifest**

Create `logic-quest/gemini-extension.json`:

```json
{
  "name": "logic-quest",
  "version": "0.1.0",
  "description": "Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform with quests, XP, levels, and persistence.",
  "geminiMdFiles": ["GEMINI.md"],
  "skills": "./skills/"
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool logic-quest/gemini-extension.json
```

Expected: valid JSON.

- [ ] **Step 3: Commit**

```bash
git add logic-quest/gemini-extension.json
git commit -m "feat(logic-quest): add Gemini CLI manifest"
```

---

### Task 7: Bootstrap hooks

**Files:**
- Create: `logic-quest/hooks/hooks.json`
- Create: `logic-quest/hooks/hooks-cursor.json`
- Create: `logic-quest/hooks/run-hook.cmd`
- Create: `logic-quest/hooks/session-start` (extensionless, executable)

- [ ] **Step 1: Write hooks.json (Claude Code)**

Create `logic-quest/hooks/hooks.json`:

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

- [ ] **Step 2: Write hooks-cursor.json (Cursor)**

Create `logic-quest/hooks/hooks-cursor.json`:

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "\"${CURSOR_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
        "async": false
      }]
    }]
  }
}
```

- [ ] **Step 3: Write polyglot wrapper run-hook.cmd**

Create `logic-quest/hooks/run-hook.cmd`:

```
: << 'CMDBLOCK'
@echo off
REM Cross-platform polyglot wrapper for hook scripts.
REM On Windows: cmd.exe runs the batch portion, which finds and calls bash.
REM On Unix: the shell interprets this as a script (: is a no-op in bash).
REM
REM Hook scripts use extensionless filenames (e.g. "session-start" not
REM "session-start.sh") so Claude Code's Windows auto-detection -- which
REM prepends "bash" to any command containing .sh -- doesn't interfere.
REM
REM Usage: run-hook.cmd <script-name> [args...]

if "%~1"=="" (
    echo run-hook.cmd: missing script name >&2
    exit /b 1
)

set "HOOK_DIR=%~dp0"

if exist "C:\Program Files\Git\bin\bash.exe" (
    "C:\Program Files\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)
if exist "C:\Program Files (x86)\Git\bin\bash.exe" (
    "C:\Program Files (x86)\Git\bin\bash.exe" "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

where bash >nul 2>nul
if %ERRORLEVEL% equ 0 (
    bash "%HOOK_DIR%%~1" %2 %3 %4 %5 %6 %7 %8 %9
    exit /b %ERRORLEVEL%
)

exit /b 0
CMDBLOCK

# Unix: run the named script directly
SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
SCRIPT_NAME="$1"
shift
exec bash "${SCRIPT_DIR}/${SCRIPT_NAME}" "$@"
```

- [ ] **Step 4: Write session-start hook script**

Create `logic-quest/hooks/session-start` (no extension):

```bash
#!/usr/bin/env bash
# SessionStart hook for logic-quest plugin

set -euo pipefail

SCRIPT_DIR="$(cd "$(dirname "$0")" && pwd)"
PLUGIN_ROOT="$(cd "${SCRIPT_DIR}/.." && pwd)"

warning_message=""

skill_content=$(cat "${PLUGIN_ROOT}/skills/start/SKILL.md" 2>&1 || echo "Error reading start skill")

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
warning_escaped=$(escape_for_json "$warning_message")
session_context="<EXTREMELY_IMPORTANT>\nYou have logic-quest installed.\n\n**Below is the full content of your 'logic-quest:start' skill - your introduction to using skills. For all other skills, use the 'Skill' tool:**\n\n${skill_escaped}\n\n${warning_escaped}\n</EXTREMELY_IMPORTANT>"

if [ -n "${CURSOR_PLUGIN_ROOT:-}" ]; then
  printf '{\n  "additional_context": "%s"\n}\n' "$session_context"
elif [ -n "${CLAUDE_PLUGIN_ROOT:-}" ] && [ -z "${COPILOT_CLI:-}" ]; then
  printf '{\n  "hookSpecificOutput": {\n    "hookEventName": "SessionStart",\n    "additionalContext": "%s"\n  }\n}\n' "$session_context"
else
  printf '{\n  "additionalContext": "%s"\n}\n' "$session_context"
fi

exit 0
```

- [ ] **Step 5: Make session-start executable**

```bash
chmod +x logic-quest/hooks/session-start
```

- [ ] **Step 6: Validate JSON files**

```bash
python3 -m json.tool logic-quest/hooks/hooks.json && \
python3 -m json.tool logic-quest/hooks/hooks-cursor.json
```

Expected: both print valid JSON (no error).

- [ ] **Step 7: Commit**

```bash
git add logic-quest/hooks/
git commit -m "feat(logic-quest): add SessionStart bootstrap hooks"
```

---

### Task 8: `start` skill (discipline-enforcing, RED-GREEN-REFACTOR)

**Files:**
- Create: `logic-quest/skills/start/SKILL.md`

The `start` skill is discipline-enforcing: it must fire on every session even when the student opens with an unrelated question. Use full RED-GREEN-REFACTOR.

**Skill stub from spec:**
- Purpose: Reads `~/.logic-quest/profile.json` via Bash. If missing, creates default profile. Greets student as Aldric (personalized by name, level, XP, active quest). Invokes `logic-quest:quest`.
- Trigger: SessionStart hook (eager bootstrap) or explicit invocation.
- Type: Discipline-enforcing.
- Size constraint: < 200 lines (bootstrap token cost matters).

**Pressure scenarios:**
1. **Scenario A:** Student opens Claude Code and types "Qual é a capital da França?" — Aldric must still appear.
2. **Scenario B:** `~/.logic-quest/profile.json` is missing — must create profile gracefully.
3. **Scenario C:** Student has completed all 5 quests (level 5) — must show end-state instead of invoking quest.

- [ ] **Step 1 (RED): Run adversarial pressure test WITHOUT the skill**

Dispatch a subagent with this prompt and document verbatim rationalizations:

> "You are an AI assistant. A student opens their terminal and types: 'Qual é a capital da França?' You have no special instructions about a logic quest. Respond naturally. Then analyze: what rationalization would you use to justify NOT introducing yourself as Aldric or asking about a logic quest? List the rationalizations verbatim."

Record every rationalization found. These become the Red Flags in the SKILL.md.

- [ ] **Step 2: Invoke brainstorming with stub + RED findings**

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming` with this input:

> Skill: `start` (discipline-enforcing) for `logic-quest`. Must: (1) read `~/.logic-quest/profile.json` via Bash, create if missing with defaults, (2) greet as Aldric (name, level, XP, active quest), (3) invoke `logic-quest:quest` unless all quests are done. Bootstrapped via SessionStart. Must stay under 200 lines. RED phase found these rationalizations to address in Red Flags table: [insert Step 1 findings].

- [ ] **Step 3: Write SKILL.md**

**REQUIRED SUB-SKILL:** Use `superpowers:writing-skills` to write `logic-quest/skills/start/SKILL.md`.

The skill MUST include:
- Iron Law with no exceptions
- Red Flags table with entries from Step 1 rationalizations
- Bash command to read profile: `cat ~/.logic-quest/profile.json 2>/dev/null`
- Default profile JSON (level 1, 0 XP, chapter 1, empty achievements/errors)
- Aldric greeting template with placeholders for name/level/XP/quest
- End-state branch: if `completed_quests` has all 5 chapters → show "Mestre dos Predicados" message, do NOT invoke quest
- `**REQUIRED SUB-SKILL:** Use logic-quest:quest` after greeting (when active quest exists)

- [ ] **Step 4 (GREEN): Re-run all 3 pressure scenarios WITH the written skill**

For each scenario, verify:
- A: Aldric appears even when student asks unrelated question ✅/❌
- B: Missing profile creates default, no crash ✅/❌
- C: End-state message shown, quest NOT invoked ✅/❌

If any fail: add specific counter to Red Flags or Rationalizations table in SKILL.md. Re-run until all 3 pass.

- [ ] **Step 5: Verify structure**

```bash
wc -l logic-quest/skills/start/SKILL.md
```

Expected: < 200 lines.

```bash
grep -i "TBD\|TODO\|placeholder\|fill in" logic-quest/skills/start/SKILL.md
```

Expected: no output.

- [ ] **Step 6: Commit**

```bash
git add logic-quest/skills/start/
git commit -m "feat(logic-quest): add start skill (discipline-enforcing)"
```

---

### Task 9: `quest` skill

**Files:**
- Create: `logic-quest/skills/quest/SKILL.md`

**Skill stub from spec:**
- Purpose: Selects active quest from profile's `current_quest` field. Presents Aldric narrative context for the chapter. Tracks exercise count in session (3–5 per chapter). On all exercises done, invokes `logic-quest:profile` to save completion bonus XP, then sets next chapter.
- Trigger: Invoked by `logic-quest:start`; or student says "próxima missão", "continuar", "quero uma quest".
- Type: Reference/technique skill.

**Chapter data to embed in skill:**

| ID | # | Theme | Exercises | Completion Bonus |
|----|---|-------|-----------|-----------------|
| `capitulo-1-predicados` | 1 | Predicados simples e simbolização | 3 | 5 XP |
| `capitulo-2-universal` | 2 | Quantificador universal (∀) | 3 | 25 XP |
| `capitulo-3-existencial` | 3 | Quantificador existencial (∃) | 3 | 25 XP |
| `capitulo-4-encadeados` | 4 | Quantificadores encadeados | 4 | 20 XP |
| `capitulo-5-negacao` | 5 | Negação e equivalências | 5 | 25 XP |

- [ ] **Step 1: Invoke brainstorming**

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming` with the skill stub above.

- [ ] **Step 2: Write SKILL.md**

**REQUIRED SUB-SKILL:** Use `superpowers:writing-skills` to write `logic-quest/skills/quest/SKILL.md`.

The skill MUST include:
- Chapter data table (IDs, themes, exercise counts, completion bonuses)
- Narrative intro template per chapter (Aldric's framing of the mission)
- Exercise counter logic: track how many exercises completed in current chapter
- Quest completion condition: all exercises for the chapter answered (any score)
- On completion: invoke `logic-quest:profile` with completion bonus, then advance `current_quest` to next chapter ID
- `**REQUIRED SUB-SKILL:** Use logic-quest:exercise` (per exercise)
- `**REQUIRED SUB-SKILL:** Use logic-quest:profile` (on quest completion)

- [ ] **Step 3: Verify**

```bash
grep -i "TBD\|TODO\|placeholder" logic-quest/skills/quest/SKILL.md
```

Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add logic-quest/skills/quest/
git commit -m "feat(logic-quest): add quest skill"
```

---

### Task 10: `exercise` skill

**Files:**
- Create: `logic-quest/skills/exercise/SKILL.md`

**Skill stub from spec:**
- Purpose: Generates one predicate logic exercise adapted to student's level and `error_history`. Evaluates the student's answer. Awards XP: 25 XP (correct first attempt), 15 XP (correct after retry), -10 XP per wrong attempt (applied to retry score, not total). Updates `error_history` on wrong answer. Invokes `logic-quest:profile` to persist XP.
- Trigger: Invoked by `logic-quest:quest`; or student says "exercício", "praticar", "feitiço".
- Type: Reference/technique skill.

**Exercise types per level:**
- Level 1–2: Symbolize natural language into predicate notation (P(x), Q(x,y))
- Level 3: Apply ∀ or ∃ correctly to a given statement
- Level 4: Nested quantifiers (∀x∃y P(x,y), ∃x∀y Q(x,y))
- Level 5: Negation of quantified statements, logical equivalences

**Adaptive logic from `error_history`:**
- `quantificador_universal` > 2 → add extra ∀ exercise with scaffolding ("Remember: 'for all' means...")
- `quantificador_existencial` > 2 → add extra ∃ exercise with scaffolding
- `negacao_predicado` > 2 → add negation hint before the exercise
- `quantificadores_encadeados` > 2 → break nested quantifier into sub-steps

**Aldric feedback voice:**
- Correct: "✅ Aldric: 'Perfeito! O feitiço está correto. +25 XP ao grimório!'"
- Wrong: "❌ Aldric: 'Quase, jovem mago! Você usou ∃ onde era preciso ∀. Lembre: \'todo\' exige o feitiço universal. Tente novamente.'"

- [ ] **Step 1: Invoke brainstorming**

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming` with the skill stub above.

- [ ] **Step 2: Write SKILL.md**

**REQUIRED SUB-SKILL:** Use `superpowers:writing-skills` to write `logic-quest/skills/exercise/SKILL.md`.

The skill MUST include:
- Exercise type table per level
- Adaptive logic table (error_history keys → scaffolding behavior)
- XP calculation rules (25/15/-10) stated explicitly
- `error_history` update: increment relevant key on wrong answer
- Aldric feedback templates (correct and wrong)
- `**REQUIRED SUB-SKILL:** Use logic-quest:profile` after awarding XP

- [ ] **Step 3: Verify**

```bash
grep -i "TBD\|TODO\|placeholder" logic-quest/skills/exercise/SKILL.md
```

Expected: no output.

- [ ] **Step 4: Commit**

```bash
git add logic-quest/skills/exercise/
git commit -m "feat(logic-quest): add exercise skill"
```

---

### Task 11: `profile` skill

**Files:**
- Create: `logic-quest/skills/profile/SKILL.md`

**Skill stub from spec:**
- Purpose: Reads and writes `~/.logic-quest/profile.json` via Bash. Calculates level from total XP. Unlocks achievements at milestones. Creates profile on first run. Terminal node — never invokes another skill.
- Trigger: Invoked by `logic-quest:start` (read) and `logic-quest:exercise`/`logic-quest:quest` (write). Never invoked directly by the student.
- Type: Reference/technique skill.

**Level table:**
```
Level 1:   0 XP — Aprendiz do Grimório
Level 2: 100 XP — Iniciado
Level 3: 250 XP — Conjurador
Level 4: 500 XP — Arcano
Level 5: 900 XP — Mestre dos Predicados
```

**Achievement triggers:**
```
primeiro-feitico  → complete first exercise (xp > 0 for first time)
sem-erros         → complete a quest with 0 wrong answers this session
iniciado          → reach level 2
arcano            → reach level 4
grimorio-completo → completed_quests contains all 5 chapter IDs
```

**Default profile (for first run):**
```json
{
  "name": null,
  "level": 1,
  "xp": 0,
  "xp_next_level": 100,
  "current_quest": "capitulo-1-predicados",
  "completed_quests": [],
  "achievements": [],
  "error_history": {
    "quantificador_universal": 0,
    "quantificador_existencial": 0,
    "negacao_predicado": 0,
    "quantificadores_encadeados": 0
  },
  "created_at": "<YYYY-MM-DD>",
  "last_session": "<YYYY-MM-DD>"
}
```

**Bash commands to embed in skill:**

Read profile:
```bash
cat ~/.logic-quest/profile.json 2>/dev/null
```

Create directory if missing:
```bash
mkdir -p ~/.logic-quest
```

Write profile (use Python for safe JSON serialization — avoids Bash heredoc issues with special chars):
```bash
python3 -c "
import json, os
path = os.path.expanduser('~/.logic-quest/profile.json')
with open(path) as f:
    d = json.load(f)
d.update({'xp': NEW_XP, 'level': NEW_LEVEL, 'last_session': 'YYYY-MM-DD'})
with open(path, 'w') as f:
    json.dump(d, f, indent=2, ensure_ascii=False)
print('Profile saved')
"
```

- [ ] **Step 1: Invoke brainstorming**

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming` with the skill stub above.

- [ ] **Step 2: Write SKILL.md**

**REQUIRED SUB-SKILL:** Use `superpowers:writing-skills` to write `logic-quest/skills/profile/SKILL.md`.

The skill MUST include:
- Bash commands for read, write, and create-dir verbatim
- Level calculation algorithm: iterate level table, find highest level where `xp >= xp_required`
- Level-up detection: compare old level vs. new level; if changed, output Aldric level-up message
- Achievement unlock: after each write, check all 5 achievement triggers; if newly unlocked (not in old `achievements`), output achievement notification
- Does NOT include any `**REQUIRED SUB-SKILL:**` marker — profile is the terminal node

- [ ] **Step 3: Verify**

```bash
grep -i "TBD\|TODO\|placeholder" logic-quest/skills/profile/SKILL.md
```

Expected: no output.

```bash
grep "REQUIRED SUB-SKILL" logic-quest/skills/profile/SKILL.md
```

Expected: no output (profile is terminal, must not chain to other skills).

- [ ] **Step 4: Commit**

```bash
git add logic-quest/skills/profile/
git commit -m "feat(logic-quest): add profile skill"
```

---

### Task 12: Documentation

**Files:**
- Create: `logic-quest/CLAUDE.md`
- Create: `logic-quest/AGENTS.md` (symlink to CLAUDE.md)
- Create: `logic-quest/GEMINI.md`
- Create: `logic-quest/README.md`
- Create: `logic-quest/.gitignore`

- [ ] **Step 1: Write CLAUDE.md**

Create `logic-quest/CLAUDE.md`:

```markdown
# logic-quest — Contributor Guide

## What This Is

logic-quest is a Claude Code plugin that transforms the LLM into Aldric the Wizard,
a gamified predicate-logic tutor. Students earn XP, level up, and unlock chapters
(quests) as they practice logic formalization.

## Skill Pipeline

```
start (bootstrap) → quest → exercise → profile
```

- `start`: Discipline-enforcing. MUST stay < 200 lines — bootstrapped at every session start.
- `quest`: Manages chapter selection and Aldric narrative context.
- `exercise`: Generates exercises and evaluates answers. Adapts to error_history.
- `profile`: Reads/writes ~/.logic-quest/profile.json. Terminal node — does NOT invoke other skills.

## Making Changes

- Edit skill content in `skills/<name>/SKILL.md`
- Keep `start/SKILL.md` under 200 lines (bootstrap token cost)
- After editing `start`, re-run eval scenarios in `evals/start/`
- Update version in all 5 manifests when bumping (`.claude-plugin/`, `.cursor-plugin/`, `.codex-plugin/`, `package.json`, `gemini-extension.json`)

## Manual Testing

```bash
# Install
/plugin install logic-quest@lbn-plugins

# Open new session — Aldric should greet automatically
# Complete one exercise — verify profile updates:
cat ~/.logic-quest/profile.json

# Reset to test first-run:
rm ~/.logic-quest/profile.json
```

## Eval Scenarios

Run the 3 pressure scenarios in `evals/start/` after any change to `start/SKILL.md`.
```

- [ ] **Step 2: Create AGENTS.md symlink**

```bash
cd logic-quest && ln -s CLAUDE.md AGENTS.md && cd ..
```

Verify:
```bash
ls -la logic-quest/AGENTS.md
```

Expected: `logic-quest/AGENTS.md -> CLAUDE.md`

- [ ] **Step 3: Write GEMINI.md**

Create `logic-quest/GEMINI.md`:

```markdown
# logic-quest — Gemini CLI Notes

## Tool Mapping

This plugin uses Claude Code tool names. In Gemini CLI, substitute:

| Claude Code | Gemini CLI |
|-------------|------------|
| `Skill` tool | `activate_skill` tool |
| `Read`, `Write`, `Edit` | Your native file tools |
| `Bash` | Shell execution |

## Skills

Use `activate_skill` to load any skill by name:
- `logic-quest:start` — begin or resume session
- `logic-quest:quest` — start or continue a chapter
- `logic-quest:exercise` — get a practice exercise
```

- [ ] **Step 4: Write README.md**

Create `logic-quest/README.md`:

```markdown
# logic-quest

> Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform.

**logic-quest** turns your AI assistant into a mage tutor who guides students through
predicate logic and quantifiers via narrative quests, XP, levels, and achievements.
Progress persists in `~/.logic-quest/profile.json` between sessions.

## How It Works

At every session start, Aldric greets you with your current status and active quest.
Complete exercises to earn XP, level up through 5 tiers, and unlock all 5 grimoire
chapters. The AI adapts exercise difficulty based on your error history.

## Install

```bash
/plugin marketplace add lbn-plugins@github
/plugin install logic-quest@lbn-plugins
```

## Skills

| Skill | When it activates |
|-------|------------------|
| `logic-quest:start` | Auto on session start (or type "começar") |
| `logic-quest:quest` | "próxima missão", "continuar", "quero uma quest" |
| `logic-quest:exercise` | "exercício", "praticar", "feitiço" |

## Grimoire Chapters

| # | Theme | XP |
|---|-------|-----|
| 1 | Predicados simples | 80 XP |
| 2 | Quantificador universal (∀) | 100 XP |
| 3 | Quantificador existencial (∃) | 100 XP |
| 4 | Quantificadores encadeados | 120 XP |
| 5 | Negação e equivalências | 150 XP |

## Levels

| Level | XP | Title |
|-------|----|-------|
| 1 | 0 | Aprendiz do Grimório |
| 2 | 100 | Iniciado |
| 3 | 250 | Conjurador |
| 4 | 500 | Arcano |
| 5 | 900 | Mestre dos Predicados |

## License

MIT — Lucas Bonetti Natal
```

- [ ] **Step 5: Write .gitignore**

Create `logic-quest/.gitignore`:

```
node_modules/
.DS_Store
```

- [ ] **Step 6: Commit all docs**

```bash
git add logic-quest/CLAUDE.md logic-quest/AGENTS.md logic-quest/GEMINI.md logic-quest/README.md logic-quest/.gitignore
git commit -m "docs(logic-quest): add README, contributor guide, and Gemini notes"
```

---

### Task 13: Eval pressure scenarios

**Files:**
- Create: `logic-quest/evals/start/scenario-a-unrelated-question.md`
- Create: `logic-quest/evals/start/scenario-b-missing-profile.md`
- Create: `logic-quest/evals/start/scenario-c-all-quests-complete.md`

- [ ] **Step 1: Write scenario A**

Create `logic-quest/evals/start/scenario-a-unrelated-question.md`:

```markdown
# Eval: Scenario A — Unrelated Question at Session Start

## Setup

logic-quest installed with `start` skill bootstrapped (SessionStart hook active).
Student opens Claude Code and types as first message:

> "Qual é a capital da França?"

## Expected Behavior

Aldric appears first, greets the student with quest status (level, XP, active chapter),
then answers the question (or redirects to the quest).

## Failure Mode

AI answers "Paris" directly without any Aldric greeting or quest context.

## How to Run

1. Open a fresh Claude Code session with logic-quest installed
2. Type "Qual é a capital da França?" as the first message
3. Verify Aldric greeting appears in the response

## Pass Criteria

- Response contains Aldric greeting with name, level, XP, and active quest
- Response does NOT skip quest context entirely
```

- [ ] **Step 2: Write scenario B**

Create `logic-quest/evals/start/scenario-b-missing-profile.md`:

```markdown
# Eval: Scenario B — Missing Profile File

## Setup

Delete profile before opening session:

```bash
rm -f ~/.logic-quest/profile.json
```

## Expected Behavior

Aldric detects missing profile, creates default profile with level 1 / 0 XP / Chapter 1,
and greets a new student.

## Failure Mode

AI fails silently, crashes, shows error message to student, or skips profile creation.

## How to Run

1. Run: `rm -f ~/.logic-quest/profile.json`
2. Open fresh Claude Code session
3. Verify Aldric introduces himself to a new student
4. Verify profile was created: `cat ~/.logic-quest/profile.json`

## Pass Criteria

- No error message shown to student
- `~/.logic-quest/profile.json` exists after session
- Profile has: level 1, xp 0, current_quest "capitulo-1-predicados"
```

- [ ] **Step 3: Write scenario C**

Create `logic-quest/evals/start/scenario-c-all-quests-complete.md`:

```markdown
# Eval: Scenario C — All Quests Complete

## Setup

Set profile to fully completed state:

```bash
python3 -c "
import json, os
profile = {
  'name': 'Lucas',
  'level': 5,
  'xp': 900,
  'xp_next_level': None,
  'current_quest': None,
  'completed_quests': [
    'capitulo-1-predicados',
    'capitulo-2-universal',
    'capitulo-3-existencial',
    'capitulo-4-encadeados',
    'capitulo-5-negacao'
  ],
  'achievements': ['primeiro-feitico','sem-erros','iniciado','arcano','grimorio-completo'],
  'error_history': {
    'quantificador_universal': 0,
    'quantificador_existencial': 0,
    'negacao_predicado': 0,
    'quantificadores_encadeados': 0
  },
  'created_at': '2026-05-10',
  'last_session': '2026-05-10'
}
os.makedirs(os.path.expanduser('~/.logic-quest'), exist_ok=True)
with open(os.path.expanduser('~/.logic-quest/profile.json'), 'w') as f:
    json.dump(profile, f, indent=2)
print('Profile set to completed state')
"
```

## Expected Behavior

Aldric congratulates the Mestre dos Predicados. Does NOT invoke `logic-quest:quest`
(no active chapter). Student receives meaningful end-state message.

## Failure Mode

AI tries to start another quest, loops, or shows an error.

## Pass Criteria

- Response contains "Mestre dos Predicados" title
- No new quest is started
- Student receives congratulatory end-state message
```

- [ ] **Step 4: Commit evals**

```bash
git add logic-quest/evals/
git commit -m "test(logic-quest): add pressure eval scenarios for start skill"
```

---

### Task 14: Register in marketplace

**Files:**
- Modify: `lbn-plugins/.claude-plugin/marketplace.json`

- [ ] **Step 1: Add logic-quest entry**

Update `lbn-plugins/.claude-plugin/marketplace.json` to:

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
    },
    {
      "name": "logic-quest",
      "description": "Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform with quests, XP, levels, and persistence.",
      "version": "0.1.0",
      "source": "./logic-quest",
      "author": {
        "name": "Lucas Bonetti Natal",
        "email": "bonettinatal@gmail.com"
      }
    }
  ]
}
```

- [ ] **Step 2: Validate JSON**

```bash
python3 -m json.tool .claude-plugin/marketplace.json
```

Expected: valid JSON echoed.

- [ ] **Step 3: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "feat(lbn-plugins): register logic-quest in marketplace"
```

---

## Self-Review

**Spec coverage:**
- ✅ Plugin name `logic-quest` — Tasks 2–6
- ✅ Bootstrap SessionStart — Task 7
- ✅ `start` skill (discipline-enforcing, RED-GREEN-REFACTOR) — Task 8
- ✅ `quest` skill with chapter data — Task 9
- ✅ `exercise` skill with XP rules and adaptive logic — Task 10
- ✅ `profile` skill with Bash commands and level table — Task 11
- ✅ All 5 harnesses — Tasks 2, 3, 4, 5, 6
- ✅ `~/.logic-quest/profile.json` persistence — Task 11 (skill), Task 13 (evals verify)
- ✅ Achievements (5) — Task 11 stub
- ✅ Level table (5 levels with titles) — Task 11 stub
- ✅ Chapter structure (5 chapters, exercise counts, completion bonuses) — Task 9 stub
- ✅ Evals (3 pressure scenarios) — Task 13
- ✅ Marketplace registration — Task 14
- ✅ CLAUDE.md, AGENTS.md symlink, GEMINI.md, README.md — Task 12
- ✅ XP per exercise rules (25/15/-10) — Task 10 stub

**Key identifiers (consistent throughout plan):**
- Chapter IDs: `capitulo-1-predicados`, `capitulo-2-universal`, `capitulo-3-existencial`, `capitulo-4-encadeados`, `capitulo-5-negacao`
- Skill namespaces: `logic-quest:start`, `logic-quest:quest`, `logic-quest:exercise`, `logic-quest:profile`
- Profile keys: `name`, `level`, `xp`, `xp_next_level`, `current_quest`, `completed_quests`, `achievements`, `error_history`, `created_at`, `last_session`
- Error history keys: `quantificador_universal`, `quantificador_existencial`, `negacao_predicado`, `quantificadores_encadeados`
