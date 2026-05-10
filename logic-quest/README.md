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
