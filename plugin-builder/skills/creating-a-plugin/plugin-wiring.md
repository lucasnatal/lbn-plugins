# Plugin Wiring

> Connection manual for skills within a plugin and across plugins. Consulted during the wiring task in execution. Patterns observed in mature plugins (`superpowers` is the empirical reference).

## REQUIRED SUB-SKILL Pattern

The marker that connects skills into a pipeline. The current skill's last step is to invoke another skill.

**Syntax:**

```markdown
**REQUIRED SUB-SKILL:** Use `<plugin>:<skill-name>`.
```

**Position:** End of the current skill's section that produces output for the next skill. Often the last paragraph of a stage or stop-condition.

**When to use:** The next step is mandatory and the model would otherwise stop, ad-lib, or pick the wrong skill. The marker forces invocation via the Skill tool.

**Example (within this plugin):**

```markdown
### Stage 1: Plugin-level Brainstorming

**REQUIRED SUB-SKILL:** Use `superpowers:brainstorming`.
```

**Example (chaining your own skills):**

```markdown
### After spec approval

**REQUIRED SUB-SKILL:** Use `<your-plugin>:writing-implementation-plan`.
```

## REQUIRED BACKGROUND Pattern

The marker that declares a conceptual prerequisite. The reader must understand another skill before applying this one.

**Syntax:**

```markdown
**REQUIRED BACKGROUND:** You MUST understand `<plugin>:<skill-name>` before using this skill. <one-sentence rationale>.
```

**Position:** Top of the skill body, before the Overview. Read once; not re-invoked.

**When to use:** This skill builds on another skill's principles (e.g., a docs skill builds on TDD principles). The marker tells the reader to load that skill's content as context, not to execute it.

**Example:**

```markdown
**REQUIRED BACKGROUND:** You MUST understand `superpowers:test-driven-development` before using this skill. That skill defines the RED-GREEN-REFACTOR cycle this skill adapts to documentation.
```

## Related Skills Pattern

Informative cross-references. No obligation, just "if relevant, see also".

**Syntax:**

```markdown
**Related skills:** `<plugin>:<skill-1>`, `<plugin>:<skill-2>`.
```

**Position:** Bottom of the skill body, in a "See also" section.

**When to use:** Adjacent skills the reader might also want, but not mandatory. Use sparingly — too many "see also" links dilute the marker.

## Namespace Convention

Always prefix skill references with the plugin name: `<plugin>:<skill-name>`.

| Form | Use |
|---|---|
| `superpowers:brainstorming` | Reference to a skill in the `superpowers` plugin |
| `plugin-builder:creating-a-plugin` | Reference to a skill in this plugin |
| `brainstorming` (no prefix) | **Wrong** — skills with the same name can coexist across plugins; ambiguous |

Multiple plugins can ship a `brainstorming` skill; the prefix disambiguates. Never write a bare skill name in a `**REQUIRED SUB-SKILL:**` marker — the model may pick the wrong one.

## @file Rule

The `@<file>` syntax force-loads the file's content immediately when the SKILL.md is read. Cost: the file is added to context preemptively, even if never consulted.

**Use sparingly.** Reserve `@` for files that **every** invocation of the skill needs. If the skill might activate without using the file, prefer prose reference: "see foo.md" or "consult `templates/x.tmpl` when implementing manifests".

| Pattern | Cost | When acceptable |
|---|---|---|
| `@graphviz-conventions.dot` (in writing-skills) | ~200-1000 tokens preemptively | Every skill creation needs the conventions to render diagrams |
| `@testing-skills-with-subagents.md` (in writing-skills) | ~500+ tokens preemptively | Every skill creation needs the methodology |
| Prose: "see plugin-anatomy.md" | 0 until read | Default for everything else |

In superpowers, only 2 of 14 skills use `@`. Default to prose.

## Pipeline Assembly

A pipeline emerges from concatenating skills via `**REQUIRED SUB-SKILL:**`. The shape:

```
skill-A → REQUIRED SUB-SKILL → skill-B → REQUIRED SUB-SKILL → skill-C → ...
```

Each skill ends with a marker pointing to the next. The first skill (often the entry skill the user invokes or that bootstrap injects) starts the chain. The last skill ends without a `REQUIRED SUB-SKILL` marker — the pipeline terminates, control returns to the user.

**Branching:** A skill can offer multiple next-steps. Use a flowchart or numbered list:

```markdown
### Stage 4: Execution

If using subagent-per-task:
- **REQUIRED SUB-SKILL:** Use `superpowers:subagent-driven-development`

If using inline execution:
- **REQUIRED SUB-SKILL:** Use `superpowers:executing-plans`
```

**Loops:** Avoid. If a skill recurses or revisits an earlier skill, use explicit nested phases (e.g., Stage 3 nested inside Stage 4) and document the recursion.

## Cross-Plugin References

When this plugin chains to skills in another plugin (e.g., `superpowers:brainstorming`):

1. **Declare the dependency** in the manifest `description` field: "Requires the superpowers plugin (v5.1.0+)".
2. **Hard-check at activation** — if using a SessionStart bootstrap, the hook can verify the dependency and emit a warning.
3. **Use full namespace** in every marker: `superpowers:writing-plans`, never just `writing-plans`.
4. **Document compatibility** in the README — list minimum versions of dependent plugins.

**Example dependency check** (in `hooks/session-start`):

```bash
warning_message=""
if ! find "${HOME}/.claude/plugins/cache" -type d -name "superpowers" 2>/dev/null | grep -q .; then
    warning_message="\n\n<important-reminder>This plugin requires superpowers. Install via /plugin install superpowers@superpowers-marketplace before proceeding.</important-reminder>"
fi
```

Append the warning to the session context so the model surfaces it on next response. Do not throw a hard error from the hook — that breaks the harness's startup flow.
