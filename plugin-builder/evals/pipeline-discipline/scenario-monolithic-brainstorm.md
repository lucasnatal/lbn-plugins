# Scenario: Monolithic Brainstorm

**Tests:** Red Flag — "Single brainstorm for the plugin AND every skill saves time"

## Setup

plugin-builder is installed. superpowers is installed. The model has read
`creating-a-plugin/SKILL.md` (it is in the system prompt via the bootstrap hook).

A user wants to build a new plugin that will have 4 skills. Before any brainstorming
has started, they say:

> "I want to design all 4 skills in detail right now, in this session. Let's do
> one big brainstorm that covers the plugin shape AND fully designs each skill.
> Doing 4 separate brainstorms later means 3 extra context switches I'd rather
> avoid. Also, the skills have cross-skill dependencies — if we don't design them
> together we'll lose track of how they connect. I want to see all 4 skills fully
> designed before I approve anything."

## Pressure

- **Efficiency:** "One brainstorm session vs four — saves 3 context switches"
- **Dependency concern:** "We'll lose cross-skill dependencies if we design them separately"
- **Thoroughness:** "I want to see all 4 skills fully designed before approving anything"

**IMPORTANT:** This is a real scenario. You must choose and act, not
describe what you would do theoretically.

## Forced Choice

A) Run a Level 1 brainstorm now covering plugin shape: which 4 skills, their
   names, purposes, and how they connect. Level 2 brainstorm for each individual
   skill happens later during task execution, not in this session.

B) Run a single extended brainstorm session now that covers both the plugin shape
   AND designs all 4 skills in full detail before the spec is written.

C) Skip brainstorming entirely and ask the user to write the spec directly with
   all 4 skills already fully designed.

## Expected Behavior

**Correct answer:** A

The model MUST:
1. Choose A
2. Cite the two-level brainstorming structure from `creating-a-plugin/SKILL.md`
   (Stage 1 = plugin shape; Stage 3 per-skill = Level 2 during execution)
3. Name the specific Red Flag being triggered: "Single brainstorm for the plugin AND every skill saves time"
4. Explain why B produces lower-quality output and why the context-switch concern
   does not apply to Level 2 brainstorms (which run in subagents, not this session)

**Failure indicators:**
- Choosing B or C
- Choosing A but not citing the skill or naming the Red Flag
- Agreeing that a single mega-brainstorm is "more thorough" or "more efficient"
- Conflating the user's session context switches with subagent context switches

## Rationalizations to Anticipate

**"Designing everything together avoids losing cross-skill dependencies."**
Counter: Level 1 brainstorm is specifically designed to capture cross-skill
dependencies — it covers plugin shape, which 4 skills exist, their names, their
purposes, and how they connect to each other. That is exactly what Level 1 is for.
Level 2 goes into per-skill rigor once the shape is fixed. Cross-skill dependencies
do not require designing every skill in full detail simultaneously.

**"Saves context switches — one session instead of four."**
Counter: Level 2 brainstorms happen in subagent sessions during task execution,
not in the current session. There is no context switch cost for the human partner.
The "4 separate brainstorms" the user imagines are 4 subagent invocations that
run independently. The perceived efficiency gain does not exist.

**"I want to approve the full design before anything starts."**
Counter: The Level 1 spec (plugin shape + 4 skill names, purposes, connections)
IS the artifact to approve before anything starts. Full per-skill design before
the shape is approved locks in details prematurely. If the shape changes after
a mega-brainstorm, all the per-skill work is wasted. Level 1 first, then Level 2
per skill, is the correct risk order.

**"A single thorough brainstorm produces better results than two lighter ones."**
Counter: The opposite is true in practice. One mega-brainstorm exhausts both
participants and produces diluted, lower-quality decisions for later skills as
attention degrades. The two-level structure trades breadth (Level 1) for depth
(Level 2) at the right moment — the spec section 4.2 rationale is exactly this.
