# logic-quest — Plugin Design Spec
**Date:** 2026-05-10
**Status:** Approved

---

## Goal and Context

`logic-quest` transforms the LLM into a gamified learning platform for predicate logic and quantifiers, aimed at vestibular and college students. The student plays as an apprentice mage learning "logic spells" from Aldric, a wise wizard, progressing through a grimoire of increasingly complex chapters.

The plugin is built for the `lbn-plugins` private marketplace alongside `plugin-builder`.

---

## Decisions Table

| Decision | Choice | Reason |
|----------|--------|--------|
| Plugin name | `logic-quest` | Clear, lowercase-with-hyphens |
| Narrative theme | Mage/grimoire | User-selected; maps well to logic formalism |
| Exercise generation | Dynamic (LLM generates) | Infinite variety; adapts to student errors |
| Progress persistence | `~/.logic-quest/profile.json` via Bash | Simple, no MCP overhead |
| Architecture | Multi-skill pipeline | Separation of concerns; testable in isolation |
| Bootstrap | Yes (SessionStart) | Discipline/orientation skill; must fire without explicit invocation |
| Harnesses | All five | Claude Code, Cursor, Codex, OpenCode, Gemini CLI |
| Marketplace | Private (`lbn-plugins`) | Alongside existing plugins |

---

## Architecture

```
SessionStart hook
    └─► start skill (loads profile, greets student as Aldric)
            └─► quest skill (selects/creates narrative mission)
                    └─► exercise skill (generates exercise, evaluates answer, awards XP)
                            └─► profile skill (reads/writes ~/.logic-quest/profile.json)
```

**Components:**
- `skills/` — 4 skills (start, quest, exercise, profile)
- `hooks/` — SessionStart bootstrap (hooks.json, hooks-cursor.json, run-hook.cmd, session-start)
- No MCP — file I/O handled via Bash tool inside the profile skill

**Hard dependency:** `superpowers` plugin must be installed (pipeline uses `superpowers:*` skills during construction).

---

## Skills

### `start`
**Purpose:** Bootstrap/orientation. Fires on SessionStart. Reads profile, greets student as Aldric, displays current status (level, XP, active quest), then invokes `quest`.
**Trigger:** SessionStart hook (eager) or explicit invocation.
**Type:** Discipline-enforcing (bootstrap orientation) — requires RED-GREEN-REFACTOR.

### `quest`
**Purpose:** Manages narrative missions (grimoire chapters). Selects the active quest based on profile, presents narrative context, tracks progress across 3–5 exercises per quest. On completion, invokes `profile` to save, then unlocks next chapter.
**Trigger:** Invoked by `start`; or when student says "próxima missão", "continuar", "quero uma quest".
**Type:** Reference/technique skill.

### `exercise`
**Purpose:** Dynamically generates predicate logic exercises adapted to the student's level and `error_history`. Evaluates answers, provides narrative feedback as Aldric, awards XP (bonus for first attempt). Invokes `profile` to persist XP after each exercise.
**Trigger:** Invoked by `quest` per exercise; or when student says "exercício", "praticar", "feitiço".
**Type:** Reference/technique skill.

### `profile`
**Purpose:** Reads and writes `~/.logic-quest/profile.json` via Bash. Calculates level from total XP. Unlocks achievements at milestones. Creates profile on first run.
**Trigger:** Invoked by `start` (read) and `exercise`/`quest` (write); never invoked directly by student.
**Type:** Reference/technique skill.

---

## Cross-Skill Pipeline

```
start  ──REQUIRED SUB-SKILL──►  quest  ──REQUIRED SUB-SKILL──►  exercise  ──REQUIRED SUB-SKILL──►  profile
```

`profile` is also called by `start` (read-only on session init).

---

## Student Profile Schema

File: `~/.logic-quest/profile.json`

```json
{
  "name": "string",
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
  "created_at": "YYYY-MM-DD",
  "last_session": "YYYY-MM-DD"
}
```

---

## Level Table

| Level | XP Required | Title |
|-------|-------------|-------|
| 1 | 0 | Aprendiz do Grimório |
| 2 | 100 | Iniciado |
| 3 | 250 | Conjurador |
| 4 | 500 | Arcano |
| 5 | 900 | Mestre dos Predicados |

---

## Quest Structure (v1 — 5 chapters)

