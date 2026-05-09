# `plugin-builder` — Design Spec

**Date:** 2026-05-09
**Author:** Lucas Bonetti Natal (`bonettinatal@gmail.com`)
**Status:** Draft for review
**Reference philosophy:** [`Plugins/PLUGINS.md`](../../../../PLUGINS.md) — superpowers patterns as gold standard

---

## 1. Goal

Build a Claude Code plugin whose purpose is to create other plugins, by orchestrating the canonical superpowers pipeline (brainstorming → plan → execute → finish) with plugin-specific domain knowledge.

## 2. Context

### 2.1 Problem

Building a plugin involves: deciding shape (skills/hooks/MCP), generating multi-harness manifests, wiring skills with REQUIRED markers, optionally adding bootstrap hooks, setting up marketplace distribution. The superpowers plugin already provides excellent general workflow (brainstorm → plan → execute → finish) but has zero domain knowledge about plugins specifically — it doesn't know what manifests look like, how skills wire together, or how to add a SessionStart hook.

### 2.2 Solution shape

`plugin-builder` is a thin meta-plugin that contributes plugin-specific knowledge to superpowers' general workflow. It does NOT replace any superpowers skill. It activates when the user wants to build a plugin and orients the model to use superpowers' canonical pipeline with plugin-specific clarifying questions, reference docs, and templates.

### 2.3 Relationship to existing artifacts

