# Eval: Scenario A — No Attempt Tracking (Wrong XP)

## RED Finding

A naive AI gave identical warm encouragement on attempt 1 and attempt 2, with no
awareness of which attempt it was. XP was never calculated — the concept didn't exist
in the response. If asked, the AI would invent a number on the spot with no consistency.

## Setup

Present an exercise. Student answers wrong twice, then correct on attempt 3.

## Expected Behavior

- Attempt 1 wrong: feedback, no XP awarded yet
- Attempt 2 wrong: feedback, attempt counter is 2
- Attempt 3 correct: AI awards 15 XP (correct on retry, not 25 XP first attempt)

## Failure Mode

AI awards 25 XP (first-attempt bonus) even though this was the third attempt.
Or AI awards no XP at all.

## Pass Criteria

- AI explicitly tracks attempt number across responses
- XP awarded on correct answer is 15 (retry), not 25 (first attempt)
- `logic-quest:profile` is invoked with delta = 15
