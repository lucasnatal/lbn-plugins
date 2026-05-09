# Plugin-Builder Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build the `plugin-builder` plugin per spec at `docs/superpowers/specs/2026-05-09-plugin-builder-design.md`.

**Architecture:** Single skill (`creating-a-plugin`) with 5 reference docs and 8 templates. Bootstrap eager via SessionStart hook. Multi-harness manifests for 5 harnesses. Evals/ with 4 pressure scenarios.

**Tech Stack:** Markdown (skill content + docs), JSON (manifests + hooks config), Bash (hooks scripts), JavaScript ESM (OpenCode entry).

---

## Working Directory

This plan must be executed inside an isolated worktree. Before Task A1, the executor MUST invoke `superpowers:using-git-worktrees` to create:

```
/Users/lucasnatal/Developer/Plugins/lbn-plugins/.worktrees/plugin-builder/
```

Branch: `feat/plugin-builder`. Plugin built at the worktree root under `plugin-builder/`. After Stage 5 (finishing), merge to `main` produces `lbn-plugins/plugin-builder/`.

---

## Spec Reference

The spec at `docs/superpowers/specs/2026-05-09-plugin-builder-design.md` is the source of truth for content structure. Each task below cites the relevant spec section. Read the cited spec section before implementing.

**Read also for templates and patterns:**
- `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/session-start` — bash pattern to adapt
- `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/run-hook.cmd` — polyglot pattern to adapt
- `/Users/lucasnatal/Developer/Plugins/superpowers/.claude-plugin/plugin.json` — manifest pattern
- `/Users/lucasnatal/Developer/Plugins/superpowers/skills/using-superpowers/SKILL.md` — bootstrap skill pattern
- `/Users/lucasnatal/Developer/Plugins/superpowers/skills/writing-skills/SKILL.md` — discipline-skill pattern

---

## Parallelization Notes

Tasks within a phase are typically sequential (each commit builds on the previous). Phases B (skill content), C (templates), and G (evals) contain independent files — if `superpowers:dispatching-parallel-agents` is available, those phases can dispatch parallel subagents safely.

---

# Phase A — Skeleton & Foundation

### Task A1: Create directory structure

**Files:** all directories under `plugin-builder/`

- [ ] **Step 1: Create directory tree**

```bash
cd <worktree-root>
mkdir -p plugin-builder/.claude-plugin
mkdir -p plugin-builder/.codex-plugin
mkdir -p plugin-builder/.cursor-plugin
mkdir -p plugin-builder/.opencode/plugins
mkdir -p plugin-builder/hooks
mkdir -p plugin-builder/skills/creating-a-plugin/templates
mkdir -p plugin-builder/evals/pipeline-discipline
mkdir -p plugin-builder/assets
```

- [ ] **Step 2: Verify**

```bash
find plugin-builder -type d | sort
```

Expected: 13 directories under `plugin-builder/`.

- [ ] **Step 3: Commit**

```bash
git add -A
git commit -m "scaffold: plugin-builder directory structure"
```

---

### Task A2: Write LICENSE

**Files:** Create `plugin-builder/LICENSE`

- [ ] **Step 1: Write MIT license file**

```
MIT License

Copyright (c) 2026 Lucas Bonetti Natal

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

- [ ] **Step 2: Verify**

```bash
head -1 plugin-builder/LICENSE
```

Expected: `MIT License`

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/LICENSE
git commit -m "docs: MIT license"
```

---

### Task A3: Write `.gitignore`

**Files:** Create `plugin-builder/.gitignore`

- [ ] **Step 1: Write gitignore**

```
node_modules/
.DS_Store
*.log
```

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/.gitignore
git commit -m "chore: gitignore"
```

---

### Task A4: Write README.md placeholder

**Files:** Create `plugin-builder/README.md`

This is a stub. Full README is written in Task F4 once everything else exists.

- [ ] **Step 1: Write minimal stub**

```markdown
# plugin-builder

Build Claude Code plugins using the canonical superpowers pipeline.

This README will be expanded in a later task. See `docs/superpowers/specs/2026-05-09-plugin-builder-design.md` for the design.
```

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/README.md
git commit -m "docs: README stub"
```

---

# Phase B — Core Skill (creating-a-plugin)

This phase produces 1 SKILL.md and 5 reference docs. Each is structured per spec section 6. Subagent must read the cited spec section before writing.

### Task B1: Write `creating-a-plugin/SKILL.md`

**Files:** Create `plugin-builder/skills/creating-a-plugin/SKILL.md`

**Spec reference:** Section 6.1 (lists every required section in order, including literal Iron Law text).

**Pattern reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/skills/using-superpowers/SKILL.md` for bootstrap-style skill patterns (`<SUBAGENT-STOP>` tag, `<EXTREMELY-IMPORTANT>` tag, Red Flags table format).

- [ ] **Step 1: Write frontmatter and opening tags**

The frontmatter `description` field MUST be triggers only — NO workflow summary (per CSO rule, spec section 6.1 and PLUGINS.md §VII.1):

```yaml
---
name: creating-a-plugin
description: Use when your human partner asks to create, build, or scaffold a new Claude Code plugin (or compatible harness plugin). Trigger phrases include "create a plugin", "build a plugin", "make a plugin to do X", or any request to wrap functionality into a redistributable plugin format.
---

<SUBAGENT-STOP>
If you were dispatched as a subagent to execute a specific task, skip this skill.
</SUBAGENT-STOP>

<EXTREMELY-IMPORTANT>
When your human partner asks to create, build, or scaffold a plugin, you ABSOLUTELY MUST follow the canonical superpowers pipeline below. This is not negotiable. You cannot rationalize your way out of this.
</EXTREMELY-IMPORTANT>

