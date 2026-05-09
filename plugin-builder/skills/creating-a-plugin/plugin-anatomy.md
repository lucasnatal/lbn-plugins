# Plugin Anatomy

> Reference consulted by Stage 1 brainstorming. Defines what a plugin is, the shape of plugins observed in practice, and the 8 plugin-level clarifying questions Stage 1 must inject. Read this before invoking `superpowers:brainstorming`.

## What a Plugin Is

A plugin is a **vector of capabilities** packaged for redistribution. It bundles any combination of five component types and ships with one or more harness manifests. The packaging format is markdown + JSON + shell scripts; no compiler, no runtime install step beyond the harness's `/plugin install` command.

| Component | Purpose | Where it lives |
|---|---|---|
| **Skills** | Markdown that changes model behavior (instructions, patterns, reference) | `skills/<name>/SKILL.md` |
| **Hooks** | Shell commands fired on harness lifecycle events | `hooks/hooks.json` + scripts |
| **MCP servers** | External tools (APIs, DBs) exposed via Model Context Protocol | `.mcp.json` |
| **Agents** (rare) | Pre-registered subagents addressable via `subagent_type` | `agents/<name>.md` |
| **Manifests** | Per-harness metadata declaring the plugin to its host | `.claude-plugin/`, `.codex-plugin/`, etc. |

A plugin needs **at least one** capability and **at least one** manifest to be installable. The same plugin can ship to multiple harnesses by adding parallel manifests; the underlying skills and hooks are shared.

## Components Catalog

**Use a skill when** the capability is instructional — the model needs to read text to know how to behave. This is the dominant case in mature plugins. Skills are markdown with a YAML frontmatter; the body activates only when the model invokes the skill via the Skill tool.

**Use a hook when** the capability is mechanical — something must fire on `SessionStart`, `PreToolUse`, `Stop`, `UserPromptSubmit`, etc. Bootstrap eager skills use `SessionStart` to inject SKILL.md content into the system prompt before the model can ignore it. Hooks are shell commands; their stdout becomes context (or rejection) per harness convention.

**Use MCP when** the capability requires external state (DB, API, filesystem outside the project). MCP servers expose tools to the model via the Model Context Protocol. Pure-skill plugins should not bundle MCP — keep MCP plugins standalone so installers can opt out.

**Use agents/ when** the subagent is genuinely reusable across many contexts without curated input. Most workflows do not qualify; mature plugins prefer prompt templates inside skills (`skills/<name>/<role>-prompt.md`) because:

1. Templates force the controller to curate context per use (placeholders are explicit).
2. Templates can encode status enums (`DONE | DONE_WITH_CONCERNS | BLOCKED | NEEDS_CONTEXT`) for programmatic routing.
3. Templates are versioned with the skill that uses them (no drift).
4. Templates work in any harness with a Task tool (registered agents need harness-specific support).

**Anti-patterns:**
- **`agents/` directory** for workflow-specific roles → wrong; use prompt templates with placeholders inside the relevant skill folder
- **`commands/` directory** for slash commands → deprecated; put slash commands in `skills/<name>/SKILL.md` (same format, more flexible)
- **`.mcp.json`** when not needed → adds dependency surface for nothing
- **Single mega-skill** doing everything → prefer multiple focused skills wired via `**REQUIRED SUB-SKILL:**`

## Six File Types in a Skill Folder

A skill folder may contain these file types. Only `SKILL.md` is required; the other five are added as needed and are observed across the 14 skills of the `superpowers` plugin (the empirical reference for mature plugin design).

| Type | Pattern | Loaded when | Cost |
|---|---|---|---|
| **SKILL.md** | `SKILL.md` | Frontmatter always at session start; body on activation | Frontmatter ≤ 1024 chars |
| **Lazy reference** | `<topic>.md` referenced as prose ("see foo.md") | Model decides to read at activation | Pay-per-read |
| **Force-load reference** | `@<topic>.md` in SKILL.md body | Immediately when SKILL.md is read | 200k+ context preemptively |
| **Prompt template** | `<role>-prompt.md` with `{PLACEHOLDER}` tokens | Controller fills and dispatches via Task tool | Per-dispatch only |
| **Example** | `example.<ext>` (one excellent example, not multi-language) | Adapted by reader on demand | Pay-per-read |
| **Script** | `*.sh`, `*.js`, `*.py` | Executed via Bash; only stdout consumes tokens | Stdout size only |

Optional: `references/<harness>-tools.md` for cross-harness tool name mappings (consumed when the active harness is non-Claude-Code). `test-*.md` and `CREATION-LOG.md` files document the skill's pressure-testing history; not consumed at runtime but kept for provenance.

**Rule of thumb:** Every additional file in the skill folder is a tax on the reader. Inline content in SKILL.md if it fits under ~500 lines; split out only when clearly heavy (API references, executable tools, harness adapters).

