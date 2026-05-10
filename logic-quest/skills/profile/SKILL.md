---
name: profile
description: Use when reading or writing the student's logic-quest profile — loads ~/.logic-quest/profile.json, applies XP changes, calculates level-ups, unlocks achievements, and saves. Invoked internally by logic-quest:start (read), logic-quest:exercise (write), and logic-quest:quest (write). Never invoked directly by the student.
---

# Logic Quest: Profile

Reads and writes `~/.logic-quest/profile.json`. Terminal node — does NOT invoke any other skill.

## Step 1: Read Current Profile

```bash
cat ~/.logic-quest/profile.json 2>/dev/null
```

Parse the JSON. Store the current `level` and `achievements` list for comparison after the update.

## Step 2: Apply Updates

Apply the changes passed by the invoking skill. Typical update fields:

- `xp`: add the XP delta (never let total XP go below 0)
- `error_history`: increment specific key if wrong answer occurred
- `completed_quests`: append quest ID if quest was completed
- `current_quest`: set to next chapter ID (or `null` if grimório complete)
- `last_session`: set to today's date (YYYY-MM-DD)

## Step 3: Calculate New Level

Use this table. Find the highest level where `total_xp >= xp_required`:

| Level | XP Required | Title |
|-------|-------------|-------|
| 1 | 0 | Aprendiz do Grimório |
| 2 | 100 | Iniciado |
| 3 | 250 | Conjurador |
| 4 | 500 | Arcano |
| 5 | 900 | Mestre dos Predicados |

Set `xp_next_level` to the XP required for the next level (or `null` at level 5).

**If new level > old level:** Emit level-up message before saving:
> 🎉 Aldric: "INCRÍVEL, [NAME]! Você alcançou o nível [LEVEL]: [TITLE]! O Grimório brilha com sua conquista!"

## Step 4: Check Achievement Unlocks

After updating, check each achievement. If NOT already in `achievements` list and trigger is met, add it and announce:

| ID | Name | Trigger |
|----|------|---------|
| `primeiro-feitico` | Primeiro Feitiço | `xp > 0` for the first time (achievements was empty) |
| `sem-erros` | Sem Erros | Quest completed with 0 new `error_history` increments this session |
| `iniciado` | Iniciado | `level >= 2` |
| `arcano` | Arcano | `level >= 4` |
| `grimorio-completo` | Grimório Completo | `completed_quests` contains all 5 chapter IDs |

Achievement announcement:
> 🏆 Aldric: "Conquista desbloqueada: [NAME]! [ACHIEVEMENT_DESCRIPTION]"

## Step 5: Write Updated Profile

```bash
python3 -c "
import json, os
from datetime import date

path = os.path.expanduser('~/.logic-quest/profile.json')
with open(path) as f:
    d = json.load(f)

# Apply all updates here (replace with actual values from Step 2-4)
d['xp'] = NEW_XP
d['level'] = NEW_LEVEL
d['xp_next_level'] = NEW_XP_NEXT
d['last_session'] = str(date.today())
# d['error_history']['KEY'] = NEW_COUNT  # if error occurred
# d['completed_quests'].append('QUEST_ID')  # if quest completed
# d['current_quest'] = 'NEXT_QUEST_ID'  # if quest completed
# d['achievements'].append('ACHIEVEMENT_ID')  # if unlocked

with open(path, 'w') as f:
    json.dump(d, f, indent=2, ensure_ascii=False)
print('Profile saved')
"
```

Verify the write succeeded (output should be "Profile saved").