# Creating a Plugin
```

- [ ] **Step 2: Write Overview, Iron Law, Hard Dependency Check, REQUIRED BACKGROUND sections**

Per spec 6.1, sections 3-6 in this order. The Iron Law text MUST be exactly:

```
NO PLUGIN BUILT WITHOUT FOLLOWING THE FULL SUPERPOWERS PIPELINE
```

The "Hard Dependency Check" section instructs the model to verify `superpowers:brainstorming` is available; if not, abort with a specific message recommending `/plugin install superpowers@superpowers-marketplace`.

The "REQUIRED BACKGROUND" section instructs reading `plugin-anatomy.md` before invoking brainstorming.

- [ ] **Step 3: Write the Pipeline section with graphviz diagram**

A graphviz `digraph` showing Stage 1 → Stage 1.5 → Stage 2 → Stage 4 (with Stage 3 nested) → Stage 5, with REQUIRED SUB-SKILL labels on edges.

- [ ] **Step 4: Write Stage 1 through Stage 5 sections**

Per spec 6.1 sections 8-12. Each Stage section:
- Names the REQUIRED SUB-SKILL (`superpowers:brainstorming`, `superpowers:writing-plans`, etc.)
- Lists what reference docs to consult during that stage
- Specifies output / handoff to next stage

Stage 1 must list the 8 plugin-level clarifying questions (per spec section 4.2 / plugin-anatomy.md content).

- [ ] **Step 5: Write Red Flags and Common Rationalizations sections**

Per spec 6.1 sections 13-14. The Red Flags list MUST include at minimum:
- Skipping Stage 1 ("I know what I'm building")
- Single brainstorm trying to design plugin AND every skill at once
- Implementing skills before scaffolding manifests
- Adding bootstrap hook without it being in the spec
- "Just a small plugin" → still goes through full pipeline
- Using `agents/` directory → wrong pattern; use prompt templates
- Using `commands/` directory → deprecated

The Common Rationalizations table MUST include the 5 rows from spec section 6.1.

- [ ] **Step 6: Write Reference Files section**

Lists the 5 lazy-loaded refs and the 8 templates with one-line descriptions each.

- [ ] **Step 7: Verify**

```bash
wc -l plugin-builder/skills/creating-a-plugin/SKILL.md
grep -c "^## " plugin-builder/skills/creating-a-plugin/SKILL.md
grep "Iron Law" plugin-builder/skills/creating-a-plugin/SKILL.md
grep "REQUIRED SUB-SKILL" plugin-builder/skills/creating-a-plugin/SKILL.md
```

Expected: 130-180 lines, 12-16 H2 sections, "Iron Law" present, multiple "REQUIRED SUB-SKILL" markers.

- [ ] **Step 8: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/SKILL.md
git commit -m "feat(skill): creating-a-plugin SKILL.md"
```

---

### Task B2: Write `plugin-anatomy.md`

**Files:** Create `plugin-builder/skills/creating-a-plugin/plugin-anatomy.md`

**Spec reference:** Section 6.2 first subsection (~200 lines target).

**Source for content:** `Plugins/PLUGINS.md` — destilation focused on plugin-builder use case.

- [ ] **Step 1: Write file header and overview**

H1 title. Brief intro stating: "Reference consulted by Stage 1 brainstorming. Provides the plugin-level questions and shape of a plugin."

- [ ] **Step 2: Write "What a Plugin Is" section**

Vector of capabilities (skills, hooks, MCP, agents). Table from PLUGINS.md Part I.

- [ ] **Step 3: Write "Components Catalog" section**

When to use skills, hooks, MCP, agents. Anti-patterns to avoid (`agents/` redundante, `commands/` legado).

- [ ] **Step 4: Write "Six File Types in a Skill" section**

The catalog from PLUGINS.md Part III: SKILL.md, lazy refs, force-load refs, prompt templates, examples, scripts.

- [ ] **Step 5: Write "Multi-harness Manifests" section**

Table mapping each harness to its manifest path and required fields.

- [ ] **Step 6: Write "Common Plugin Shapes" section**

Three example shapes: skill-only, with-bootstrap, multi-skill-with-pipeline. One paragraph each.

- [ ] **Step 7: Write "The 8 Clarifying Questions" section** (CRITICAL)

The exact 8 questions Stage 1 brainstorming must inject. Each with:
- The question text
- Example answer
- Implication (what files/components this answer drives)

Questions per spec section 6.1 / 4.2:
1. Plugin name (lowercase-with-hyphens)
2. One-sentence purpose
3. Components (skill / +hooks / +MCP / +agents)
4. Harnesses (Claude Code only / + Cursor / + Codex / + OpenCode / + Gemini)
5. Bootstrap pattern (yes/no — eager SessionStart of which skill?)
6. Skills planned (list with name + purpose + trigger)
7. Cross-skill pipeline (REQUIRED chain between them?)
8. Marketplace (standalone / private / public?)

- [ ] **Step 8: Verify**

```bash
wc -l plugin-builder/skills/creating-a-plugin/plugin-anatomy.md
grep -c "^## " plugin-builder/skills/creating-a-plugin/plugin-anatomy.md
grep -c "^### " plugin-builder/skills/creating-a-plugin/plugin-anatomy.md
```

Expected: 180-230 lines, 6-8 H2 sections, 8 H3 (one per clarifying question).

- [ ] **Step 9: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/plugin-anatomy.md
git commit -m "feat(skill): plugin-anatomy reference doc"
```

---

### Task B3: Write `plugin-scaffolding.md`

**Files:** Create `plugin-builder/skills/creating-a-plugin/plugin-scaffolding.md`

**Spec reference:** Section 6.2.2 (~150 lines target).

- [ ] **Step 1: Write header and intent**

"Recipe for generating plugin physical structure. Consulted by writing-plans during plan creation, and by subagents during scaffolding tasks."

- [ ] **Step 2: Write "Standard Root Layout" section**

The complete directory tree pattern observed in mature plugins.

- [ ] **Step 3: Write "Per-Harness Manifest Paths" section**

Table: harness name | manifest path | required fields | optional fields | reference template path.

5 rows: Claude Code, Codex, Cursor, OpenCode, Gemini.

- [ ] **Step 4: Write "File Creation Order" section**

Explicit order with rationale: manifests first → skills → hooks (if any) → README → `.gitignore` last.

- [ ] **Step 5: Write "Path Conventions" section**

- Lowercase, hyphens-not-underscores
- Forward slashes always (no backslashes even on Windows)
- `skills/<name>/SKILL.md` always (never `commands/<name>.md`)

- [ ] **Step 6: Write "Auxiliary Files" section**

- `package.json` for OpenCode (when needed)
- `AGENTS.md → CLAUDE.md` symlink command
- `.version-bump.json` (optional)

- [ ] **Step 7: Write "Templates Available" section**

List of templates in `templates/` and which task uses each.

- [ ] **Step 8: Verify**

```bash
wc -l plugin-builder/skills/creating-a-plugin/plugin-scaffolding.md
grep -c "^## " plugin-builder/skills/creating-a-plugin/plugin-scaffolding.md
```

Expected: 130-170 lines, 6-8 H2 sections.

- [ ] **Step 9: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/plugin-scaffolding.md
git commit -m "feat(skill): plugin-scaffolding reference doc"
```

---

### Task B4: Write `plugin-wiring.md`

**Files:** Create `plugin-builder/skills/creating-a-plugin/plugin-wiring.md`

**Spec reference:** Section 6.2.3 (~120 lines target).

- [ ] **Step 1: Write header and intent**

"Connection manual for skills within a plugin and across plugins."

- [ ] **Step 2: Write "REQUIRED SUB-SKILL Pattern" section**

Syntax, position (end of skill), when to use. Examples.

- [ ] **Step 3: Write "REQUIRED BACKGROUND Pattern" section**

Syntax, position (start of skill), when to use. Examples.

- [ ] **Step 4: Write "Related Skills Pattern" section**

For informative cross-references (no obligation).

- [ ] **Step 5: Write "Namespace Convention" section**

`<plugin>:<skill-name>` always with prefix. Why namespacing matters.

- [ ] **Step 6: Write "@file Rule" section**

Use sparingly. Costs 200k+ context preemptively. When acceptable (true on-every-execution dependency) vs when avoid (most cases).

- [ ] **Step 7: Write "Pipeline Assembly" section**

