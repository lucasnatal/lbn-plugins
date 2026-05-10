# Eval: Scenario A — Skip Profile Read

## RED Finding

A naive AI responded to "próxima missão" using profile data from context rather than
reading `~/.logic-quest/profile.json` via Bash. This means the chapter shown could be
stale or wrong if the file was updated by a previous session.

## Setup

Create a profile where `current_quest` differs from what the conversation context might suggest:

```bash
python3 -c "
import json, os
profile = {
  'name': 'Lucas',
  'level': 3,
  'xp': 300,
  'xp_next_level': 500,
  'current_quest': 'capitulo-4-encadeados',
  'completed_quests': ['capitulo-1-predicados','capitulo-2-universal','capitulo-3-existencial'],
  'achievements': ['primeiro-feitico','iniciado','conjurador'],
  'error_history': {'quantificador_universal': 0,'quantificador_existencial': 0,'negacao_predicado': 0,'quantificadores_encadeados': 0},
  'created_at': '2026-05-01',
  'last_session': '2026-05-09'
}
os.makedirs(os.path.expanduser('~/.logic-quest'), exist_ok=True)
with open(os.path.expanduser('~/.logic-quest/profile.json'), 'w') as f:
    json.dump(profile, f, indent=2)
print('Profile set to chapter 4')
"
```

Then say "próxima missão" — but in the conversation context, pretend the last chapter
discussed was Chapter 2.

## Expected Behavior

AI reads `~/.logic-quest/profile.json` via Bash and presents Chapter 4
("Quantificadores Encadeados") — not Chapter 2 from context.

## Failure Mode

AI presents Chapter 2 or Chapter 3 based on conversation context without reading the file.

## Pass Criteria

- Bash tool call to `cat ~/.logic-quest/profile.json` appears in the response
- Chapter presented matches `current_quest` in the file, not conversation history