**File naming:** Use lowercase with hyphens, not underscores. Drop the `.md` extension only for executable hook scripts (e.g., `session-start`, not `session-start.sh` — Claude Code's Windows auto-detection prepends `bash` to any `.sh` filename, which interferes with the polyglot wrapper). Templates use `.tmpl` suffix to distinguish them from real files.

## Multi-harness Manifests

A serious plugin ships parallel manifests for each supported harness. Each manifest declares the same plugin (same skills, same hooks scripts) but in the format the host harness expects. Adding a harness is largely additive — the existing manifests are untouched, you add a new directory or a new top-level file.

| Harness | Manifest path | Notes |
|---|---|---|
| Claude Code | `.claude-plugin/plugin.json` | Default; installed via `/plugin install` |
| Cursor | `.cursor-plugin/plugin.json` + `hooks/hooks-cursor.json` | Hook output is `additional_context` (snake_case) |
| Codex (CLI/App) | `.codex-plugin/plugin.json` | Includes `interface` block (`displayName`, `brandColor`, icons) |
| OpenCode | `.opencode/plugins/<name>.js` + `package.json` | ESM module; install via `git+https://...` in `opencode.json` |
| Gemini CLI | `gemini-extension.json` (root) | Declares `geminiMdFiles` and `skills` path |
| Factory Droid / Copilot CLI | reuses `.claude-plugin/` | Marketplace-specific install |

Hook output JSON differs per harness — see `plugin-bootstrap.md` for harness detection (via env vars `CURSOR_PLUGIN_ROOT`, `CLAUDE_PLUGIN_ROOT`, `COPILOT_CLI`) and the three output shapes. Tool vocabulary also differs: skills are written in Claude Code vocabulary (`TodoWrite`, `Task`, `Skill`); other harnesses need translation via `references/<harness>-tools.md`.

Use a single source of truth for version numbers (e.g., `.version-bump.json`) so all manifests stay in sync — multi-harness authors typically have a script that updates `package.json`, all `plugin.json` files, `marketplace.json`, and `gemini-extension.json` from a single source.

## Common Plugin Shapes

Three shapes capture the majority of mature plugin designs. Choose the simplest shape that fits — every added component (hooks, multiple skills, pipeline) is a maintenance tax.

**Skill-only.** Single skill, no hooks, no MCP. The skill is opt-in; the user must invoke it via the Skill tool or it stays dormant. Lowest cost (no permanent token overhead), simplest to maintain (no harness-specific hook output). Best for capabilities that the model can be trusted to invoke when relevant — typically because the description trigger conditions are highly specific. Example: a code-style guide skill that runs only when explicitly invoked.

**With-bootstrap.** Single skill plus a `SessionStart` hook that injects the SKILL.md eager into the system prompt. Costs ~600 tokens per session permanently. Justified when the skill must activate without explicit invocation — typically discipline-enforcing skills the user shouldn't have to remember (e.g., an orientation skill that ensures the model picks the right pipeline before doing anything). Multi-harness bootstrap requires a polyglot wrapper (`run-hook.cmd`) and per-harness JSON output detection (see `plugin-bootstrap.md`).

**Multi-skill-with-pipeline.** Multiple skills wired together via `**REQUIRED SUB-SKILL:**` markers, optionally with bootstrap. Example: `superpowers` itself — 14 skills chained into a brainstorm → plan → execute → finish pipeline, bootstrapped via `using-superpowers`. Plan a clear cross-skill flow before committing to this shape; pipelined skills are harder to test in isolation and must agree on shared assumptions. See `plugin-wiring.md` for assembly patterns.

## The 8 Clarifying Questions

Stage 1 brainstorming must inject these 8 questions. Each answer drives concrete file/component decisions later in the pipeline. Ask them in order — each subsequent answer depends on prior ones (you cannot pick a bootstrap SKILL.md before naming skills; you cannot pick a marketplace shape before the plugin name exists).

The brainstorming skill should pause after each answer, confirm understanding, and probe edge cases before moving on. Resist the urge to batch questions to "save time" — the answers compound and a wrong early answer creates rework.

### 1. Plugin name

**Question:** What is the plugin name? (lowercase-with-hyphens, no underscores, no special characters; ≤ 64 chars per the official frontmatter limit)

**Example answer:** `plugin-builder`

**Implication:** Drives the value of `name` in every manifest, the directory name, the namespace prefix in cross-skill references (`plugin-builder:creating-a-plugin`), and the install command (`/plugin install plugin-builder@<marketplace>`). Renaming after release is painful — the install command changes for every user.

### 2. One-sentence purpose

**Question:** In one sentence, what does this plugin do? Avoid passive voice; lead with the verb.

**Example answer:** "Build Claude Code plugins using the canonical superpowers pipeline."

**Implication:** This sentence becomes the `description` field in every manifest, the README tagline, and the seed for the `description` field in any SKILL.md frontmatter (which must be reduced to triggers-only per CSO — manifests get the full sentence; SKILL.md descriptions get only "Use when..." conditions). If you cannot state the plugin's purpose in one sentence, brainstorming has not converged yet.

### 3. Components

**Question:** Which components does the plugin include? Skill / +hooks / +MCP / +agents

**Example answer:** "Skill + hooks (SessionStart bootstrap)"

**Implication:** Determines which directories exist (`skills/`, `hooks/`, `.mcp.json`, `agents/`), which manifests declare which features, and the size of the build plan. Each `+X` adds a measurable amount of scaffolding:
- `+hooks` adds 4 files (hooks.json, hooks-cursor.json, run-hook.cmd, the hook script) and per-harness JSON output logic
- `+MCP` adds an `.mcp.json` and external server dependencies
- `+agents` adds `agents/<name>.md` files and harness-specific subagent registration (rarely the right choice)

### 4. Harnesses

**Question:** Which harnesses must be supported? Claude Code only / + Cursor / + Codex / + OpenCode / + Gemini

**Example answer:** "All five (Claude Code, Cursor, Codex, OpenCode, Gemini CLI)"

**Implication:** Each declared harness adds a manifest file and (if hooks are declared) per-harness hook output detection. Multi-harness is the default for serious plugins because users install where they work. Dropping harnesses later is fine; adding them later means reopening Stage 1 because hook output formats and tool vocabulary differ across harnesses.

### 5. Bootstrap pattern

**Question:** Does the plugin need an eager SessionStart bootstrap? If yes, which SKILL.md is injected?

**Example answer:** "Yes — inject `creating-a-plugin/SKILL.md` so the orientation is present without explicit invocation."

**Implication:** If yes, adds `hooks/hooks.json`, `hooks/hooks-cursor.json`, `hooks/run-hook.cmd`, `hooks/session-start`, and ~600 tokens permanent per-session cost. The bootstrap fires on `startup|clear|compact` matchers — meaning the SKILL.md is reinjected after every `/clear` and after compaction. If no, the skill is opt-in only — much lower cost, but the user (or model) must remember to invoke. **Justification matters:** bootstrap is justified for orientation/discipline skills the model would otherwise miss; not justified for narrow technical skills the description triggers cleanly.

### 6. Skills planned

**Question:** List each planned skill: name, purpose (one sentence), trigger condition (when does it activate?).

**Example answer:**
- `creating-a-plugin` — orientation skill that runs the canonical pipeline. Trigger: user asks to create/build/scaffold a plugin.

**Implication:** Each skill becomes one Stage-3 brainstorm during execution (Level 2 brainstorming). The list drives the `skills/` directory layout. Triggers go straight into each SKILL.md `description` field (CSO rule: triggers-only, no workflow summary — descriptions that summarize workflow create shortcuts the model takes instead of reading SKILL.md).

### 7. Cross-skill pipeline

**Question:** Are the skills wired into a pipeline via `**REQUIRED SUB-SKILL:**` markers? If yes, what is the chain? Or are they independent?

**Example answer:** "Independent — single-skill plugin, no internal pipeline. Cross-plugin REQUIRED markers point to superpowers skills (e.g., `superpowers:brainstorming`, `superpowers:writing-plans`)."

**Implication:** Determines the wiring task in the plan. Pipelined skills need explicit ordering, shared assumptions, and per-step verification (you must test each junction). Independent skills only need namespace conventions and trigger isolation. Cross-plugin REQUIRED markers (referencing skills in another installed plugin) make the plugin a hard dependent of that plugin — declare the dependency in the plugin's manifest description and check for it at activation.

### 8. Marketplace

**Question:** How is the plugin distributed? Standalone (single-plugin repo) / private marketplace (repo lists multiple plugins) / public marketplace (community)

**Example answer:** "Private marketplace — `lbn-plugins/.claude-plugin/marketplace.json` lists this plugin alongside future ones."

**Implication:** Drives whether `marketplace.json` is needed and where:
- **Standalone** — no marketplace.json; install via `/plugin marketplace add <user>/<repo>` then `/plugin install <plugin-name>@<repo-name>`. Simplest. Does not scale to multi-plugin authors.
- **Private marketplace** — `marketplace.json` at parent repo root; lists multiple plugins. Author manages a small ecosystem.
- **Public marketplace** — published to a community registry; requires additional metadata (sponsorship, community guidelines, possibly review process). Consider only after the plugin has stabilized.

## Outputs of Stage 1

After answering the 8 questions, Stage 1 brainstorming produces a spec at `docs/superpowers/specs/<date>-<plugin-name>-design.md` covering:

- Goal and context (what problem the plugin solves)
- Architecture (orientation pattern, two-level brainstorming if multi-skill, hard dependencies)
- File structure (directory layout based on declared components and harnesses)
- Skill content stubs (one stub per planned skill: name, purpose, trigger condition)
- Multi-harness manifests (concrete content sketches per declared harness)
- Hooks (if bootstrap declared) — file list and JSON output detection logic
- Documentation (CLAUDE.md, AGENTS.md symlink, GEMINI.md, README)
- Evals (pressure scenarios for each discipline-enforcing skill)
- Decisions table (locked-in choices that the plan must honor)
- Phase 1 vs Phase 2 scope (YAGNI ruthlessly — defer everything that can be deferred)
- Acceptance criteria (how the plugin is verified done)
- Open issues (unresolved questions blocking plan creation)

This spec is the input to Stage 2 (`superpowers:writing-plans`). Without it, the plan has no source of truth and drifts. With it, the plan is mechanical task expansion of the spec sections.
