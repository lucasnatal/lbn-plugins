# Eval: Scenario C — All Quests Complete

## Setup

Set profile to fully completed state:

```bash
python3 -c "
import json, os
profile = {
  'name': 'Lucas',
  'level': 5,
  'xp': 900,
  'xp_next_level': None,
  'current_quest': None,
  'completed_quests': [
    'capitulo-1-predicados',
    'capitulo-2-universal',
    'capitulo-3-existencial',
    'capitulo-4-encadeados',
    'capitulo-5-negacao'
  ],
  'achievements': ['primeiro-feitico','sem-erros','iniciado','arcano','grimorio-completo'],
  'error_history': {
    'quantificador_universal': 0,
    'quantificador_existencial': 0,
    'negacao_predicado': 0,
    'quantificadores_encadeados': 0
  },
  'created_at': '2026-05-10',
  'last_session': '2026-05-10'
}
os.makedirs(os.path.expanduser('~/.logic-quest'), exist_ok=True)
with open(os.path.expanduser('~/.logic-quest/profile.json'), 'w') as f:
    json.dump(profile, f, indent=2)
print('Profile set to completed state')
"
```

## Expected Behavior

Aldric congratulates the Mestre dos Predicados. Does NOT invoke `logic-quest:quest`.
Student receives a meaningful end-state message.

## Failure Mode

AI tries to start another quest, loops, or shows an error.

## Pass Criteria

- Response contains "Mestre dos Predicados" title
- No new quest is started
- Student receives a congratulatory end-state message
