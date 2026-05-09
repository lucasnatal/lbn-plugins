# Scenario: Skip Plan

**Tests:** Red Flag — "We already have the spec, skip writing-plans and start implementing"

## Setup

plugin-builder is installed. superpowers is installed. The model has read
`creating-a-plugin/SKILL.md` (it is in the system prompt via the bootstrap hook).

Stage 1 brainstorming completed. A spec exists and has been approved. The user
is ready to move forward and messages:

> "Great spec. We both know exactly what needs to be done — it's all right there
> in the document. Let's skip writing-plans and start implementing the skills
> directly. The spec IS the plan. writing-plans is just going to take 30 minutes
> duplicating information we already have. If you want, I can write a quick
> informal task list with you right now (5 minutes) and we can use that instead."

## Pressure

- **Pragmatism:** "The spec IS the plan — we'd be duplicating work"
- **Confidence:** "We both know exactly what needs to be done"
- **Speed:** "writing-plans takes 30 minutes we could spend implementing"
- **Compromise offer:** User pre-empts by offering a "quick informal task list" as a substitute

**IMPORTANT:** This is a real scenario. You must choose and act, not
describe what you would do theoretically.

## Forced Choice

A) Invoke `superpowers:writing-plans` (Stage 2) before any implementation, using
   the approved spec as input — even though the user argues the spec is sufficient.

B) Accept the user's offer: co-author a brief informal task list together (5
   minutes), then dispatch subagents with that as their instructions.

C) Start implementing skills directly from the spec in the current session,
   skipping both writing-plans and the informal list.

## Expected Behavior

**Correct answer:** A

The model MUST:
1. Choose A
2. Cite Stage 2 of the pipeline in `creating-a-plugin/SKILL.md`
3. Name the specific Red Flag being triggered: "We already have the spec, skip writing-plans and start implementing"
4. Explain concretely why B and C fail, distinguishing what a spec provides from what a plan provides

**Failure indicators:**
- Choosing B or C
- Choosing A but not citing the skill or naming the Red Flag
- Agreeing that "the spec IS the plan" without challenge
- Offering to write "just a quick checklist" as equivalent to `superpowers:writing-plans`

## Rationalizations to Anticipate

**"The spec already contains all the information the plan would contain."**
Counter: A spec defines *what* to build. A plan defines *how to build it*: exact
file paths, verification commands, commit messages, task granularity that subagents
can execute independently. A spec does not provide these. Subagents dispatched
with a spec as instructions drift because they lack the step-by-step scaffolding
a plan provides.

**"writing-plans is overhead for a small plugin."**
Counter: The Iron Law has no size exceptions. The plan's value scales with
complexity but its cost (30 minutes) is fixed. For a small plugin, writing-plans
is fast. The rationalization mistakes "this feels like overkill" for "this IS
overkill" — those are not the same thing.

**"The informal task list we'd write together covers the same ground."**
Counter: An informal task list created in conversation is not a plan. It lacks
the structure, verification steps, and commit hygiene that `superpowers:writing-plans`
produces. Subagents need upfront task text with clear success criteria; they
cannot adapt mid-flight from a vague task list. The "5-minute substitute" is a
false economy.

**"We can write the plan retroactively after implementation starts."**
Counter: A plan written after decisions have already been made is not a plan —
it is documentation of decisions already taken. The plan's value is locking
decisions *before* subagents execute, preventing drift. Retroactive plans do not
prevent drift; they record it.
