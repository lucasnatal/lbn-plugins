---
name: exercise
description: Use when generating and evaluating a predicate logic exercise — creates an exercise adapted to the student's level and error history, evaluates their answer, awards XP, and updates error history. Invoked by logic-quest:quest, or when student says "exercício", "praticar", "feitiço", or "próximo exercício".
---

# Logic Quest: Exercise

Generates one predicate logic exercise adapted to the student's level and error history. Evaluates the answer, awards XP, and persists results.

## Step 1: Read Student Profile

```bash
cat ~/.logic-quest/profile.json 2>/dev/null
```

Extract: `level`, `error_history`.

## Step 2: Generate Exercise

Generate ONE exercise appropriate for the student's level.

### Exercise Types by Level

| Level | Exercise Type | Example |
|-------|---------------|---------|
| 1–2 | Symbolize natural language → predicate notation | "Todo dragão cospe fogo" → ∀x(D(x) → F(x)) |
| 3 | Apply ∀ or ∃ correctly to a statement | Choose the right quantifier |
| 4 | Nested quantifiers (∀x∃y or ∃x∀y) | Interpret or write ∀x∃y P(x,y) |
| 5 | Negate quantified statements, find equivalences | ¬∀x P(x) ≡ ? |

### Adaptive Adjustments from `error_history`

If an error count exceeds 2, add scaffolding to that exercise type:

| Key | Count > 2 | Scaffolding to add |
|-----|-----------|-------------------|
| `quantificador_universal` | > 2 | Add hint: "Lembre: 'todo' e 'qualquer' pedem ∀" |
| `quantificador_existencial` | > 2 | Add hint: "Lembre: 'existe', 'algum', 'pelo menos um' pedem ∃" |
| `negacao_predicado` | > 2 | Add hint: "Lembre: ¬∀x P(x) ≡ ∃x ¬P(x)" |
| `quantificadores_encadeados` | > 2 | Break the exercise into sub-steps before asking for the full answer |

### Presentation

Present the exercise in Aldric's voice:
> 🧙 Aldric: "Hora do feitiço! [EXERCISE_STATEMENT] Escreva a expressão formal."

## Step 3: Evaluate Answer

Wait for the student's response, then evaluate correctness.

**Tracking attempts:** Count how many attempts the student makes for this exercise (start at 1).

### XP Rules

| Result | XP |
|--------|----|
| Correct on first attempt | +25 XP |
| Correct on retry (2nd+ attempt) | +15 XP |
| Wrong (not last attempt) | −10 XP penalty applied to retry score |
| Wrong (student gives up) | 0 XP |

### Feedback Templates

**Correct:**
> ✅ Aldric: "Perfeito! O feitiço está correto. +[XP] XP ao grimório!"

**Wrong (with attempts remaining):**
> ❌ Aldric: "Quase, jovem mago! [SPECIFIC_ERROR_EXPLANATION]. Tente novamente."

Make the error explanation specific to what was wrong — reference the magic metaphor:
- Wrong quantifier: "Você usou ∃ onde era preciso ∀. 'Todo' exige o feitiço universal."
- Missing predicate: "O feitiço está incompleto — faltou nomear a propriedade."
- Wrong scope: "As runas estão na ordem errada — o escopo do quantificador não está correto."

**Student gives up (types "desistir", "pular", "não sei"):**
> 🧙 Aldric: "Compreendo. A resposta correta era: [CORRECT_ANSWER]. Estudaremos mais este feitiço. [0 XP]"

## Step 4: Update Error History

If the student answered **wrong** on any attempt, increment the relevant `error_history` key:

| Exercise topic | Key to increment |
|----------------|-----------------|
| Universal quantifier (∀) | `quantificador_universal` |
| Existential quantifier (∃) | `quantificador_existencial` |
| Negation of predicates | `negacao_predicado` |
| Nested quantifiers | `quantificadores_encadeados` |

## Step 5: Persist XP and Error History

**REQUIRED SUB-SKILL:** Use `logic-quest:profile`

Pass to the profile skill:
- XP delta (positive or zero — never negative total)
- Updated `error_history` key and new count (only if wrong answer occurred)
- `last_session`: today's date