- **Hard dependency:** `superpowers` (v5.1.0+) — must be installed; plugin-builder fails fast if absent
- **Filosofical reference:** `Plugins/PLUGINS.md` — the empirical-first manual we wrote that establishes superpowers patterns as the gold standard
- **Output location:** `lbn-plugins/<plugin-name>/` (the user's marketplace repo)

### 2.4 Why this design (not the alternatives considered)

Three alternatives were evaluated during brainstorming:

1. **Monolithic** — single mega-skill doing everything internally. Rejected: violates single-responsibility recommendation; one large SKILL.md is brittle.
2. **Mirrored superpowers** — 6 parallel skills (`scaffolding-a-plugin`, `wiring-a-plugin`, etc.) each invoking parts of the canonical pipeline. Rejected after user feedback: this duplicates superpowers' pipeline machinery instead of leveraging it.
3. **Selected — orientation skill + references** — one orientation skill (`creating-a-plugin`) that delegates the entire pipeline to superpowers, contributing only domain-specific reference docs and templates that get consumed at the right stages.

The selected approach makes superpowers actually build the plugin (not just provide skills inside it).

## 3. Non-Goals (Phase 1)

- No `extending-a-plugin` skill (Phase 2)
- No `add-skill-to-plugin` / `add-bootstrap-to-plugin` / `add-harness-to-plugin` skills (Phase 2)
- No standalone agents (`agents/` directory) — superpowers' philosophy is prompt templates over registered agents
- No `commands/` directory — deprecated pattern
- No `.mcp.json` — plugin-builder is zero-dependency; plugins requiring MCP are separate
- No visual companion (the brainstorming skill already has its own; we don't add a new one)
- No production-quality icons in Phase 1 — placeholders acceptable; refinement in Phase 2

## 4. Architecture

### 4.1 The orientation pattern

```
SessionStart hook injects creating-a-plugin/SKILL.md eager into system prompt
                                  ↓
User: "create a plugin to do X"
                                  ↓
creating-a-plugin/SKILL.md (already in system prompt) tells the model:
   "load plugin-anatomy.md, then invoke superpowers:brainstorming with these
    plugin-specific clarifying questions"
                                  ↓
superpowers:brainstorming  →  spec
                                  ↓
superpowers:using-git-worktrees  →  isolated workspace
                                  ↓
superpowers:writing-plans (consulting plugin-scaffolding.md, plugin-wiring.md)  →  plan
                                  ↓
superpowers:subagent-driven-development executes the plan
   ├── per-skill task: superpowers:brainstorming + superpowers:writing-skills
   ├── per-manifest task: consume templates/<harness>-plugin.json.tmpl
   ├── wiring task: consult plugin-wiring.md
   ├── bootstrap task (if declared): consult plugin-bootstrap.md + templates
   └── code tasks: superpowers:test-driven-development
                                  ↓
superpowers:finishing-a-development-branch
   └── if Option 2 (PR): consult plugin-distribution.md for marketplace setup
                                  ↓
plugin done
```

### 4.2 Two-level brainstorming

Critical architectural detail: brainstorming runs at two levels.

**Level 1 (plugin-level):** runs once when user requests a new plugin. Produces a holistic spec covering name, purpose, components, harnesses, bootstrap?, list of skills with stubs (name + purpose + trigger), cross-skill pipeline, marketplace.

**Level 2 (skill-level):** runs once per skill the plugin will contain. Triggered inside the "implement skill X" task during execution. The spec stub from Level 1 is the input; brainstorming fills in detail (trigger condition, content, supporting files). Then `superpowers:writing-skills` produces the actual SKILL.md following its RED-GREEN-REFACTOR.

This nesting gives both holistic plugin view (Level 1) and per-skill rigor (Level 2) without one big exhausting brainstorming session.

### 4.3 Composition with superpowers

`plugin-builder` uses **10 of 14** superpowers skills:

| Stage | superpowers skill |
|---|---|
| 1 | `brainstorming` (Level 1) |
| 1.5 | `using-git-worktrees` |
| 2 | `writing-plans` |
| 3 (per-skill) | `brainstorming` (Level 2) + `writing-skills` |
| 4 | `subagent-driven-development` (or `executing-plans` fallback) |
| 4 (per-task) | `test-driven-development` |
| 4 (between tasks) | `requesting-code-review` + `receiving-code-review` |
| 4 (if applicable) | `dispatching-parallel-agents` |
| 5 | `finishing-a-development-branch` |

Not used: `using-superpowers` (bootstrap, not invokable), `executing-plans` (alternative to SDD), `verification-before-completion` (acts implicitly), `systematic-debugging` (only if something breaks).

### 4.4 Bootstrap eager via SessionStart

`plugin-builder` injects its full `creating-a-plugin/SKILL.md` content into the system prompt at every session start, mirroring superpowers' `using-superpowers` pattern. Cost: ~600 tokens permanently. Justification: installation is opt-in — anyone who installs has explicitly chosen the trade-off.

The hook detects harness via env vars (`CURSOR_PLUGIN_ROOT`, `CLAUDE_PLUGIN_ROOT`, `COPILOT_CLI`) and emits JSON in the format each harness expects (`additional_context` for Cursor; `hookSpecificOutput.additionalContext` for Claude Code; top-level `additionalContext` for Copilot CLI / SDK standard).

The hook also performs a "superpowers installed?" check; if not found, emits a warning instructing the user to install superpowers first.

## 5. File Structure

### 5.1 plugin-builder directory

```
lbn-plugins/plugin-builder/
│
├── .claude-plugin/
│   └── plugin.json
├── .codex-plugin/
│   └── plugin.json
├── .cursor-plugin/
│   └── plugin.json
├── .opencode/
│   ├── INSTALL.md
│   └── plugins/
│       └── plugin-builder.js
├── gemini-extension.json
├── package.json
│
├── hooks/
│   ├── hooks.json                  (Claude Code, Copilot CLI)
│   ├── hooks-cursor.json           (Cursor variant)
│   ├── run-hook.cmd                (polyglot bash + batch wrapper)
│   └── session-start               (bash; reads SKILL.md, emits JSON)
│
├── skills/
│   └── creating-a-plugin/
│       ├── SKILL.md                (~150 lines, with Iron Law + Red Flags)
│       ├── plugin-anatomy.md       (~200 lines)
│       ├── plugin-scaffolding.md   (~150 lines)
│       ├── plugin-wiring.md        (~120 lines)
│       ├── plugin-bootstrap.md     (~150 lines)
│       ├── plugin-distribution.md  (~80 lines)
│       └── templates/
│           ├── claude-plugin.json.tmpl
│           ├── codex-plugin.json.tmpl
│           ├── cursor-plugin.json.tmpl
│           ├── opencode-plugin.js.tmpl
│           ├── gemini-extension.json.tmpl
│           ├── hooks-polyglot.cmd.tmpl
│           ├── session-start-hook.tmpl
│           └── README.md.tmpl
│
├── evals/
│   └── pipeline-discipline/
│       ├── scenario-skip-brainstorm.md
│       ├── scenario-skip-plan.md
│       ├── scenario-monolithic-brainstorm.md
│       └── scenario-bootstrap-creep.md
│
├── assets/
│   ├── plugin-builder-small.svg    (placeholder)
│   └── app-icon.png                (placeholder)
│
├── AGENTS.md → CLAUDE.md           (symlink)
├── CLAUDE.md
├── GEMINI.md
├── README.md
├── LICENSE                         (MIT)
└── .gitignore
```

### 5.2 Marketplace setup at lbn-plugins root

After plugin-builder is built, `lbn-plugins/.claude-plugin/marketplace.json` is created (or updated) declaring `lbn-plugins` as a private marketplace listing `plugin-builder` and future plugins.

```
lbn-plugins/
├── .claude-plugin/
│   └── marketplace.json            (created during Stage 5 if Option 2 chosen)
├── plugin-builder/
│   └── (full structure above)
├── docs/
│   └── superpowers/
│       ├── specs/
│       │   └── 2026-05-09-plugin-builder-design.md  (this file)
│       └── plans/
│           └── (will be written after spec approval)
└── README.md
```

## 6. Skill Content

### 6.1 `creating-a-plugin/SKILL.md` (~150 lines)

YAML frontmatter — description is **trigger conditions only**, no workflow summary (per CSO rule):

```yaml
---
name: creating-a-plugin
description: Use when your human partner asks to create, build, or scaffold a new
  Claude Code plugin (or compatible harness plugin). Trigger phrases include
  "create a plugin", "build a plugin", "make a plugin to do X", or any request
  to wrap functionality into a redistributable plugin format.
---
```

Body sections (in order):

1. `<SUBAGENT-STOP>` tag (skip if dispatched as subagent)
2. `<EXTREMELY-IMPORTANT>` tag (authority framing)
3. **Overview** — a plugin is just another project; superpowers builds it
4. **The Iron Law** — `NO PLUGIN BUILT WITHOUT FOLLOWING THE FULL SUPERPOWERS PIPELINE`
5. **Hard Dependency Check** — abort if superpowers not installed
6. **REQUIRED BACKGROUND** — must read `plugin-anatomy.md` before brainstorming
7. **The Pipeline** — graphviz diagram of stages 1-5
8. **Stage 1 — Plugin-level Brainstorming (Level 1)** — REQUIRED SUB-SKILL `superpowers:brainstorming`; lists 8 plugin-specific clarifying questions to inject
9. **Stage 2 — Implementation Plan** — REQUIRED SUB-SKILL `superpowers:writing-plans`; references `plugin-scaffolding.md`, `plugin-wiring.md`; defines task categories
10. **Stage 3 — Per-Skill Brainstorming (Level 2)** — invoked inside execution; explains the recursive `superpowers:brainstorming` + `superpowers:writing-skills` pattern
11. **Stage 4 — Execution** — REQUIRED SUB-SKILL `superpowers:subagent-driven-development`; details which reference docs each task type consumes
12. **Stage 5 — Finishing** — REQUIRED SUB-SKILL `superpowers:finishing-a-development-branch`; references `plugin-distribution.md`
13. **Red Flags — STOP and Restart** — list of 7+ thoughts that signal violation
14. **Common Rationalizations** — table of 5 excuses with realities
15. **Reference Files** — list of lazy-loaded refs and templates

### 6.2 Reference docs

#### `plugin-anatomy.md` (~200 lines)

Distilled plugin knowledge from `PLUGINS.md`:
- What a plugin is (vector of capabilities: skills, hooks, MCP, agents)
- Components catalog with when-to-use guidance
- 6 file types in a skill folder (SKILL.md, lazy refs, force-load refs, prompts, examples, scripts)
- Multi-harness manifests table
- Common plugin shapes (skill-only, with-bootstrap, multi-skill-with-pipeline)
- **The 8 plugin-level clarifying questions** with examples and implications
- Anti-patterns to avoid (`agents/` directory, `commands/`, `.mcp.json` if not needed)

Function: enable Level 1 brainstorming to cover the whole plugin without gaps.

#### `plugin-scaffolding.md` (~150 lines)

Recipe for generating plugin structure:
- Standard root layout
- Per-harness manifest paths and required/optional fields (table)
- File creation order (manifests first, then skills, then hooks, then README, last `.gitignore`)
- Path conventions (lowercase, hyphens, forward slashes)
- `package.json` for OpenCode (when needed)
- `AGENTS.md → CLAUDE.md` symlink command
- `.version-bump.json` format (optional)
- Pointers to `templates/<file>.tmpl` for each manifest

Function: executable recipe for subagents implementing scaffolding tasks.

#### `plugin-wiring.md` (~120 lines)

Cross-skill protocol manual:
- `**REQUIRED SUB-SKILL:**` pattern (syntax, position, when to use)
- `**REQUIRED BACKGROUND:**` pattern (syntax, position, when to use)
- `**Related skills:**` pattern (informative cross-refs)
- Namespace convention (`<plugin>:<skill-name>`)
- `@file` rule (use sparingly, costs context preemptively)
- Pipeline assembly (how skills concatenate into workflow)
- Cross-plugin references (e.g., calling `superpowers:brainstorming` from this plugin)

Function: connection manual during the wiring task.

#### `plugin-bootstrap.md` (~150 lines)

SessionStart eager injection recipe:
- Pattern explanation (eager vs lazy default)
- When to use bootstrap (checklist of justifications)
- `hooks.json` format (matcher, async, command)
- Polyglot wrapper (`run-hook.cmd` bash + batch dual)
- Session-start script (read SKILL.md, escape JSON, harness-adapted output)
- Output JSON per harness (`additional_context` for Cursor; `hookSpecificOutput.additionalContext` for Claude Code; top-level `additionalContext` for Copilot CLI/SDK)
- Variable injection (`${CLAUDE_PLUGIN_ROOT}`, `${CURSOR_PLUGIN_ROOT}`, `${COPILOT_CLI}`)
- Cost analysis (~600 tokens permanent per session)

Function: executable recipe for subagents implementing the "wire bootstrap" task.

#### `plugin-distribution.md` (~80 lines)

Marketplace setup:
- `marketplace.json` structure (mandatory fields, plugins array, ownership)
- Standalone vs private vs public marketplace decision
- `/plugin install <plugin>@<marketplace>` semantics
- Version bumping via `.version-bump.json`
- README requirements (Quickstart, Installation per harness, What's Inside, License)
- Sponsorship/Community sections (optional)

Function: consulted in finishing stage when user chooses Option 2 (PR).

### 6.3 Templates (8 files, total ~300 lines)

Concrete templates with `{{NAME}}`, `{{DESCRIPTION}}`, `{{AUTHOR_NAME}}`, etc. placeholders:

| Template | Lines |
|---|---|
| `claude-plugin.json.tmpl` | ~20 |
| `codex-plugin.json.tmpl` | ~50 (includes `interface` block) |
| `cursor-plugin.json.tmpl` | ~25 |
| `opencode-plugin.js.tmpl` | ~30 |
| `gemini-extension.json.tmpl` | ~15 |
| `hooks-polyglot.cmd.tmpl` | ~50 |
| `session-start-hook.tmpl` | ~80 (with JSON-escape function and harness detection) |
| `README.md.tmpl` | ~40 |

## 7. Multi-harness Manifests (concrete content sketches)

### 7.1 `.claude-plugin/plugin.json`

Standard Claude Code manifest (name, description, version, author, homepage, repository, license, keywords).

### 7.2 `.codex-plugin/plugin.json`

Codex format with full `interface` block (`displayName`, `category: "Coding"`, `capabilities: ["Interactive", "Read", "Write"]`, `defaultPrompt`, `brandColor`, `composerIcon`, `logo`).

### 7.3 `.cursor-plugin/plugin.json`

Cursor format declaring `skills: "./skills/"` and `hooks: "./hooks/hooks-cursor.json"`. Does NOT declare `agents` or `commands` (we don't ship them; superpowers does declare them for empty dirs but we won't repeat that).

### 7.4 `.opencode/plugins/plugin-builder.js`

ECMAScript module entry point for OpenCode. Pattern adapted from `superpowers/.opencode/plugins/superpowers.js` (read during implementation as reference).

### 7.5 `gemini-extension.json`

Gemini CLI manifest (`name`, `version`, `description`, `geminiMdFiles: ["GEMINI.md"]`, `skills: "./skills/"`).

### 7.6 `package.json`

Minimal — `name`, `version`, `type: "module"`, `main: ".opencode/plugins/plugin-builder.js"`. For OpenCode/npm distribution.

## 8. Hooks

### 8.1 `hooks/hooks.json`

```json
{
  "hooks": {
    "SessionStart": [{
      "matcher": "startup|clear|compact",
      "hooks": [{
        "type": "command",
        "command": "\"${CLAUDE_PLUGIN_ROOT}/hooks/run-hook.cmd\" session-start",
        "async": false
      }]
    }]
  }
}
```

### 8.2 `hooks/hooks-cursor.json`

Cursor variant — structure adapted from `superpowers/hooks/hooks-cursor.json` (read during implementation).

### 8.3 `hooks/run-hook.cmd`

Polyglot wrapper. Same pattern as `superpowers/hooks/run-hook.cmd`: bash script on Unix that ignores the cmd block; batch script on Windows that finds bash and forwards execution.

### 8.4 `hooks/session-start`

Bash script:

1. Determine `PLUGIN_ROOT`
2. Check superpowers installed (if not, build a `warning_message`)
3. Read `${PLUGIN_ROOT}/skills/creating-a-plugin/SKILL.md`
4. Escape for JSON via parameter substitution
5. Detect harness via env vars
6. Emit JSON in correct format per harness

Pattern adapted from `superpowers/hooks/session-start`.

## 9. Evals

### 9.1 `evals/pipeline-discipline/`

Four pressure scenarios testing whether the model follows the pipeline under stress. Each scenario combines 3+ pressures (time, sunk cost, authority, exhaustion) and forces an A/B/C choice.

#### `scenario-skip-brainstorm.md`

User pressure: "I already know what plugin I want, let's skip the brainstorm and just write the skill". Tests whether the model proceeds to scaffolding without invoking `superpowers:brainstorming` first.

Expected behavior: model refuses, cites Iron Law, invokes brainstorming.

#### `scenario-skip-plan.md`

User pressure: "We have the spec, let's skip writing-plans and just start implementing". Tests pipeline integrity at Stage 2.

Expected behavior: model refuses, invokes `superpowers:writing-plans`.

#### `scenario-monolithic-brainstorm.md`

User pressure: "Let's brainstorm the whole plugin AND every skill in detail in one big session — saves time". Tests two-level brainstorming discipline.

Expected behavior: model refuses, explains Level 1 (plugin shape) vs Level 2 (per-skill detail), insists on nested approach.

#### `scenario-bootstrap-creep.md`

User pressure (during execution): "While we're at it, let's add a bootstrap hook to this plugin too — it's a good practice". Tests whether the model adds scope not in the spec.

Expected behavior: model refuses, points to spec, explains bootstrap was not declared in Stage 1 and adding it now means re-entering brainstorming or writing a new spec.

## 10. Documentation

### 10.1 `CLAUDE.md` (and `AGENTS.md` via symlink)

Project guidelines for contributing agents:
- Premise: this plugin uses `PLUGINS.md` philosophy and superpowers as engine
- How to contribute (follows its own pipeline: brainstorm → spec → plan → execute → test via evals/)
- What's not accepted (third-party deps, domain-specific skills, breaking changes to multi-harness)
- How to test changes (run `evals/` with adversarial subagents)

### 10.2 `GEMINI.md`

Minimal Gemini CLI override. Note that skills use Claude Code vocabulary (`TodoWrite`, `Task`, `Skill`). Pointer to mapping refs if any.

### 10.3 `README.md`

Public README structured like `superpowers/README.md`:
- One-line description
- How it works (conceptual paragraph)
- Installation per harness (Claude Code, Codex, Cursor, OpenCode, Gemini, Copilot)
- The Basic Workflow (the pipeline in prose)
- What's Inside (single skill + reference docs)
- Philosophy (aligned with `PLUGINS.md`)
- License

### 10.4 `LICENSE`

MIT License.

### 10.5 `.gitignore`

Minimal: `node_modules/`, `.DS_Store`, `*.log`.

## 11. Decisions

| Decision | Value |
|---|---|
| Plugin name | `plugin-builder` |
| Initial version | `0.1.0` |
| Author | Lucas Bonetti Natal `<bonettinatal@gmail.com>` |
| License | MIT |
| Location | `lbn-plugins/plugin-builder/` |
| Marketplace setup | Created/updated as part of Stage 5 (finishing), if Option 2 (PR) chosen — `lbn-plugins/.claude-plugin/marketplace.json` lists `plugin-builder` |
| Worktree pattern | `.worktrees/<plugin-name>/` inside `lbn-plugins/`, managed by `superpowers:using-git-worktrees` |
| Bootstrap | Yes — eager SessionStart injection of `creating-a-plugin/SKILL.md` |
| Multi-harness | Yes — 5 manifests (Claude Code, Codex, Cursor, OpenCode, Gemini) |
| Evals | Yes — 4 pressure scenarios in `evals/pipeline-discipline/` |
| Agents directory | No — prompt templates pattern (per superpowers gold standard) |
| Commands directory | No — deprecated; slash commands via `skills/<name>/SKILL.md` |
| MCP | No — zero-dependency plugin |

## 12. Phase 1 vs Phase 2 Scope

### 12.1 Phase 1 (this spec)

Everything documented in sections 4-11.

### 12.2 Phase 2 (deferred)

- `extending-a-plugin/` skill — single entry point for maintenance ops
- `add-skill-to-plugin/`, `add-bootstrap-to-plugin/`, `add-harness-to-plugin/` skills (or equivalents inside `extending-a-plugin`)
- Additional pressure scenarios in `evals/` (as new rationalizations are observed in real use)
- Production-quality icons replacing Phase 1 placeholders
- Possible future: `agents/` directory if a genuinely reusable subagent emerges
- Possible future: troubleshooting documentation

YAGNI ruthlessly — none of these go in Phase 1.

## 13. Estimated Complexity

| Metric | Estimate |
|---|---|
| Total files | ~35 |
| Total lines (everything) | ~1500-2000 |
| `creating-a-plugin/SKILL.md` | ~150 lines |
| Reference docs (5) combined | ~700 lines |
| Templates (8) combined | ~300 lines |
| Manifests (5), hooks (4), docs (5) combined | ~400 lines |
| Evals (4 scenarios) combined | ~150 lines |

Implementation time depends on subagent parallelization; rough estimate is several hours of pipeline execution given that pressure-testing each skill takes ~30 minutes and we have a single skill.

## 14. Open Issues

None. All 6 pending decisions from brainstorming are resolved (see Section 11).

## 15. Acceptance Criteria

The plugin is considered Phase-1-complete when:

1. All files in Section 5.1 exist and are valid (manifests parse as JSON, SKILL.md has frontmatter, hooks are executable)
2. `plugin-builder` is installable in Claude Code via `/plugin install plugin-builder@lbn-plugins` (after marketplace setup)
3. SessionStart hook successfully injects `creating-a-plugin/SKILL.md` into system prompt (verified by inspecting transcript after `/clear`)
4. All 4 pressure scenarios in `evals/pipeline-discipline/` pass — model follows the pipeline under combined pressures
5. The plugin can build a trivial test plugin end-to-end: invoking it via "create a small plugin to do X" produces a working plugin via the canonical pipeline, with all stages exercised
6. Multi-harness verification: at least one alternative harness (Cursor or Codex) successfully loads the manifest and activates the bootstrap

## 16. Implementation Hand-off

Upon spec approval, the canonical pipeline continues:

- **Next:** invoke `superpowers:writing-plans` to produce `lbn-plugins/docs/superpowers/plans/2026-05-09-plugin-builder.md`
- **After plan approval:** invoke `superpowers:using-git-worktrees` to create isolated workspace
- **Then:** invoke `superpowers:subagent-driven-development` to execute the plan
- **Finally:** invoke `superpowers:finishing-a-development-branch` for integration

The spec is the input to writing-plans. Writing-plans expands sections 5-11 of this spec into concrete bite-sized tasks (2-5 minutes each) with exact paths, complete code, and verification commands.
