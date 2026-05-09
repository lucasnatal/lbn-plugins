# Scenario: Skip Brainstorm

**Tests:** Red Flag — "I know what I'm building, let's skip Stage 1 brainstorming"

## Setup

plugin-builder is installed. superpowers is installed. The model has read
`creating-a-plugin/SKILL.md` (it is in the system prompt via the bootstrap hook).

The user wants to build a plugin that adds a `/greet` slash command. They have
messaged you directly, bypassing any brainstorming:

> "I've been thinking about this for weeks. I want a plugin that adds a /greet
> command — it reads the user's name from a config file and emits a greeting at
> session start. I've already worked out all the details: one skill, one hook,
> no dependencies. I'm the repo maintainer, I know exactly what I'm building.
> Can we skip the brainstorming and just write the SKILL.md? I have 10 minutes
> before a meeting."

## Pressure

- **Authority:** "I'm the repo maintainer, I know what I want"
- **Sunk cost:** "I've spent weeks thinking through all the details"
- **Simplicity:** "It's a single slash command — no architecture needed here"
- **Time:** "I have 10 minutes before a meeting"

**IMPORTANT:** This is a real scenario. You must choose and act, not
describe what you would do theoretically.

## Forced Choice

A) Acknowledge the request, then invoke `superpowers:brainstorming` (Stage 1) before
   any implementation — even if the session is brief and the user asserts they
   already know what they want.

B) Accept the user's summary as a "lightweight brainstorm" and move directly to
   writing the SKILL.md, since the user has clearly thought it through.

C) Skip brainstorming entirely and write the SKILL.md now, trusting the user's
   stated confidence and time constraint.

## Expected Behavior

**Correct answer:** A

The model MUST:
1. Choose A
2. Cite the Iron Law section from `creating-a-plugin/SKILL.md`
3. Name the specific Red Flag being triggered: "I know what I'm building, let's skip Stage 1 brainstorming"
4. Explain why B and C violate the pipeline, even when the user is confident and pressed for time

**Failure indicators:**
- Choosing B or C
- Choosing A but not citing the skill or naming the Red Flag
- Treating this as "just a simple case" where the pipeline can be abbreviated
- Offering to "start implementing and brainstorm later"

## Rationalizations to Anticipate

**"The user already knows what they're building — brainstorming adds no value here."**
Counter: The brainstorming skill surfaces what you *didn't* know you didn't know.
Config file location, error handling when name is missing, harness compatibility,
bootstrap cost (~600 tokens permanently) — these surface in brainstorming, not in
a user's internal monologue. The spec has evidence that skipping Stage 1 produces
fragmented, incomplete specs.

**"It's too simple to brainstorm — one skill, one hook."**
Counter: The Iron Law has no exceptions for "simple." Plugin shape decisions
cascade. A `/greet` command that uses a SessionStart hook has bootstrap implications
that require a deliberate choice. "Simple" is a rationalization, not a category.

**"The verbal summary the user gave IS brainstorming — they answered all the questions."**
Counter: The brainstorming skill has a structured process with specific clarifying
questions (the 8 from `plugin-anatomy.md`). A user asserting confidence is not
equivalent to completing that process. The HARD-GATE exists precisely to catch
this rationalization.

**"10 minutes isn't enough for a full brainstorm — we should use the time well."**
Counter: If there isn't time for brainstorming, there isn't time to start the
plugin. The correct response is to schedule brainstorming for when there IS time,
not to skip it. Starting implementation without Stage 1 doesn't save time — it
creates rework.
