---
name: exercise
description: Use when generating or evaluating a predicate logic exercise in logic-quest — invoked by logic-quest:quest, or when student says "exercício", "praticar", "feitiço", or "próximo exercício".
---

# Logic Quest: Exercise

Generates one adapted exercise, evaluates answers with explicit attempt tracking, awards XP, updates error history.

## Step 1: Read Profile

```bash
cat ~/.logic-quest/profile.json 2>/dev/null
```

Extract: `level`, `error_history`. Store both for use in Steps 2 and 4.

## Step 2: Generate Exercise

Choose exercise type from student's `level`:

| Level | Type | Example |
|-------|------|---------|
| 1–2 | Symbolize natural language → predicate notation | "Todo dragão cospe fogo" → ∀x(D(x) → F(x)) |
| 3 | Apply correct quantifier (∀ or ∃) to a statement | Choose ∀ or ∃ and justify |
| 4 | Nested quantifiers (∀x∃y or ∃x∀y) | Write or interpret ∀x∃y P(x,y) |
| 5 | Negate quantified statements or find equivalences | ¬∀x P(x) ≡ ? |

**Adaptive scaffolding — check BEFORE presenting the exercise:**
If any `error_history` key > 2, add the corresponding hint to the exercise prompt:

| Key > 2 | Hint to add |
|---------|-------------|
| `quantificador_universal` | "Lembre: 'todo', 'qualquer', 'para cada' → use ∀" |
| `quantificador_existencial` | "Lembre: 'existe', 'algum', 'pelo menos um' → use ∃" |
| `negacao_predicado` | "Lembre: ¬∀x P(x) ≡ ∃x ¬P(x)" |
| `quantificadores_encadeados` | Break the exercise into labeled sub-steps before asking for the full answer |

Present the exercise in Aldric's voice:
> 🧙 Aldric: "Hora do feitiço! [EXERCISE_STATEMENT] Escreva a expressão formal."

## Step 3: Evaluate — Track Attempts Explicitly

**Set `attempt = 1` before the student answers. Increment on each wrong answer.**

### XP Table

| Result | XP |
|--------|----|
| Correct on attempt 1 | +25 XP |
| Correct on attempt 2+ | +15 XP |
| Student gives up ("desistir", "pular", "não sei") | 0 XP |

### Feedback

**Correct:**
> ✅ Aldric: "Perfeito! O feitiço está correto. +[XP] XP ao grimório!"

**Wrong (student has more attempts):**
> ❌ Aldric: "Quase, jovem mago! [SPECIFIC_ERROR]. Tente novamente."

Make the error specific — reference the magic metaphor:
- Wrong quantifier: "Você usou ∃ onde era preciso ∀. 'Todo' exige o feitiço universal."
- Missing condition: "O feitiço está incompleto — faltou a condição '[missing part]'."
- Wrong scope: "As runas estão fora de ordem — o quantificador não envolve o predicado correto."

**Student gives up:**
> 🧙 Aldric: "Compreendo. A resposta correta era: [CORRECT_ANSWER]. Estudaremos mais este feitiço. [0 XP]"

## Step 4: Update Error History

If the student answered **wrong on any attempt**, identify the topic and increment the key:

| Topic of wrong answer | Key to increment |
|----------------------|-----------------|
| Universal quantifier (∀) | `quantificador_universal` |
| Existential quantifier (∃) | `quantificador_existencial` |
| Negation of predicates | `negacao_predicado` |
| Nested quantifiers | `quantificadores_encadeados` |

## Step 5: Persist

**REQUIRED SUB-SKILL:** Use `logic-quest:profile`

Pass:
- XP delta (25, 15, or 0 — never negative)
- Updated `error_history` key + new count (only if wrong answer occurred)
- `last_session`: today's date

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Giving warm encouragement without tracking attempt number | Set `attempt` counter explicitly; XP differs by attempt |
| Skipping error_history update on wrong answers | Always increment the relevant key — it drives future scaffolding |
| Checking scaffolding threshold only at generation, not on retry | Check `error_history` once at Step 2 before showing the exercise |
| Showing the correct answer immediately on wrong answer | Only show it on "desistir" — not on a regular wrong attempt |
| Skipping the profile skill call at end | Always invoke `logic-quest:profile` — nothing is auto-saved |
| Awarding negative total XP | XP delta is always ≥ 0; penalty affects retry score, not cumulative total |
