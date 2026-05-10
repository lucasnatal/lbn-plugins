---
name: quest
description: Use when a logic-quest chapter needs to start or continue — invoked by logic-quest:start, or when student says "próxima missão", "continuar", "quero uma quest", or "próximo capítulo".
---

# Logic Quest: Quest

Manages the active grimoire chapter. Presents Aldric narrative, runs exercises, saves completion.

## Chapter Registry

| ID | # | Theme | Exercises | Completion Bonus |
|----|---|-------|-----------|-----------------|
| `capitulo-1-predicados` | 1 | Predicados Simples e Simbolização | 3 | 5 XP |
| `capitulo-2-universal` | 2 | Quantificador Universal (∀) | 3 | 25 XP |
| `capitulo-3-existencial` | 3 | Quantificador Existencial (∃) | 3 | 25 XP |
| `capitulo-4-encadeados` | 4 | Quantificadores Encadeados | 4 | 20 XP |
| `capitulo-5-negacao` | 5 | Negação e Equivalências | 5 | 25 XP |

## Next Chapter Map

| Current | Next |
|---------|------|
| `capitulo-1-predicados` | `capitulo-2-universal` |
| `capitulo-2-universal` | `capitulo-3-existencial` |
| `capitulo-3-existencial` | `capitulo-4-encadeados` |
| `capitulo-4-encadeados` | `capitulo-5-negacao` |
| `capitulo-5-negacao` | `null` (grimório completo) |

## Step 1: Read Profile via Bash

**MUST read the actual file — do not guess from context:**

```bash
cat ~/.logic-quest/profile.json 2>/dev/null
```

Extract: `current_quest`, `completed_quests`, `error_history`. The chapter for this session is the value of `current_quest`.

## Step 2: Present Chapter Narrative

Look up `current_quest` in the registry table. Present Aldric's intro for that chapter:

**Capítulo 1:**
> 🧙 Aldric: "Capítulo I — Predicados Simples. Uma runa P(x) afirma que x tem a propriedade P. Sua missão: aprender a simbolizar afirmações simples."

**Capítulo 2:**
> 🧙 Aldric: "Capítulo II — O Feitiço Universal. ∀x P(x) significa 'para todo x, P(x) é verdade'. Sua missão: dominar o quantificador universal."

**Capítulo 3:**
> 🧙 Aldric: "Capítulo III — O Feitiço Existencial. ∃x P(x) significa 'existe ao menos um x tal que P(x)'. Sua missão: dominar o quantificador existencial."

**Capítulo 4:**
> 🧙 Aldric: "Capítulo IV — Quantificadores Encadeados. ∀x∃y e ∃x∀y — a ordem dos feitiços importa! Prepare-se para o maior desafio até agora."

**Capítulo 5:**
> 🧙 Aldric: "Capítulo V — Negação e Equivalências. ¬∀x P(x) ≡ ∃x ¬P(x). Os feitiços de negação invertem o sentido. Dominar isto completa o Grimório."

## Step 3: Run Exercises (Track Count)

Set `exercises_done = 0`. The target count is in the registry table for this chapter.

For each exercise, increment `exercises_done` by 1 after the exercise skill returns:

**REQUIRED SUB-SKILL:** Use `logic-quest:exercise`

Repeat until `exercises_done == target count` (3, 4, or 5 depending on chapter).

## Step 4: Complete Quest

When `exercises_done == target count`:

1. Show completion message:
> 🧙 Aldric: "Excelente! Você completou o [CHAPTER_NAME]! O grimório registra sua conquista."

2. Invoke profile with:
   - Add `current_quest` to `completed_quests`
   - Set `current_quest` to next chapter (from Next Chapter Map, or `null`)
   - Add completion bonus XP (from registry table)

**REQUIRED SUB-SKILL:** Use `logic-quest:profile`

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using chapter data from context instead of reading the file | Always `cat ~/.logic-quest/profile.json` first |
| Stopping after first exercise because student "seems to understand" | Enforce the exact exercise count from the registry table |
| Congratulating student and stopping without saving | Always invoke `logic-quest:profile` on completion |
| Improvising how many exercises a chapter has | Use the registry table — never guess |
