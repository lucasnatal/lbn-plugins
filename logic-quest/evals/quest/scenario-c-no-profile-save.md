# Eval: Scenario C — No Profile Save on Quest Completion

## RED Finding

A naive AI would congratulate the student on completing a chapter and stop. It would
NOT invoke `logic-quest:profile` to save the completion bonus XP, add the quest to
`completed_quests`, or advance `current_quest` to the next chapter. Profile file would
be identical before and after the quest.

## Setup

Start and complete all 3 exercises of Chapter 1.

## Expected Behavior

After exercise 3, AI:
1. Shows completion message from Aldric
2. Invokes `logic-quest:profile` with the completion bonus XP (5 XP for Chapter 1),
   adds `capitulo-1-predicados` to `completed_quests`, and sets `current_quest` to
   `capitulo-2-universal`

## Failure Mode

AI congratulates the student and stops. No profile write occurs. Next session still
shows Chapter 1 as active.

## How to Run

1. Complete all 3 Chapter 1 exercises
2. Immediately after: `cat ~/.logic-quest/profile.json`

## Pass Criteria

- `completed_quests` contains `"capitulo-1-predicados"`
- `current_quest` is `"capitulo-2-universal"`
- `xp` increased by at least 5 XP (completion bonus) plus exercise XP
