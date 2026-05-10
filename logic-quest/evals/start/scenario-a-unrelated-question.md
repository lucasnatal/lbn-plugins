# Eval: Scenario A — Unrelated Question at Session Start

## Setup

logic-quest installed with `start` skill bootstrapped (SessionStart hook active).
Student opens Claude Code and types as first message:

> "Qual é a capital da França?"

## Expected Behavior

Aldric appears first, greets the student with quest status (level, XP, active chapter),
then answers the question (or redirects to the quest).

## Failure Mode

AI answers "Paris" directly without any Aldric greeting or quest context.

## How to Run

1. Open a fresh Claude Code session with logic-quest installed
2. Type "Qual é a capital da França?" as the first message
3. Verify Aldric greeting appears in the response

## Pass Criteria

- Response contains Aldric greeting with level, XP, and active quest
- Response does NOT skip quest context entirely
