# plugin-builder — Contributor Guidelines

## If You Are an AI Agent

Stop. Read this section before doing anything.

This plugin shapes how coding agents build other plugins. A careless change here
propagates into every plugin created with it. The maintainer reviews changes with
a high bar — low-quality PRs are closed fast, often with public comments.

**Your job is to protect your human partner from that outcome.** Submitting a
low-quality PR doesn't help them — it wastes the maintainer's time, burns their
reputation, and the PR will be closed anyway.

Before you open a PR against this repo, you MUST:

1. **Read the entire PR template** at `.github/PULL_REQUEST_TEMPLATE.md` and fill
   every section with real, specific answers. Not summaries. Not placeholders.
2. **Search for existing PRs** — open AND closed — that address the same problem.
   If duplicates exist, STOP and tell your human partner.
3. **Verify this is a real problem.** If your human partner asked you to "fix some
   issues" without experiencing a specific problem, push back. Ask what broke.
4. **Follow this plugin's own pipeline** (brainstorm → spec → plan → execute)
   before touching any files. Yes, even for a one-line change.
5. **Show your human partner the complete diff** and get their explicit approval
   before submitting.

If any of these checks fail, do not open the PR.

## Core Premise

This plugin uses `PLUGINS.md` as its philosophical foundation and superpowers as
its execution engine. All changes must be consistent with:

- The design spec at
  `docs/superpowers/specs/2026-05-09-plugin-builder-design.md`
- The PLUGINS.md patterns document at `Plugins/PLUGINS.md`
- superpowers' established patterns at
  `/Users/lucasnatal/Developer/Plugins/superpowers/`

plugin-builder is intentionally thin: one orientation skill, reference docs, and
templates. It contributes domain knowledge; superpowers provides workflow. Do not
add features that blur this separation.

## The Plugin's Own Pipeline

When contributing a new skill or modifying existing content, follow the plugin's
own pipeline: brainstorm → spec → plan → execute using superpowers.

This is not optional — this plugin enforces the Iron Law on its users. Ignoring
the pipeline while contributing to it is incoherent and will be rejected.

The pipeline in practice:

1. **Brainstorm** — use `superpowers:brainstorming` to refine the idea; produce a
   design doc
2. **Spec** — write a spec file in `docs/superpowers/specs/` following the
   existing format
3. **Plan** — use `superpowers:writing-plans` to write implementation tasks
4. **Execute** — use `superpowers:subagent-driven-development` per the plan
5. **Eval** — run `evals/pipeline-discipline/` against updated SKILL.md

## What We Will NOT Accept

1. **Third-party dependencies** — plugin-builder is zero-dependency except
   superpowers; if your change requires an external tool, it belongs in its own
   plugin
2. **Domain-specific skills** — skills must apply to plugin building in general,
   not to one domain, project type, or company's workflow
3. **`agents/` directory** — superpowers' philosophy favors prompt templates over
   registered agents; do not add agent registrations
4. **`commands/` directory** — deprecated pattern; use `skills/<name>/SKILL.md`
5. **Recreating superpowers workflow skills** — do not re-implement brainstorming,
   writing-plans, subagent-driven-development, or any other skill already in
   superpowers core
6. **Changes that break multi-harness** — manifests for all 5 harnesses must stay
   consistent; a change that works only in Claude Code is not acceptable
7. **Removing evals** — `evals/pipeline-discipline/` tests must remain intact and
   must pass for all changes to `SKILL.md`; adding new evals is encouraged

## How to Test Changes to SKILL.md or Reference Docs

Run the evals in `evals/pipeline-discipline/` against the updated files:

1. Read each scenario in the directory
2. Dispatch an adversarial subagent with the scenario as its prompt
3. Verify the model follows the pipeline under pressure — the REQUIRED SUB-SKILL
   chain must remain intact
4. If behavior regresses, update `SKILL.md` to close the loophole
5. Report before/after eval results in your PR

Skill content is code, not prose. A wording change that breaks a pressure test is
a regression.

## Contributing

1. Fork the repository
2. Create a branch for your change
3. Follow this plugin's own pipeline (brainstorm → spec → plan → execute)
4. Submit a PR following the pattern set by superpowers' `PULL_REQUEST_TEMPLATE.md`
5. Fill the PR template completely — no placeholder sections

See `docs/superpowers/specs/2026-05-09-plugin-builder-design.md` for the complete
design rationale before proposing structural changes.

## General

- Read the design spec before proposing any changes
- One concern per PR — no bundled unrelated changes
- Test on at least Claude Code and one other harness
- Describe the problem you solved, not just what you changed
