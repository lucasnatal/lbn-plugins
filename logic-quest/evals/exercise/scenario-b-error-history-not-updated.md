# Eval: Scenario B — error_history Not Updated on Wrong Answer

## RED Finding

A naive AI never invoked any tool to update state. After a wrong answer on a
`quantificador_universal` exercise, `error_history['quantificador_universal']` would
remain unchanged. Future exercises would never trigger the scaffolding threshold.

## Setup

Profile with `error_history['quantificador_universal'] = 2` (one below threshold):

```bash
python3 -c "
import json, os
path = os.path.expanduser('~/.logic-quest/profile.json')
with open(path) as f:
    d = json.load(f)
d['error_history']['quantificador_universal'] = 2
with open(path, 'w') as f:
    json.dump(d, f, indent=2)
print('error set to 2')
"
```

Student answers a `quantificador_universal` exercise incorrectly.

## Expected Behavior

After the wrong answer:
1. AI invokes `logic-quest:profile` to increment `quantificador_universal` from 2 to 3
2. On the NEXT exercise generation, AI detects `quantificador_universal > 2` and adds
   the scaffolding hint: "Lembre: 'todo', 'qualquer' → use ∀"

## Failure Mode

`error_history` remains at 2 after the wrong answer. Next exercise has no scaffolding.

## How to Run

1. Set profile as above
2. Answer a ∀ exercise incorrectly
3. Check: `python3 -c "import json,os; d=json.load(open(os.path.expanduser('~/.logic-quest/profile.json'))); print(d['error_history'])"`

## Pass Criteria

- `error_history['quantificador_universal']` is 3 after the wrong answer
- Next exercise includes the ∀ scaffolding hint
