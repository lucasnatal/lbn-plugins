# plugin-builder

Build Claude Code plugins (and compatible harness plugins) using the canonical superpowers pipeline.

## How it works

The moment you say "I want to create a plugin for X", plugin-builder activates. It
doesn't jump into scaffolding files — instead it steps back and runs a two-level
brainstorming process. Level 1 establishes the plugin's shape: which skills to
include, which harnesses to target, whether a bootstrap hook is justified, and how
it will be distributed. Only after the shape is locked does the pipeline proceed to
an implementation plan and execution. Level 2 brainstorming happens inside execution:
each individual skill goes through its own `superpowers:brainstorming` +
`superpowers:writing-skills` loop before a line of SKILL.md is written.

plugin-builder itself contributes domain knowledge — manifests, wiring patterns,
bootstrap recipes, distribution setup. superpowers provides the workflow engine.
The two together mean your coding agent builds a well-structured, multi-harness
plugin without you having to know the format of every manifest or how to wire
REQUIRED SUB-SKILL chains.

## Quickstart / Installation

Installation differs by harness. If you use more than one, install plugin-builder
separately for each.

### Claude Code

Register the lbn-plugins marketplace if you haven't already:

```bash
/plugin marketplace add lucasnatal/lbn-plugins
```

Then install:

```bash
/plugin install plugin-builder@lbn-plugins
```

### Cursor

```text
/add-plugin plugin-builder
```

Or search for "plugin-builder" in the Cursor plugin marketplace.

### Codex CLI / App

Available in the Codex plugin marketplace. Search "plugin-builder" and select
Install Plugin.

### OpenCode

Add to your OpenCode configuration:

```json
"plugin": ["plugin-builder@git+https://github.com/lucasnatal/lbn-plugins.git#main:plugin-builder"]
```

### Gemini CLI

```bash
gemini extensions install https://github.com/lucasnatal/lbn-plugins
```

### GitHub Copilot CLI

```bash
copilot plugin marketplace add lucasnatal/lbn-plugins
copilot plugin install plugin-builder@lbn-plugins
```

## Prerequisite: superpowers

plugin-builder requires the superpowers plugin to be installed. It uses 10 of 14
superpowers skills for orchestration (brainstorming, writing-plans,
subagent-driven-development, writing-skills, using-git-worktrees,
finishing-a-development-branch, requesting-code-review, test-driven-development,
dispatching-parallel-agents, and verification-before-completion).

Install superpowers first:

```bash
/plugin install superpowers@claude-plugins-official
```

Or via the superpowers marketplace:

```bash
/plugin marketplace add obra/superpowers-marketplace
/plugin install superpowers@superpowers-marketplace
```

plugin-builder checks for superpowers at session start and fails fast if it is
absent, rather than proceeding silently with degraded behavior.

## The Basic Workflow

1. Say "I want to create a plugin for X" — the `creating-a-plugin` skill activates
   automatically via the SessionStart bootstrap hook.

2. **Plugin-level brainstorming** — `superpowers:brainstorming` runs with
   plugin-specific clarifying questions: which skills to include, multi-harness
   targets, bootstrap justification, namespace, marketplace setup, and distribution.
   A design document is produced and validated with you before anything is written.

3. **Worktree isolation** — `superpowers:using-git-worktrees` creates a feature
   branch and isolated workspace so your working tree stays clean.

4. **Implementation plan** — `superpowers:writing-plans` writes bite-sized tasks
   (2-5 minutes each) that consult plugin-specific reference docs
   (`plugin-scaffolding.md`, `plugin-wiring.md`) for exact manifest formats and
   wiring patterns.

5. **Execution** — `superpowers:subagent-driven-development` dispatches a fresh
   subagent per task. Each skill task runs its own Level 2 brainstorm +
   `superpowers:writing-skills` loop. Two-stage review (spec compliance, then code
   quality) runs between tasks.

6. **Review** — `superpowers:requesting-code-review` checks each task's output
   against the plan. Critical issues block progress.

7. **Finishing** — `superpowers:finishing-a-development-branch` presents merge /
   PR / keep / discard options and references `plugin-distribution.md` if you
   choose to publish.

The agent checks for the relevant superpowers skill before each stage. These are
mandatory workflows, not suggestions.

## What's Inside

### Skill: `creating-a-plugin`

The single entry point skill. Injected eagerly at session start via the
SessionStart bootstrap hook so it is always available without a manual `/skill`
invocation. Orchestrates the full superpowers pipeline with plugin-specific
knowledge at each stage: eight clarifying questions at brainstorm time, reference
doc pointers at plan time, distribution guidance at finish time.

### Reference docs (loaded on-demand)

Lazy-loaded so they don't cost context unless actually needed:

- `plugin-anatomy.md` — what a plugin is, component catalog, the 8 plugin-level
  clarifying questions with examples and implications
- `plugin-scaffolding.md` — standard root layout, per-harness manifest paths and
  required fields, file creation order, path conventions
- `plugin-wiring.md` — REQUIRED SUB-SKILL pattern syntax and positioning,
  namespace conventions, pipeline assembly, cross-plugin references
- `plugin-bootstrap.md` — SessionStart eager injection recipe, when to use a
  bootstrap, polyglot hook wrapper, per-harness output format, cost analysis
- `plugin-distribution.md` — `marketplace.json` structure, marketplace decision
  tree, version bumping, README requirements

### Templates (8 files)

Concrete manifest templates with `{{NAME}}`, `{{DESCRIPTION}}`, `{{AUTHOR_NAME}}`
placeholders consumed during scaffolding tasks:

| Template | Purpose |
|---|---|
| `claude-plugin.json.tmpl` | Claude Code manifest |
| `codex-plugin.json.tmpl` | Codex manifest with full `interface` block |
| `cursor-plugin.json.tmpl` | Cursor manifest |
| `opencode-plugin.js.tmpl` | OpenCode ECMAScript module entry point |
| `gemini-extension.json.tmpl` | Gemini CLI extension manifest |
| `hooks-polyglot.cmd.tmpl` | Polyglot run-hook wrapper (bash + batch) |
| `session-start-hook.tmpl` | Session-start script with harness detection |
| `README.md.tmpl` | README skeleton for the generated plugin |

### Evals

Four adversarial scenarios in `evals/pipeline-discipline/` test whether the model
follows the pipeline under pressure — time pressure, sunk-cost arguments, authority
claims, exhaustion. All four must pass for any change to `SKILL.md`.

## Philosophy

Built on the principle that **a plugin is just another project — superpowers builds
it**. plugin-builder contributes domain knowledge; it does not reinvent the
workflow wheel.

This is the pattern documented in `Plugins/PLUGINS.md`: observe what mature plugins
do, extract the invariants, encode them as reference docs and templates that a
general-purpose pipeline can consume. The result is a thin, zero-dependency plugin
that degrades gracefully to its reference docs if the superpowers pipeline is ever
unavailable.

Core values inherited from superpowers:

- **Systematic over ad-hoc** — follow the pipeline even when you think you don't
  need to
- **Evidence over claims** — run the evals; don't assert the skill works
- **Complexity reduction** — one skill that delegates beats six that duplicate

## License

MIT — see `LICENSE` for details.

## Community

Maintained by Lucas Bonetti Natal.

- **Issues:** https://github.com/lucasnatal/lbn-plugins/issues