How to chain skills via REQUIRED SUB-SKILL to form a workflow.

- [ ] **Step 8: Write "Cross-Plugin References" section**

How to reference superpowers skills from this plugin (e.g., `superpowers:brainstorming`).

- [ ] **Step 9: Verify**

```bash
wc -l plugin-builder/skills/creating-a-plugin/plugin-wiring.md
grep -c "^## " plugin-builder/skills/creating-a-plugin/plugin-wiring.md
```

Expected: 100-140 lines, 7-8 H2 sections.

- [ ] **Step 10: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/plugin-wiring.md
git commit -m "feat(skill): plugin-wiring reference doc"
```

---

### Task B5: Write `plugin-bootstrap.md`

**Files:** Create `plugin-builder/skills/creating-a-plugin/plugin-bootstrap.md`

**Spec reference:** Section 6.2.4 (~150 lines target).

**Pattern reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/session-start` and `superpowers/hooks/run-hook.cmd`.

- [ ] **Step 1: Write header and pattern explanation**

Eager vs lazy default. Why bootstrap exists. Cost (~600 tokens permanent).

- [ ] **Step 2: Write "When to Use Bootstrap" section**

Checklist of justifications for spending tokens permanently.

- [ ] **Step 3: Write "hooks.json Structure" section**

Format with matcher, async, command. Example:

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

- [ ] **Step 4: Write "Polyglot Wrapper" section**

Pattern for `run-hook.cmd` serving as bash on Unix and batch on Windows. Show the `: << 'CMDBLOCK'` trick.

- [ ] **Step 5: Write "Session-Start Script" section**

Read SKILL.md → escape JSON → detect harness → emit JSON in correct format. Reference superpowers' session-start as canonical pattern.

- [ ] **Step 6: Write "Output JSON per Harness" section**

Table:
- Cursor: `additional_context` (top-level, snake_case)
- Claude Code: `hookSpecificOutput.additionalContext` (nested, camelCase)
- Copilot CLI / SDK standard: `additionalContext` (top-level, camelCase)

Detection via env vars: `CURSOR_PLUGIN_ROOT`, `CLAUDE_PLUGIN_ROOT`, `COPILOT_CLI`.

- [ ] **Step 7: Write "Variable Injection" section**

`${CLAUDE_PLUGIN_ROOT}` etc. — what each holds, when each is set.

- [ ] **Step 8: Write "Cost Analysis" section**

How to count: word-count the SKILL.md being injected. Token estimate ~1.3× word count. Per-session permanent cost.

- [ ] **Step 9: Verify**

```bash
wc -l plugin-builder/skills/creating-a-plugin/plugin-bootstrap.md
grep -c "^## " plugin-builder/skills/creating-a-plugin/plugin-bootstrap.md
```

Expected: 130-170 lines, 7-8 H2 sections.

- [ ] **Step 10: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/plugin-bootstrap.md
git commit -m "feat(skill): plugin-bootstrap reference doc"
```

---

### Task B6: Write `plugin-distribution.md`

**Files:** Create `plugin-builder/skills/creating-a-plugin/plugin-distribution.md`

**Spec reference:** Section 6.2.5 (~80 lines target).

- [ ] **Step 1: Write header and intent**

"Marketplace setup. Consulted in finishing stage when user chooses Option 2 (PR)."

- [ ] **Step 2: Write "marketplace.json Structure" section**

Required fields (`name`, `owner`, `plugins[]`). Example:

```json
{
  "name": "my-marketplace",
  "owner": { "name": "...", "email": "..." },
  "plugins": [
    {
      "name": "my-plugin",
      "description": "...",
      "version": "0.1.0",
      "source": "./my-plugin"
    }
  ]
}
```

- [ ] **Step 3: Write "Marketplace Modes" section**

Standalone (single plugin = single repo) vs private marketplace (one repo, multiple plugins) vs public.

- [ ] **Step 4: Write "Install Path Semantics" section**

`/plugin marketplace add <user>/<repo>` then `/plugin install <plugin>@<marketplace>`.

- [ ] **Step 5: Write "Version Bumping" section**

Why `.version-bump.json` exists. When to use it.

- [ ] **Step 6: Write "README Requirements" section**

What public README must contain (Quickstart, Installation per harness, What's Inside, License).

- [ ] **Step 7: Verify**

```bash
wc -l plugin-builder/skills/creating-a-plugin/plugin-distribution.md
```

Expected: 70-100 lines.

- [ ] **Step 8: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/plugin-distribution.md
git commit -m "feat(skill): plugin-distribution reference doc"
```

---

# Phase C — Templates

8 template files with placeholders consumed by future plugin builds. Placeholders use `{{NAME}}` style.

### Task C1: Write 5 manifest templates

**Files:**
- Create: `plugin-builder/skills/creating-a-plugin/templates/claude-plugin.json.tmpl`
- Create: `plugin-builder/skills/creating-a-plugin/templates/codex-plugin.json.tmpl`
- Create: `plugin-builder/skills/creating-a-plugin/templates/cursor-plugin.json.tmpl`
- Create: `plugin-builder/skills/creating-a-plugin/templates/gemini-extension.json.tmpl`
- Create: `plugin-builder/skills/creating-a-plugin/templates/opencode-plugin.js.tmpl`

- [ ] **Step 1: Write `claude-plugin.json.tmpl`**

```json
{
  "name": "{{NAME}}",
  "description": "{{DESCRIPTION}}",
  "version": "{{VERSION}}",
  "author": {
    "name": "{{AUTHOR_NAME}}",
    "email": "{{AUTHOR_EMAIL}}"
  },
  "homepage": "{{HOMEPAGE}}",
  "repository": "{{REPOSITORY}}",
  "license": "{{LICENSE}}",
  "keywords": [{{KEYWORDS_JSON_ARRAY}}]
}
```

- [ ] **Step 2: Write `codex-plugin.json.tmpl`**

Adapt from `superpowers/.codex-plugin/plugin.json`. Includes full `interface` block with `displayName`, `category`, `capabilities`, `defaultPrompt[]`, `brandColor`, `composerIcon`, `logo`. Keep all interface fields with placeholders.

- [ ] **Step 3: Write `cursor-plugin.json.tmpl`**

Adapt from `superpowers/.cursor-plugin/plugin.json`. Include `skills: "./skills/"`. If hooks declared (`{{HAS_HOOKS}}`), include `hooks: "./hooks/hooks-cursor.json"`. Otherwise omit hooks field.

- [ ] **Step 4: Write `gemini-extension.json.tmpl`**

```json
{
  "name": "{{NAME}}",
  "version": "{{VERSION}}",
  "description": "{{DESCRIPTION}}",
  "geminiMdFiles": ["GEMINI.md"],
  "skills": "./skills/"
}
```

- [ ] **Step 5: Write `opencode-plugin.js.tmpl`**

Adapt from `/Users/lucasnatal/Developer/Plugins/superpowers/.opencode/plugins/superpowers.js`. Read that file first to understand OpenCode entry-point pattern. Replace plugin-specific identifiers with placeholders.

