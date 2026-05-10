# logic-quest — Gemini CLI Notes

## Tool Mapping

This plugin uses Claude Code tool names. In Gemini CLI, substitute:

| Claude Code | Gemini CLI |
|-------------|------------|
| `Skill` tool | `activate_skill` tool |
| `Read`, `Write`, `Edit` | Your native file tools |
| `Bash` | Shell execution |

## Skills

Use `activate_skill` to load any skill by name:
- `logic-quest:start` — begin or resume session
- `logic-quest:quest` — start or continue a chapter
- `logic-quest:exercise` — get a practice exercise
