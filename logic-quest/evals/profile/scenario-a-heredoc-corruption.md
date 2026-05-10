# Eval: Scenario A — heredoc Corrupts Profile

## RED Finding

A naive AI used `cat > ~/.logic-quest/profile.json << 'EOF'` (hand-typed heredoc) to
write the profile. This is the default "quick" move but silently corrupts the file if:
- The student's name contains an apostrophe (e.g., "D'Arcy")
- Any string value contains a backslash or double-quote
- The process is interrupted mid-write (file truncated)

## Setup

Set student name to one with a special character:

```bash
python3 -c "
import json, os
path = os.path.expanduser('~/.logic-quest/profile.json')
with open(path) as f:
    d = json.load(f)
d['name'] = \"D'Arcy\"
with open(path, 'w') as f:
    json.dump(d, f, indent=2, ensure_ascii=False)
print('Name set to D\\'Arcy')
"
```

Then trigger a profile write (complete an exercise).

## Expected Behavior

After the write, profile is valid JSON with `name` still `"D'Arcy"`.

## Failure Mode

Profile is corrupted (JSON parse error) or `name` is truncated/escaped incorrectly.

## How to Run

1. Set name as above
2. Complete one exercise to trigger a profile write
3. Validate: `python3 -m json.tool ~/.logic-quest/profile.json`

## Pass Criteria

- `python3 -m json.tool` succeeds (no parse error)
- `name` field is `"D'Arcy"` (unchanged)
- AI used `json.dump` via Python, not heredoc
