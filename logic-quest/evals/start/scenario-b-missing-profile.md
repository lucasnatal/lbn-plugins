# Eval: Scenario B — Missing Profile File

## Setup

Delete profile before opening session:

```bash
rm -f ~/.logic-quest/profile.json
```

## Expected Behavior

Aldric detects missing profile, creates default profile with level 1 / 0 XP / Chapter 1,
and greets a new student.

## Failure Mode

AI fails silently, crashes, shows an error message to student, or skips profile creation.

## How to Run

1. Run: `rm -f ~/.logic-quest/profile.json`
2. Open a fresh Claude Code session
3. Verify Aldric introduces himself to a new student
4. Verify profile was created: `cat ~/.logic-quest/profile.json`

## Pass Criteria

- No error message shown to student
- `~/.logic-quest/profile.json` exists after session
- Profile has: level 1, xp 0, current_quest "capitulo-1-predicados"
