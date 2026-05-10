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
- Update version in all 5 manifests when bumping:
  - `.claude-plugin/plugin.json`
  - `.cursor-plugin/plugin.json`
  - `.codex-plugin/plugin.json`
  - `package.json`
  - `gemini-extension.json`

## Manual Testing

```bash
# Install
/plugin install logic-quest@lbn-plugins

# Open new session — Aldric should greet automatically

# Inspect profile after an exercise:
cat ~/.logic-quest/profile.json

# Reset to test first-run:
rm ~/.logic-quest/profile.json
```

## Eval Scenarios

Run the 3 pressure scenarios in `evals/start/` after any change to `start/SKILL.md`.
