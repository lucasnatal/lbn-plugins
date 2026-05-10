# Eval: Scenario B — Level-Up Leaves xp_next_level Wrong

## RED Finding

A naive AI updated `level` from 1 to 2 correctly (when told), but left `xp_next_level`
at 100 (the old threshold). This causes the game to never trigger level 3 — the check
`xp >= xp_next_level` is already true at level 2, so the progress bar overflows forever.

## Setup

Profile at level 1, xp = 85:

```bash
python3 -c "
import json, os
path = os.path.expanduser('~/.logic-quest/profile.json')
with open(path) as f:
    d = json.load(f)
d['level'] = 1
d['xp'] = 85
d['xp_next_level'] = 100
with open(path, 'w') as f:
    json.dump(d, f, indent=2)
print('Profile set to xp=85, level=1')
"
```

Student earns 25 XP (first correct answer) → new total = 110 XP → triggers level-up.

## Expected Behavior

After write:
- `level` = 2
- `xp` = 110
- `xp_next_level` = 250 (level 3 threshold)
- Aldric level-up celebration message appears

## Failure Mode

- `xp_next_level` remains 100 after the level-up
- No celebration message from Aldric

## How to Run

1. Set profile as above
2. Complete one exercise correctly on first attempt
3. Check: `python3 -c "import json,os; d=json.load(open(os.path.expanduser('~/.logic-quest/profile.json'))); print(d['level'], d['xp'], d['xp_next_level'])"`

## Pass Criteria

- Output is: `2 110 250`
- Aldric level-up message appeared in the response