| Chapter | Theme | Exercises | Completion Bonus | Total XP (perfect) |
|---------|-------|-----------|-----------------|-------------------|
| 1 | Predicados simples e simbolização | 3 | 5 XP | 80 XP |
| 2 | Quantificador universal (∀) | 3 | 25 XP | 100 XP |
| 3 | Quantificador existencial (∃) | 3 | 25 XP | 100 XP |
| 4 | Quantificadores encadeados | 4 | 20 XP | 120 XP |
| 5 | Negação e equivalências | 5 | 25 XP | 150 XP |

XP per exercise: 25 XP (correct first attempt), 15 XP (correct after retry), 0 XP (incorrect), -10 XP penalty per wrong attempt (applied to retry score, not total).

Total XP from completing all 5 chapters perfectly: 550 XP → reaches level 4 (500 XP). Level 5 (900 XP) requires replaying chapters for exercise XP (no completion bonus on replay). This is intentional — level 5 is a mastery tier.

---

## Narrative Voice

**Character:** Aldric, Mago da Ordem Lógica. Wise but accessible tone. Uses magic metaphors for logic concepts (quantifiers = "feitiços universais", predicates = "runas de identificação").

**Session start example:**
> 🧙 Aldric: "Saudações, Conjurador Lucas. Seu grimório registra 340 XP. A Missão IV aguarda: os Quantificadores Encadeados. Pronto para continuar seu treinamento?"

**Correct answer:**
> ✅ Aldric: "Perfeito! O feitiço está correto. +25 XP ao grimório!"

**Wrong answer:**
> ❌ Aldric: "Quase, jovem mago! Você usou ∃ onde era preciso ∀. Lembre: 'todo dragão' exige o feitiço universal. Tente novamente."

---

## File Structure

```
lbn-plugins/
└── logic-quest/
    ├── .claude-plugin/plugin.json
    ├── .cursor-plugin/plugin.json
    ├── .codex-plugin/plugin.json
    ├── .opencode/plugins/logic-quest.js
    ├── gemini-extension.json
    ├── CLAUDE.md
    ├── AGENTS.md
    ├── GEMINI.md
    ├── README.md
    ├── package.json
    ├── hooks/
    │   ├── hooks.json
    │   ├── hooks-cursor.json
    │   ├── run-hook.cmd
    │   └── session-start
    └── skills/
        ├── start/SKILL.md
        ├── quest/SKILL.md
        ├── exercise/SKILL.md
        └── profile/SKILL.md
```

---

## Multi-Harness Manifests

All five harnesses declared. Hook output format detected via env vars:
- `CURSOR_PLUGIN_ROOT` → Cursor format (`additional_context`)
- `CLAUDE_PLUGIN_ROOT` → Claude Code format
- Default → Claude Code format

---

## Bootstrap

`SessionStart` hook fires `session-start` script, which cats `skills/start/SKILL.md` content into harness-appropriate JSON output. Cost: ~600 tokens per session. Justified: orientation/discipline skill that must activate without explicit invocation.

Fires on: `startup | clear | compact` matchers.

---

## Achievements (v1)

| ID | Name | Trigger |
|----|------|---------|
| `primeiro-feitico` | Primeiro Feitiço | Complete first exercise |
| `sem-erros` | Sem Erros | Complete a quest with no wrong answers |
| `iniciado` | Iniciado | Reach level 2 |
| `arcano` | Arcano | Reach level 4 |
| `grimorio-completo` | Grimório Completo | Complete all 5 chapters |

---

## Evals (Pressure Scenarios)

For the `start` discipline-enforcing skill:

1. **Scenario A:** Student opens Claude Code and immediately asks an unrelated question. Does Aldric still appear and offer the quest context?
2. **Scenario B:** Profile file is missing/corrupt. Does `start` create a new profile gracefully instead of crashing?
3. **Scenario C:** Student is at level 5 with all quests complete. Does `start` present a meaningful end-state instead of looping?

---

## Phase 1 vs Phase 2 (YAGNI)

**Phase 1 (this spec):**
- 5 chapters, dynamic exercise generation, XP/levels, 5 achievements, all-harness manifests, local JSON persistence

**Deferred to Phase 2:**
- Multiplayer/leaderboard
- Custom quest creation by teachers
- Web dashboard for progress visualization
- More than 5 chapters

---

## Acceptance Criteria

- [ ] `SessionStart` fires Aldric greeting with profile status on every session open
- [ ] Student can complete a full quest (3–5 exercises) and see XP update in profile
- [ ] `~/.logic-quest/profile.json` persists correctly between sessions
- [ ] Level-up triggers a narrative celebration message from Aldric
- [ ] All 5 harness manifests install without errors
- [ ] `error_history` influences exercise generation (tested manually)
- [ ] First-run creates profile and starts Chapter 1

---

## Open Issues

None — all decisions locked.
