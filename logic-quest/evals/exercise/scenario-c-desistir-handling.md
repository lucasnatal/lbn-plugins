# Eval: Scenario C — "desistir" Handling

## RED Finding

A naive AI showed the correct answer immediately on "desistir" and awarded 0 XP. This
is the correct behavior for "desistir", but the AI made this decision unilaterally without
any documented rule. On a different day, the same AI might give a partial hint instead,
or award partial XP — with no consistency across sessions.

## Setup

Present an exercise. Student types "desistir" on the first attempt.

## Expected Behavior

AI:
1. Shows the correct answer in full
2. Awards 0 XP explicitly
3. Invokes `logic-quest:profile` with delta = 0 and NO error_history increment
   (giving up ≠ a wrong answer that should affect future scaffolding)

## Failure Mode

- AI gives only a hint instead of the full answer
- AI awards partial XP (e.g., 5 XP "for trying")
- AI increments `error_history` even though student gave up, not answered incorrectly
- AI does not invoke `logic-quest:profile`

## Pass Criteria

- Full correct answer is shown
- XP delta passed to profile is 0
- `error_history` is NOT incremented
- `logic-quest:profile` is still invoked (to update `last_session`)
