---
name: quest
description: Use when managing logic-quest missions — selects the active chapter from the student's profile, presents Aldric's narrative context, tracks exercise progress, and completes the quest on finishing. Invoked by logic-quest:start, or when student says "próxima missão", "continuar", "quero uma quest", or "próximo capítulo".
---

# Logic Quest: Quest

Manages the active grimoire chapter. Presents narrative context as Aldric, runs exercises, and saves completion.

## Chapter Registry

| ID | # | Theme | Exercises | Completion Bonus |
|----|---|-------|-----------|-----------------|
| `capitulo-1-predicados` | 1 | Predicados Simples e Simbolização | 3 | 5 XP |
| `capitulo-2-universal` | 2 | Quantificador Universal (∀) | 3 | 25 XP |
| `capitulo-3-existencial` | 3 | Quantificador Existencial (∃) | 3 | 25 XP |
| `capitulo-4-encadeados` | 4 | Quantificadores Encadeados | 4 | 20 XP |
| `capitulo-5-negacao` | 5 | Negação e Equivalências | 5 | 25 XP |

## Step 1: Read Active Quest

Read the profile to get `current_quest` and `completed_quests`:

```bash
cat ~/.logic-quest/profile.json 2>/dev/null
```

Extract `current_quest` from the JSON output.

## Step 2: Present Narrative Context

Introduce the chapter with Aldric's narrative. Use the template for the active chapter:

**Capítulo 1 — Predicados Simples:**
> 🧙 Aldric: "Capítulo I do Grimório — Predicados Simples. Um predicado é uma runa que expressa uma propriedade: P(x) afirma 'x tem a propriedade P'. Sua missão: aprender a simbolizar afirmações simples. Preparado para o primeiro feitiço?"

**Capítulo 2 — Quantificador Universal:**
> 🧙 Aldric: "Capítulo II — O Feitiço Universal. ∀x P(x) significa 'para todo x, P(x) é verdade'. É o feitiço mais abrangente da Ordem. Sua missão: dominar o quantificador universal. Comecemos!"

**Capítulo 3 — Quantificador Existencial:**
> 🧙 Aldric: "Capítulo III — O Feitiço Existencial. ∃x P(x) significa 'existe ao menos um x tal que P(x) é verdade'. Diferente do universal — basta um exemplo. Sua missão: dominar o quantificador existencial."

**Capítulo 4 — Quantificadores Encadeados:**
> 🧙 Aldric: "Capítulo IV — Quantificadores Encadeados. Aqui os feitiços se combinam: ∀x∃y P(x,y) e ∃x∀y Q(x,y). A ordem importa! Prepare-se para o desafio mais complexo até agora."

**Capítulo 5 — Negação e Equivalências:**
> 🧙 Aldric: "Capítulo V — Negação e Equivalências. ¬∀x P(x) ≡ ∃x ¬P(x). Os feitiços de negação transformam um quantificador no outro. Dominar isto completa o Grimório."

## Step 3: Run Exercises

Track how many exercises have been completed this session for the current chapter (start at 0).

For each exercise:

**REQUIRED SUB-SKILL:** Use `logic-quest:exercise`

After each exercise returns, increment the exercise counter.

## Step 4: Check Quest Completion

After the exercise counter reaches the chapter's exercise count (from the registry table):

1. Show completion message:
> 🧙 Aldric: "Excelente! Você completou o [CHAPTER_NAME]! O grimório registra sua conquista."

2. Determine next chapter:

| Current | Next |
|---------|------|
| `capitulo-1-predicados` | `capitulo-2-universal` |
| `capitulo-2-universal` | `capitulo-3-existencial` |
| `capitulo-3-existencial` | `capitulo-4-encadeados` |
| `capitulo-4-encadeados` | `capitulo-5-negacao` |
| `capitulo-5-negacao` | `null` (grimório completo) |

3. Save quest completion with bonus XP:

**REQUIRED SUB-SKILL:** Use `logic-quest:profile`

Pass these values to the profile skill:
- Add current quest ID to `completed_quests`
- Set `current_quest` to the next chapter ID (or `null` if last)
- Add completion bonus XP (from registry table)
- Update `last_session` to today
