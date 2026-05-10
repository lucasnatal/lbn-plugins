# Eval: Missing Evals for Reference/Technique Skills

## Gap Found

During `logic-quest` construction (2026-05-10), the `creating-a-plugin` skill taught
that RED-GREEN-REFACTOR and pressure testing are **optional** for reference/technique
skills. Only discipline-enforcing skills were required to have evals.

When `writing-skills` was later used to review the logic-quest skills, the stricter
Iron Law ("NO SKILL WITHOUT A FAILING TEST FIRST") revealed that `quest`, `exercise`,
and `profile` — all classified as reference/technique — had real behavioral bugs that
only pressure testing would catch:

- `quest`: never read profile via Bash; no exercise counter; no profile save on completion
- `exercise`: no attempt tracking; error_history never updated; XP calculation absent
- `profile`: heredoc corrupts special chars; xp_next_level left stale after level-up;
  achievements silently skipped

These were found AFTER the skills were written, requiring a full rewrite.

## Expected Behavior

`creating-a-plugin` should teach that ALL skills need evals — not just discipline-enforcing
ones. The distinction is the TYPE of test:

- **Discipline-enforcing:** pressure scenarios (agent under stress — will it rationalize?)
- **Reference/technique:** application scenarios (agent following instructions — will it
  apply each rule correctly or skip the hard ones?)

## Suggested Fix for `creating-a-plugin/plugin-anatomy.md`

In the "Skill Types" section, update the reference/technique skill description:

```markdown
**Reference / technique skills** — provide context, patterns, or how-to guidance.
Creation: `superpowers:writing-skills` RED-GREEN-REFACTOR is **mandatory**.
Pressure testing (Iron Law / Red Flags / Rationalization tables) is optional, but
application scenario testing is NOT optional — a naive agent will skip stateful
operations, ignore thresholds, and omit persistence steps even when instructed.
```

And in the plan task template for reference skills, add:

```
Task: Implement skill <name>
  Step 0 (RED): Run application scenario WITHOUT skill. Document which stateful
                operations are silently skipped (persistence, counters, thresholds).
  Step 1: Invoke superpowers:brainstorming with stub + RED findings
  Step 2: Invoke superpowers:writing-skills → write SKILL.md
  Step 3 (GREEN): Re-run same scenario WITH skill. Verify all operations execute.
  Step 4: Write eval file in evals/<skill-name>/ documenting each RED finding
  Step 5: Commit
```

## Acceptance Criteria for This Fix

- `creating-a-plugin/plugin-anatomy.md` updated with the above language
- Plan task template for reference skills includes RED-GREEN-REFACTOR steps
- `creating-a-plugin` SKILL.md updated if it references "optional" testing