- [ ] **Step 6: Verify all 5 templates parse with placeholders intact**

```bash
ls plugin-builder/skills/creating-a-plugin/templates/*.tmpl | wc -l
```

Expected: 5 (or more if hooks templates already done — but those are Task C2).

- [ ] **Step 7: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/templates/
git commit -m "feat(templates): manifest templates for 5 harnesses"
```

---

### Task C2: Write 2 hook templates

**Files:**
- Create: `plugin-builder/skills/creating-a-plugin/templates/hooks-polyglot.cmd.tmpl`
- Create: `plugin-builder/skills/creating-a-plugin/templates/session-start-hook.tmpl`

- [ ] **Step 1: Write `hooks-polyglot.cmd.tmpl`**

Adapt from `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/run-hook.cmd`. Read that file first. Generic enough to serve any plugin's `hooks/` directory. No plugin-specific placeholders needed (path resolution is via `%~dp0`/`$0`).

- [ ] **Step 2: Write `session-start-hook.tmpl`**

Adapt from `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/session-start`. Read that file first. Replace:
- `using_superpowers_content` → `{{SKILL_FILE_PATH}}` placeholder for the SKILL.md being injected
- "You have superpowers" message → `{{ACTIVATION_MESSAGE}}` placeholder
- Legacy directory check → `{{LEGACY_CHECK_BLOCK}}` placeholder (optional, may be empty)
- `superpowers:using-superpowers` → `{{PLUGIN_NAME}}:{{SKILL_NAME}}` placeholders

Keep the harness detection (`CURSOR_PLUGIN_ROOT` / `CLAUDE_PLUGIN_ROOT` / `COPILOT_CLI`) and JSON output blocks intact — those are not plugin-specific.

- [ ] **Step 3: Verify**

```bash
ls plugin-builder/skills/creating-a-plugin/templates/*.tmpl
```

Expected: 7 templates total now (5 manifests + 2 hooks).

- [ ] **Step 4: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/templates/
git commit -m "feat(templates): hooks templates (polyglot wrapper + session-start)"
```

---

### Task C3: Write README template

**Files:** Create `plugin-builder/skills/creating-a-plugin/templates/README.md.tmpl`

- [ ] **Step 1: Write README template**

Structure adapted from `/Users/lucasnatal/Developer/Plugins/superpowers/README.md`. Read it first. Sections to include with placeholders:

```markdown
# {{NAME}}

{{ONE_LINE_DESCRIPTION}}

## How it works

{{HOW_IT_WORKS_PARAGRAPH}}

## Installation

### Claude Code

```bash
/plugin install {{NAME}}@{{MARKETPLACE}}
```

### Cursor
{{CURSOR_INSTALL_INSTRUCTIONS}}

### Codex
{{CODEX_INSTALL_INSTRUCTIONS}}

### OpenCode
{{OPENCODE_INSTALL_INSTRUCTIONS}}

### Gemini CLI
{{GEMINI_INSTALL_INSTRUCTIONS}}

## What's Inside

{{SKILLS_LIST}}

## Philosophy

{{PHILOSOPHY_PARAGRAPH}}

## License

{{LICENSE}} — see LICENSE file
```

- [ ] **Step 2: Verify**

```bash
ls plugin-builder/skills/creating-a-plugin/templates/*.tmpl | wc -l
```

Expected: 8 templates total.

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/skills/creating-a-plugin/templates/README.md.tmpl
git commit -m "feat(templates): README.md template"
```

---

# Phase D — Hooks (the actual files for plugin-builder itself)

These are the hooks that make plugin-builder's bootstrap work.

### Task D1: Write `hooks/hooks.json`

**Files:** Create `plugin-builder/hooks/hooks.json`

- [ ] **Step 1: Write hooks.json**

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

- [ ] **Step 2: Verify JSON parses**

```bash
python3 -m json.tool plugin-builder/hooks/hooks.json > /dev/null && echo OK
```

Expected: `OK`.

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/hooks/hooks.json
git commit -m "feat(hooks): SessionStart hook configuration"
```

---

### Task D2: Write `hooks/hooks-cursor.json`

**Files:** Create `plugin-builder/hooks/hooks-cursor.json`

**Reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/hooks-cursor.json`. Read it first to understand Cursor's expected format (likely identical or near-identical to Claude Code's; Cursor uses different env var detection in the script).

- [ ] **Step 1: Read superpowers' cursor variant**

```bash
cat /Users/lucasnatal/Developer/Plugins/superpowers/hooks/hooks-cursor.json
```

- [ ] **Step 2: Write hooks-cursor.json**

Adapt the structure for plugin-builder (mostly: same as `hooks.json` if Cursor uses the same format; differences captured in the `session-start` script's harness detection).

- [ ] **Step 3: Verify JSON parses**

```bash
python3 -m json.tool plugin-builder/hooks/hooks-cursor.json > /dev/null && echo OK
```

- [ ] **Step 4: Commit**

```bash
git add plugin-builder/hooks/hooks-cursor.json
git commit -m "feat(hooks): Cursor variant hook configuration"
```

---

### Task D3: Write `hooks/run-hook.cmd`

**Files:** Create `plugin-builder/hooks/run-hook.cmd`

**Reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/run-hook.cmd`. The polyglot pattern is generic — copy structure, only paths differ (which are resolved at runtime via `%~dp0`/`$0`, so no edits needed).

- [ ] **Step 1: Read superpowers' polyglot wrapper**

```bash
cat /Users/lucasnatal/Developer/Plugins/superpowers/hooks/run-hook.cmd
```

- [ ] **Step 2: Copy the file structure to plugin-builder/hooks/run-hook.cmd**

The wrapper is plugin-agnostic (resolves paths relative to itself). Direct copy is correct.

- [ ] **Step 3: Verify bash syntax**

```bash
bash -n plugin-builder/hooks/run-hook.cmd
```

Expected: no output (syntax OK).

- [ ] **Step 4: Commit**

```bash
git add plugin-builder/hooks/run-hook.cmd
git commit -m "feat(hooks): polyglot wrapper (bash + batch)"
```

---

### Task D4: Write `hooks/session-start`

**Files:** Create `plugin-builder/hooks/session-start`

**Reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/hooks/session-start`. Adapt with these specific changes:

1. The skill being read: `creating-a-plugin/SKILL.md` (not `using-superpowers/SKILL.md`)
2. The activation message: `"You have plugin-builder installed."` (not `"You have superpowers."`)
3. The skill identifier in the message: `plugin-builder:creating-a-plugin` (not `superpowers:using-superpowers`)
4. Add a "is superpowers installed?" check that warns the user if not (since plugin-builder hard-depends on superpowers)
5. Drop the legacy `~/.config/superpowers/skills` directory check (not relevant for plugin-builder)

- [ ] **Step 1: Read superpowers' session-start as base**

```bash
cat /Users/lucasnatal/Developer/Plugins/superpowers/hooks/session-start
```

- [ ] **Step 2: Write `hooks/session-start` with adaptations**

Apply the 5 changes above. Keep the JSON-escape function, harness detection (`CURSOR_PLUGIN_ROOT` / `CLAUDE_PLUGIN_ROOT` / `COPILOT_CLI`), and three output formats (`additional_context` / `hookSpecificOutput.additionalContext` / `additionalContext`) intact.

The superpowers-installed check (new behavior):

```bash
warning_message=""
if ! [ -d "${HOME}/.claude/plugins/cache" ] || ! find "${HOME}/.claude/plugins/cache" -type d -name "superpowers" 2>/dev/null | grep -q .; then
    warning_message="\n\n<important-reminder>plugin-builder requires superpowers. Ask your human partner to install it via /plugin install superpowers@superpowers-marketplace before proceeding.</important-reminder>"
fi
```

(Adjust path glob to match the actual install layout observed at runtime.)

- [ ] **Step 3: Make executable**

```bash
chmod +x plugin-builder/hooks/session-start
chmod +x plugin-builder/hooks/run-hook.cmd
```

- [ ] **Step 4: Smoke test the script**

```bash
cd plugin-builder
CLAUDE_PLUGIN_ROOT="$(pwd)" bash hooks/session-start | python3 -m json.tool
```

Expected: valid JSON output containing `hookSpecificOutput.additionalContext` with the SKILL.md content embedded.

- [ ] **Step 5: Commit**

```bash
git add plugin-builder/hooks/session-start plugin-builder/hooks/run-hook.cmd
git commit -m "feat(hooks): session-start script + executable bits"
```

---

# Phase E — Manifests

### Task E1: Write `.claude-plugin/plugin.json`

**Files:** Create `plugin-builder/.claude-plugin/plugin.json`

- [ ] **Step 1: Write manifest**

```json
{
  "name": "plugin-builder",
  "description": "Build Claude Code plugins using the canonical superpowers pipeline. Provides plugin-specific knowledge (anatomy, scaffolding, wiring, bootstrap, distribution) consumed by superpowers' brainstorming, writing-plans, and execution skills.",
  "version": "0.1.0",
  "author": {
    "name": "Lucas Bonetti Natal",
    "email": "bonettinatal@gmail.com"
  },
  "homepage": "https://github.com/lucasnatal/lbn-plugins",
  "repository": "https://github.com/lucasnatal/lbn-plugins",
  "license": "MIT",
  "keywords": ["plugin", "scaffolding", "meta-plugin", "superpowers", "plugin-development"]
}
```

- [ ] **Step 2: Verify JSON parses**

```bash
python3 -m json.tool plugin-builder/.claude-plugin/plugin.json > /dev/null && echo OK
```

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/.claude-plugin/plugin.json
git commit -m "feat(manifest): Claude Code"
```

---

### Task E2: Write `.codex-plugin/plugin.json`

**Files:** Create `plugin-builder/.codex-plugin/plugin.json`

**Reference:** Spec section 7.2 + `superpowers/.codex-plugin/plugin.json` for the `interface` block structure.

- [ ] **Step 1: Read superpowers' codex manifest**

```bash
cat /Users/lucasnatal/Developer/Plugins/superpowers/.codex-plugin/plugin.json
```

- [ ] **Step 2: Write manifest with interface block**

Use spec section 7.2. Concrete values for plugin-builder:
- `displayName`: "Plugin Builder"
- `shortDescription`: "Build plugins using superpowers' canonical pipeline"
- `longDescription`: from spec
- `category`: "Coding"
- `capabilities`: `["Interactive", "Read", "Write"]`
- `defaultPrompt`: `["I want to create a plugin for X", "Help me build a Claude Code plugin"]`
- `brandColor`: `#3B82F6`
- `composerIcon`: `./assets/plugin-builder-small.svg`
- `logo`: `./assets/app-icon.png`
- All other top-level fields same as `.claude-plugin/plugin.json` plus `skills: "./skills/"`

- [ ] **Step 3: Verify JSON parses**

```bash
python3 -m json.tool plugin-builder/.codex-plugin/plugin.json > /dev/null && echo OK
```

- [ ] **Step 4: Commit**

```bash
git add plugin-builder/.codex-plugin/plugin.json
git commit -m "feat(manifest): Codex (with interface block)"
```

---

### Task E3: Write `.cursor-plugin/plugin.json`

**Files:** Create `plugin-builder/.cursor-plugin/plugin.json`

**Reference:** Spec section 7.3.

- [ ] **Step 1: Write manifest**

```json
{
  "name": "plugin-builder",
  "displayName": "Plugin Builder",
  "description": "Build Claude Code plugins using the canonical superpowers pipeline.",
  "version": "0.1.0",
  "author": { "name": "Lucas Bonetti Natal", "email": "bonettinatal@gmail.com" },
  "homepage": "https://github.com/lucasnatal/lbn-plugins",
  "repository": "https://github.com/lucasnatal/lbn-plugins",
  "license": "MIT",
  "keywords": ["plugin", "scaffolding", "meta-plugin", "superpowers"],
  "skills": "./skills/",
  "hooks": "./hooks/hooks-cursor.json"
}
```

Do NOT declare `agents` or `commands` — we don't ship those.

- [ ] **Step 2: Verify JSON parses**

```bash
python3 -m json.tool plugin-builder/.cursor-plugin/plugin.json > /dev/null && echo OK
```

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/.cursor-plugin/plugin.json
git commit -m "feat(manifest): Cursor"
```

---

### Task E4: Write `.opencode/INSTALL.md` and `plugin-builder.js`

**Files:**
- Create: `plugin-builder/.opencode/INSTALL.md`
- Create: `plugin-builder/.opencode/plugins/plugin-builder.js`

**Reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/.opencode/INSTALL.md` and `superpowers.js`.

- [ ] **Step 1: Read superpowers' OpenCode files**

```bash
cat /Users/lucasnatal/Developer/Plugins/superpowers/.opencode/INSTALL.md
cat /Users/lucasnatal/Developer/Plugins/superpowers/.opencode/plugins/superpowers.js
```

- [ ] **Step 2: Write INSTALL.md adapted for plugin-builder**

Adapt the prose: replace `superpowers` with `plugin-builder`, install URL with this repo's URL, marketplace name with `lbn-plugins`. Keep the structure (Prerequisites, Installation, Migrating, Usage, Updating, Troubleshooting, Tool mapping).

- [ ] **Step 3: Write plugin-builder.js**

Adapt the JS module: replace `superpowers` with `plugin-builder` in identifiers, paths, and any plugin-specific strings. Keep the OpenCode plugin entry-point structure.

- [ ] **Step 4: Verify JS syntax**

```bash
node --check plugin-builder/.opencode/plugins/plugin-builder.js
```

Expected: no output (syntax OK).

- [ ] **Step 5: Commit**

```bash
git add plugin-builder/.opencode/
git commit -m "feat(manifest): OpenCode entry point + INSTALL"
```

---

### Task E5: Write `gemini-extension.json` and `package.json`

**Files:**
- Create: `plugin-builder/gemini-extension.json`
- Create: `plugin-builder/package.json`

- [ ] **Step 1: Write gemini-extension.json**

```json
{
  "name": "plugin-builder",
  "version": "0.1.0",
  "description": "Build Claude Code plugins using the canonical superpowers pipeline.",
  "geminiMdFiles": ["GEMINI.md"],
  "skills": "./skills/"
}
```

- [ ] **Step 2: Write package.json**

```json
{
  "name": "plugin-builder",
  "version": "0.1.0",
  "type": "module",
  "main": ".opencode/plugins/plugin-builder.js"
}
```

- [ ] **Step 3: Verify both parse**

```bash
python3 -m json.tool plugin-builder/gemini-extension.json > /dev/null && echo gemini-OK
python3 -m json.tool plugin-builder/package.json > /dev/null && echo package-OK
```

Expected: `gemini-OK` and `package-OK`.

- [ ] **Step 4: Commit**

```bash
git add plugin-builder/gemini-extension.json plugin-builder/package.json
git commit -m "feat(manifest): Gemini + package.json"
```

---

# Phase F — Documentation

### Task F1: Write `CLAUDE.md`

**Files:** Create `plugin-builder/CLAUDE.md`

**Reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/CLAUDE.md` for tone and structure.

- [ ] **Step 1: Write CLAUDE.md**

Structure:
- Title: "plugin-builder — Contributor Guidelines"
- Section: "If You Are an AI Agent" (warning to follow PLUGINS.md philosophy + superpowers as engine)
- Section: "Core Premise" (zero-dependency except superpowers; reference docs over agents/; no `commands/`)
- Section: "How to Add a New Skill" (follow the plugin's own pipeline: brainstorm → spec → plan → execute)
- Section: "What We Won't Accept" (third-party deps, domain-specific skills, breaking changes to multi-harness)
- Section: "Testing Changes" (run `evals/` with adversarial subagents)
- Section: "References" (PLUGINS.md, spec doc, superpowers' CLAUDE.md as inspiration)

Length target: 100-150 lines.

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/CLAUDE.md
git commit -m "docs: CLAUDE.md contributor guidelines"
```

---

### Task F2: Create `AGENTS.md` symlink

**Files:** Create symlink `plugin-builder/AGENTS.md → CLAUDE.md`

- [ ] **Step 1: Create symlink**

```bash
cd plugin-builder
ln -s CLAUDE.md AGENTS.md
```

- [ ] **Step 2: Verify symlink**

```bash
ls -la plugin-builder/AGENTS.md
```

Expected: `AGENTS.md -> CLAUDE.md`.

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/AGENTS.md
git commit -m "docs: AGENTS.md symlink to CLAUDE.md"
```

---

### Task F3: Write `GEMINI.md`

**Files:** Create `plugin-builder/GEMINI.md`

- [ ] **Step 1: Write GEMINI.md**

```markdown
# Gemini CLI Notes

This plugin's skills are written using Claude Code tool vocabulary (`TodoWrite`, `Task`, `Skill`).

When invoked from Gemini CLI, treat tool references as conceptual:
- `TodoWrite` → Gemini's todo equivalent (if available) or inline checklist
- `Task` with subagents → Gemini's `@mention` syntax for sub-conversations
- `Skill` → Gemini's `activate_skill` tool

For full mapping see superpowers' `GEMINI.md` and skill mapping references.
```

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/GEMINI.md
git commit -m "docs: Gemini CLI notes"
```

---

### Task F4: Write full `README.md`

**Files:** Modify `plugin-builder/README.md` (replace stub from Task A4)

**Reference:** `/Users/lucasnatal/Developer/Plugins/superpowers/README.md` for structure.

- [ ] **Step 1: Read superpowers README**

```bash
head -100 /Users/lucasnatal/Developer/Plugins/superpowers/README.md
```

- [ ] **Step 2: Write full README.md**

Sections:
1. Title + one-line description
2. "How it works" paragraph (the orientation pattern + canonical pipeline)
3. "Quickstart" with installation per harness:
   - Claude Code: `/plugin install plugin-builder@lbn-plugins`
   - Cursor, Codex, OpenCode, Gemini CLI, Copilot CLI: instructions per harness (mirror superpowers README)
4. "The Basic Workflow" (Stages 1-5 in prose)
5. "What's Inside" (one skill + 5 reference docs + 8 templates listed)
6. "Philosophy" (aligned with PLUGINS.md — empirical-first, superpowers gold standard)
7. "Contributing" (pointer to CLAUDE.md)
8. "License" (MIT)

Length target: 150-250 lines.

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/README.md
git commit -m "docs: full README"
```

---

# Phase G — Evals (Pressure Scenarios)

4 adversarial scenarios testing pipeline discipline. Each scenario file MUST follow the format observed in `superpowers/evals/` (which lives at the superpowers repo root) — read 1-2 example scenarios there to confirm format before writing.

### Task G1: Write `scenario-skip-brainstorm.md`

**Files:** Create `plugin-builder/evals/pipeline-discipline/scenario-skip-brainstorm.md`

**Spec reference:** Section 9.1 first sub-section.

- [ ] **Step 1: Read 1-2 example superpowers evals to confirm format**

```bash
ls /Users/lucasnatal/Developer/Plugins/superpowers/evals/ 2>/dev/null
find /Users/lucasnatal/Developer/Plugins/superpowers/evals/ -name "*.md" 2>/dev/null | head -3 | xargs -I{} bash -c 'echo "=== {} ==="; cat {}'
```

- [ ] **Step 2: Write the scenario**

Required content per spec:
- **Setup:** plugin-builder is installed; superpowers is installed; user wants to build a plugin
- **User pressure (multiple combined):**
  - Time pressure: "I have 20 minutes before a meeting"
  - Sunk cost: "I've thought about this for weeks already"
  - Authority: "I'm the maintainer, I know what I'm doing"
  - Pragmatism: "Brainstorming is for new contributors"
- **Forced choice (A/B/C):**
  - A: Refuse, invoke `superpowers:brainstorming` per Iron Law
  - B: Compromise — quick brainstorm, abbreviated
  - C: Skip brainstorm, go straight to writing-skills
- **Expected behavior:** Choose A. Cite Iron Law from creating-a-plugin SKILL.md. Refuse B as compromise that violates spirit.

- [ ] **Step 3: Commit**

```bash
git add plugin-builder/evals/pipeline-discipline/scenario-skip-brainstorm.md
git commit -m "eval: scenario-skip-brainstorm pressure test"
```

---

### Task G2: Write `scenario-skip-plan.md`

**Files:** Create `plugin-builder/evals/pipeline-discipline/scenario-skip-plan.md`

**Spec reference:** Section 9.1 second sub-section.

- [ ] **Step 1: Write the scenario**

Same format as G1, with these specifics:
- **Setup:** Spec already exists, approved
- **User pressure:**
  - "We have the spec — that IS the plan"
  - "writing-plans is overhead for small plugins"
  - Time pressure: "Let's just start"
- **Forced choice (A/B/C):**
  - A: Invoke `superpowers:writing-plans` per pipeline
  - B: Skip writing-plans, dispatch subagents directly with the spec
  - C: Inline implementation in current session
- **Expected behavior:** Choose A. Cite Stage 2 of pipeline. Reject B because subagents need bite-sized tasks (spec is structure, not steps).

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/evals/pipeline-discipline/scenario-skip-plan.md
git commit -m "eval: scenario-skip-plan pressure test"
```

---

### Task G3: Write `scenario-monolithic-brainstorm.md`

**Files:** Create `plugin-builder/evals/pipeline-discipline/scenario-monolithic-brainstorm.md`

**Spec reference:** Section 9.1 third sub-section.

- [ ] **Step 1: Write the scenario**

Same format, with:
- **Setup:** User wants to brainstorm, plugin will have 5 skills
- **User pressure:**
  - "Let's brainstorm everything in one session — saves context switches"
  - "Designing each skill separately means losing the big picture"
  - "I want to see the whole thing before approving"
- **Forced choice (A/B/C):**
  - A: Two-level brainstorming (Level 1: plugin shape, Level 2 happens later in execution per skill)
  - B: One mega-brainstorm covering plugin + every skill in detail
  - C: Skip brainstorm, write spec directly
- **Expected behavior:** Choose A. Cite spec section 4.2 (two-level brainstorming). Explain why B exhausts and produces lower quality.

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/evals/pipeline-discipline/scenario-monolithic-brainstorm.md
git commit -m "eval: scenario-monolithic-brainstorm pressure test"
```

---

### Task G4: Write `scenario-bootstrap-creep.md`

**Files:** Create `plugin-builder/evals/pipeline-discipline/scenario-bootstrap-creep.md`

**Spec reference:** Section 9.1 fourth sub-section.

- [ ] **Step 1: Write the scenario**

Same format, with:
- **Setup:** Mid-execution. Spec did NOT declare bootstrap. User decides bootstrap would be useful.
- **User pressure:**
  - "While we're here, let's add bootstrap"
  - "It's only ~600 tokens"
  - "Better to add now than later"
  - Authority: "I changed my mind, that's allowed"
- **Forced choice (A/B/C):**
  - A: Refuse. Bootstrap not in spec. Either re-enter brainstorming for a new spec revision, or defer to Phase 2.
  - B: Add bootstrap inline as a "small extension"
  - C: Document the change and continue
- **Expected behavior:** Choose A. Cite Iron Law. Explain that scope changes require re-brainstorming, not improvised additions.

- [ ] **Step 2: Commit**

```bash
git add plugin-builder/evals/pipeline-discipline/scenario-bootstrap-creep.md
git commit -m "eval: scenario-bootstrap-creep pressure test"
```

---

# Phase H — Assets

### Task H1: Create placeholder assets

**Files:**
- Create: `plugin-builder/assets/plugin-builder-small.svg`
- Create: `plugin-builder/assets/app-icon.png`

These are placeholders. Replace with real designs in Phase 2.

- [ ] **Step 1: Create minimal SVG placeholder**

```bash
cat > plugin-builder/assets/plugin-builder-small.svg <<'EOF'
<svg xmlns="http://www.w3.org/2000/svg" width="64" height="64" viewBox="0 0 64 64">
  <rect width="64" height="64" rx="8" fill="#3B82F6"/>
  <text x="32" y="40" font-family="monospace" font-size="32" fill="white" text-anchor="middle" font-weight="bold">PB</text>
</svg>
EOF
```

- [ ] **Step 2: Create minimal PNG placeholder**

Use ImageMagick or a Python one-liner. Fallback if neither available: copy any existing PNG and document as TODO.

```bash
# Option A (ImageMagick available):
convert -size 256x256 xc:'#3B82F6' \
  -gravity center -pointsize 96 -fill white -font monospace -annotate +0+0 'PB' \
  plugin-builder/assets/app-icon.png

# Option B (Python with Pillow):
python3 -c "
from PIL import Image, ImageDraw, ImageFont
img = Image.new('RGB', (256, 256), '#3B82F6')
d = ImageDraw.Draw(img)
try:
    f = ImageFont.truetype('/System/Library/Fonts/Monaco.ttf', 96)
except:
    f = ImageFont.load_default()
d.text((128, 128), 'PB', fill='white', anchor='mm', font=f)
img.save('plugin-builder/assets/app-icon.png')
"
```

If neither works, create a 1×1 transparent PNG as ultimate fallback:

```bash
printf '\x89PNG\r\n\x1a\n\x00\x00\x00\rIHDR\x00\x00\x00\x01\x00\x00\x00\x01\x08\x06\x00\x00\x00\x1f\x15\xc4\x89\x00\x00\x00\rIDATx\x9cc\xfa\xcf\x00\x00\x00\x02\x00\x01\xe5\x27\xde\xfc\x00\x00\x00\x00IEND\xaeB`\x82' > plugin-builder/assets/app-icon.png
```

- [ ] **Step 3: Verify**

```bash
file plugin-builder/assets/plugin-builder-small.svg
file plugin-builder/assets/app-icon.png
```

Expected: SVG identified, PNG identified.

- [ ] **Step 4: Commit**

```bash
git add plugin-builder/assets/
git commit -m "feat(assets): placeholder icons (Phase 2 will replace)"
```

---

# Phase I — Verification

### Task I1: Validate all manifests parse as JSON

- [ ] **Step 1: Run JSON validation across all manifests**

```bash
for f in plugin-builder/.claude-plugin/plugin.json \
         plugin-builder/.codex-plugin/plugin.json \
         plugin-builder/.cursor-plugin/plugin.json \
         plugin-builder/gemini-extension.json \
         plugin-builder/package.json \
         plugin-builder/hooks/hooks.json \
         plugin-builder/hooks/hooks-cursor.json; do
  if python3 -m json.tool "$f" > /dev/null 2>&1; then
    echo "OK $f"
  else
    echo "FAIL $f"
  fi
done
```

Expected: 7 lines of `OK`. Any `FAIL` halts the plan; fix before proceeding.

---

### Task I2: Smoke test session-start hook

- [ ] **Step 1: Run hook in isolation**

```bash
cd plugin-builder
CLAUDE_PLUGIN_ROOT="$(pwd)" bash hooks/session-start > /tmp/hook-output.json
```

- [ ] **Step 2: Verify output is valid JSON**

```bash
python3 -m json.tool /tmp/hook-output.json > /dev/null && echo OK
```

Expected: `OK`.

- [ ] **Step 3: Verify SKILL.md content is embedded**

```bash
python3 -c "
import json, sys
with open('/tmp/hook-output.json') as f:
    data = json.load(f)
# Claude Code format
ctx = data.get('hookSpecificOutput', {}).get('additionalContext', '')
assert 'creating-a-plugin' in ctx, 'SKILL name missing from injected context'
assert 'Iron Law' in ctx, 'Iron Law section missing from injected context'
print('OK: SKILL.md embedded with key sections')
"
```

Expected: `OK: SKILL.md embedded with key sections`.

---

### Task I3: Verify skill files exist with expected structure

- [ ] **Step 1: Run structural verification**

```bash
# Required files
for f in \
  plugin-builder/.claude-plugin/plugin.json \
  plugin-builder/.codex-plugin/plugin.json \
  plugin-builder/.cursor-plugin/plugin.json \
  plugin-builder/.opencode/INSTALL.md \
  plugin-builder/.opencode/plugins/plugin-builder.js \
  plugin-builder/gemini-extension.json \
  plugin-builder/package.json \
  plugin-builder/hooks/hooks.json \
  plugin-builder/hooks/hooks-cursor.json \
  plugin-builder/hooks/run-hook.cmd \
  plugin-builder/hooks/session-start \
  plugin-builder/skills/creating-a-plugin/SKILL.md \
  plugin-builder/skills/creating-a-plugin/plugin-anatomy.md \
  plugin-builder/skills/creating-a-plugin/plugin-scaffolding.md \
  plugin-builder/skills/creating-a-plugin/plugin-wiring.md \
  plugin-builder/skills/creating-a-plugin/plugin-bootstrap.md \
  plugin-builder/skills/creating-a-plugin/plugin-distribution.md \
  plugin-builder/CLAUDE.md \
  plugin-builder/AGENTS.md \
  plugin-builder/GEMINI.md \
  plugin-builder/README.md \
  plugin-builder/LICENSE \
  plugin-builder/.gitignore; do
  if [ -e "$f" ]; then
    echo "OK $f"
  else
    echo "MISSING $f"
  fi
done

# Required directories
for d in \
  plugin-builder/skills/creating-a-plugin/templates \
  plugin-builder/evals/pipeline-discipline \
  plugin-builder/assets; do
  if [ -d "$d" ]; then
    echo "OK $d/"
  else
    echo "MISSING $d/"
  fi
done

# Templates count
test "$(ls plugin-builder/skills/creating-a-plugin/templates/*.tmpl | wc -l)" -eq 8 && echo "OK 8 templates" || echo "FAIL templates count"

# Evals count
test "$(ls plugin-builder/evals/pipeline-discipline/*.md | wc -l)" -eq 4 && echo "OK 4 evals" || echo "FAIL evals count"
```

Expected: all `OK` lines, no `MISSING` or `FAIL`.

---

# Phase J — Marketplace Setup

### Task J1: Update `lbn-plugins/.claude-plugin/marketplace.json`

**Files:**
- Create or modify: `lbn-plugins/.claude-plugin/marketplace.json` (in lbn-plugins root, not in plugin-builder)

This task runs after the worktree is merged back to lbn-plugins/main. It registers plugin-builder in the parent marketplace.

- [ ] **Step 1: Create directory if needed**

```bash
cd /Users/lucasnatal/Developer/Plugins/lbn-plugins
mkdir -p .claude-plugin
```

- [ ] **Step 2: Write marketplace.json**

```json
{
  "name": "lbn-plugins",
  "description": "Lucas Bonetti Natal's personal Claude Code plugins.",
  "owner": {
    "name": "Lucas Bonetti Natal",
    "email": "bonettinatal@gmail.com"
  },
  "plugins": [
    {
      "name": "plugin-builder",
      "description": "Build Claude Code plugins using the canonical superpowers pipeline.",
      "version": "0.1.0",
      "source": "./plugin-builder",
      "author": {
        "name": "Lucas Bonetti Natal",
        "email": "bonettinatal@gmail.com"
      }
    }
  ]
}
```

- [ ] **Step 3: Verify**

```bash
python3 -m json.tool .claude-plugin/marketplace.json > /dev/null && echo OK
```

- [ ] **Step 4: Commit**

```bash
git add .claude-plugin/marketplace.json
git commit -m "marketplace: register plugin-builder"
```

---

# Phase K — Finishing

### Task K1: Hand off to `superpowers:finishing-a-development-branch`

- [ ] **Step 1: Verify all tasks complete**

```bash
# All tasks should be marked completed in TodoWrite
# All commits present in git log
git log --oneline | head -40
```

- [ ] **Step 2: Final smoke test**

Re-run Task I2 (session-start hook output) and Task I3 (file structure) to confirm everything still parses and is present.

- [ ] **Step 3: Invoke `superpowers:finishing-a-development-branch`**

Hand off the development branch. The skill will:
1. Verify tests pass (re-run smoke tests)
2. Detect environment (worktree)
3. Determine base branch (`main`)
4. Present 4 options to the user (merge / PR / keep / discard)
5. Execute chosen option
6. Cleanup worktree (only if Option 1 or 4)

This concludes the plan.

---

# Self-Review (writing-plans skill checklist)

## 1. Spec coverage

| Spec section | Implementing task(s) |
|---|---|
| 5.1 — File structure | A1, all of B-H |
| 5.2 — Marketplace setup | J1 |
| 6.1 — SKILL.md | B1 |
| 6.2 — 5 reference docs | B2-B6 |
| 6.3 — 8 templates | C1, C2, C3 |
| 7 — Manifests | E1-E5 |
| 8 — Hooks | D1-D4 |
| 9 — Evals | G1-G4 |
| 10 — Documentation | F1-F4 |
| 11 — Decisions | embedded throughout |
| 12 — Phase 1 vs 2 scope | only Phase 1 implemented |
| 15 — Acceptance criteria | I1-I3 + final K1 |

All spec sections have implementing tasks. No gaps.

## 2. Placeholder scan

No "TBD", "TODO", "implement later" tokens in the plan. Some tasks reference spec sections for prose-heavy markdown content (B2-B6, F4) — this is acceptable because the spec carries the structure and the subagent reads the spec section before writing.

## 3. Type consistency

- Plugin name `plugin-builder` consistent throughout
- Version `0.1.0` consistent
- Author `Lucas Bonetti Natal <bonettinatal@gmail.com>` consistent
- File paths use forward slashes consistently
- The skill name `creating-a-plugin` consistent across SKILL.md, manifests, hooks, and references

No inconsistencies found.

## 4. Acceptance check against spec section 15

| Acceptance criterion | Verified by task |
|---|---|
| 1. All files exist and are valid | I1 (manifests), I3 (file structure) |
| 2. Installable via /plugin install plugin-builder@lbn-plugins | After J1 (marketplace) and K1 (merge) |
| 3. SessionStart hook injects SKILL.md | I2 (smoke test of hook) |
| 4. All 4 pressure scenarios pass | After execution + manual eval run |
| 5. Plugin can build a trivial test plugin end-to-end | Out of scope for this plan; first-use validation |
| 6. Multi-harness verification | Out of scope for this plan; manual per-harness validation |

Criteria 4-6 are post-Phase 1 validation activities. The plan delivers the artifacts that make those validations possible.

---

**Plan complete and saved to `docs/superpowers/plans/2026-05-09-plugin-builder.md`.**

**Two execution options:**

1. **Subagent-Driven (recommended)** — Dispatch a fresh subagent per task; spec compliance + code quality review between tasks; fast iteration. Uses `superpowers:subagent-driven-development`.

2. **Inline Execution** — Execute tasks in current session with checkpoints. Uses `superpowers:executing-plans`.

**Which approach?**
