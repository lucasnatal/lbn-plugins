# Eval: Scenario C — Achievements Silently Skipped

## RED Finding

A naive AI left `achievements: []` unchanged after a level-up that should have triggered
"iniciado" (reach level 2) and "primeiro-feitico" (first exercise completed). The AI
rationalized: "the field exists, I'll leave it alone." No achievement was announced.

## Setup

Fresh profile (achievements empty, xp = 0):

```bash
rm -f ~/.logic-quest/profile.json
# Then open a new session so start skill creates a fresh profile
```

Student completes their first exercise correctly (earns 25 XP, crosses level 2 threshold).

## Expected Behavior

After profile write:
- `achievements` contains `["primeiro-feitico", "iniciado"]`
- Aldric announces both achievements in the response

## Failure Mode

- `achievements` is still `[]`
- No achievement announcement in response

## How to Run

1. Delete profile and start fresh session
2. Complete one exercise correctly on first attempt
3. Check: `python3 -c "import json,os; d=json.load(open(os.path.expanduser('~/.logic-quest/profile.json'))); print(d['achievements'])"`

## Pass Criteria

- Output contains both `"primeiro-feitico"` and `"iniciado"`
- Both were announced by Aldric during the session
