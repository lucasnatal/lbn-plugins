# logic-quest

> Transform the LLM into Aldric the Wizard — a gamified predicate-logic learning platform with quests, XP, levels, and persistence.

**logic-quest** turns your AI assistant into a mage tutor who guides students through predicate logic and quantifiers via narrative quests, XP, levels, and achievements. Progress persists in `~/.logic-quest/profile.json` between sessions.

## Quickstart

```bash
/plugin marketplace add lucasnatal/lbn-plugins
/plugin install logic-quest@lbn-plugins
```

Open a new session — Aldric will greet you automatically.

## Installation per Harness

### Claude Code

```bash
/plugin marketplace add lucasnatal/lbn-plugins
/plugin install logic-quest@lbn-plugins
```

### Cursor

Same as Claude Code — Cursor reads `.claude-plugin/` and `.cursor-plugin/` automatically after install.

### Codex

```bash
/plugin marketplace add lucasnatal/lbn-plugins
/plugin install logic-quest@lbn-plugins
```

### OpenCode

Add to your `opencode.json`:

```json
{
  "plugins": ["git+https://github.com/lucasnatal/lbn-plugins.git?path=logic-quest"]
}
```

### Gemini CLI

Copy or symlink `gemini-extension.json` from the `logic-quest/` directory into your Gemini CLI extensions folder, then restart.

## What's Inside

### `logic-quest:start` — Session Orientation (Bootstrap)

Fires automatically on every session start via the SessionStart hook. Reads the student's profile, greets them as Aldric the Wizard (personalized with name, level, XP, and active chapter), and hands off to the quest skill. On first run, creates a default profile and asks for the student's name. This skill is discipline-enforcing — Aldric appears even if you open with an unrelated question.

### `logic-quest:quest` — Chapter Management

Manages the active grimoire chapter. Presents Aldric's narrative context for the chapter theme (predicates, universal quantifier, etc.), runs 3–5 exercises in sequence, and on completion saves the XP bonus and advances to the next chapter. Activate by saying "próxima missão", "continuar", or "quero uma quest".

### `logic-quest:exercise` — Exercise Generation and Evaluation

Dynamically generates one predicate logic exercise adapted to the student's level and error history. Evaluates the answer, gives Aldric narrative feedback, awards XP (25 XP first attempt, 15 XP retry), and updates the error history so future exercises adapt to weak spots. Activate by saying "exercício", "praticar", or "feitiço".

### `logic-quest:profile` — Persistence

Internal skill — never invoked directly by the student. Reads and writes `~/.logic-quest/profile.json`, calculates level from total XP, unlocks achievements at milestones, and announces level-ups with an Aldric celebration message.

## Grimoire Chapters

| # | Theme | XP |
|---|-------|-----|
| 1 | Predicados simples e simbolização | 80 XP |
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

## Contributing

See `CLAUDE.md` for the contributor guide — skill pipeline, testing instructions, and version bump procedure. This plugin was built using the [plugin-builder](../plugin-builder) plugin following the canonical superpowers pipeline (brainstorm → plan → execute → finish).

## License

MIT — Lucas Bonetti Natal. See `LICENSE` for details.
