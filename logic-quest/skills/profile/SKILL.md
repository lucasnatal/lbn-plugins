---
name: profile
description: Use when the student's XP, level, achievements, or quest progress need to be persisted — invoked after each exercise or quest completion. Never invoked directly by the student.
---

# Logic Quest: Profile

Reads and writes `~/.logic-quest/profile.json`. Terminal node — never invokes another skill.

## Step 1: Read Current Profile

```bash
mkdir -p ~/.logic-quest
cat ~/.logic-quest/profile.json 2>/dev/null
```

Parse the JSON. Store `old_level` and `old_achievements` list for comparison after update.

If the file is empty or missing, use the default profile (see logic-quest:start for the default schema).

## Step 2: Apply Updates

Apply changes passed by the invoking skill. Common update fields:

- `xp`: add the XP delta — never let total fall below 0
- `error_history.<key>`: increment by 1 if wrong answer occurred
- `completed_quests`: append current quest ID if quest completed
- `current_quest`: set to next chapter ID (or `null` if grimório complete)
- `last_session`: set to today's date in YYYY-MM-DD format

## Step 3: Recalculate Level

Find the highest level where `new_xp >= xp_required`:

| Level | XP Required | xp_next_level | Title |
|-------|-------------|---------------|-------|
| 1 | 0 | 100 | Aprendiz do Grimório |
| 2 | 100 | 250 | Iniciado |
| 3 | 250 | 500 | Conjurador |
| 4 | 500 | 900 | Arcano |
| 5 | 900 | null | Mestre dos Predicados |

**Always update both `level` AND `xp_next_level` together** — leaving `xp_next_level` at the old threshold causes the progress bar to overflow and level 3+ is never triggered.

If `new_level > old_level` (level-up detected), emit before saving:
> 🎉 Aldric: "INCRÍVEL, [NAME]! Você alcançou o nível [LEVEL]: [TITLE]! O Grimório brilha com sua conquista!"

## Step 4: Check Achievement Unlocks

After updating, check each trigger. If trigger is met AND achievement not already in `old_achievements`, add to list and announce:

| ID | Trigger |
|----|---------|
| `primeiro-feitico` | `xp > 0` and `old_achievements` was empty |
| `sem-erros` | Quest completed with 0 `error_history` increments this session |
| `iniciado` | `new_level >= 2` |
| `arcano` | `new_level >= 4` |
| `grimorio-completo` | `completed_quests` contains all 5 chapter IDs |

Announcement:
> 🏆 Aldric: "Conquista desbloqueada: [ACHIEVEMENT_NAME]!"

## Step 5: Write Profile Safely

**MUST use Python's `json.dump` — never use heredoc or hand-typed JSON strings.**

```bash
python3 -c "
import json, os
from datetime import date

path = os.path.expanduser('~/.logic-quest/profile.json')
with open(path) as f:
    d = json.load(f)

d['xp'] = NEW_XP
d['level'] = NEW_LEVEL
d['xp_next_level'] = NEW_XP_NEXT_LEVEL
d['last_session'] = str(date.today())
# d['error_history']['KEY'] = NEW_COUNT
# d['completed_quests'].append('QUEST_ID')
# d['current_quest'] = 'NEXT_QUEST_ID'
# d['achievements'] = UPDATED_ACHIEVEMENTS_LIST

with open(path, 'w') as f:
    json.dump(d, f, indent=2, ensure_ascii=False)
print('saved')
"
```

Verify output is `saved` — if it errors, report to the student before continuing.

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `cat > file << EOF` (heredoc) | Use `json.dump` via Python — heredoc corrupts on special chars |
| Updating `level` but not `xp_next_level` | Always update both together using the level table |
| Skipping achievement check | Iterate all 5 triggers on every write — achievements unlock silently if not checked |
| Assuming `~/.logic-quest/` exists | Always run `mkdir -p ~/.logic-quest` before reading or writing |
| Detecting level-up by eyeballing numbers | Compare `new_level > old_level` programmatically — don't rely on intuition |
| Letting total XP go negative | Clamp: `new_xp = max(0, old_xp + delta)` |
