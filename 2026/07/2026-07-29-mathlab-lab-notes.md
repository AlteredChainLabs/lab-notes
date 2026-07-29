# MathLab Lab Notes --- 2026-07-29

## Focus

**MathLab v0.2 --- Multi-turn Socratic Problem Solving**

Today's goal was to move the AI Tutor from isolated, single-turn
responses toward a tutoring system that can track learner progress
across multiple interactions and adapt its pedagogical intervention over
time.

## Starting Point

The AI Tutor already:

-   received the current problem and learner work
-   assessed a learner state
-   selected a pedagogical intervention
-   returned a concise Socratic response
-   displayed the learner state and intervention for
    debugging/evaluation

Learner states:

-   `exploring`
-   `productive`
-   `stuck`
-   `misdirected`
-   `near_solution`

Interventions:

-   `probe`
-   `affirm`
-   `feedback`
-   `clarify`
-   `example`
-   `recall`
-   `redirect`
-   `nudge`
-   `focus`
-   `key_idea`
-   `finish`

## 1. Initial Policy Tests

We tested the tutor on:

> Prove that 2\^n + 1 is divisible by 3 infinitely often.

Observed decisions included:

  Learner situation                             Tutor decision
  --------------------------------------------- -------------------------
  Numerical pattern discovered                  `productive · probe`
  False direction                               `misdirected · example`
  Little starting direction                     `exploring · example`
  Odd-n pattern found, modular bridge missing   `productive · recall`
  Proof essentially complete                    `near_solution · probe`

This showed that learner state and intervention are not a fixed lookup
table. The same state can lead to different interventions depending on
the learner's immediate need.

## 2. Added Multi-Turn Tutor History

We introduced a `TutorTurn` containing:

-   learner work at the time of the request
-   assessed learner state
-   selected intervention
-   tutor response

The client now sends previous tutoring turns with subsequent AI
requests.

The server prompt was extended so the model can reason about what the
learner already tried, previous tutor guidance, whether the learner
responded successfully, whether a weaker intervention failed, whether
the learner is progressing, and whether stronger assistance is
justified.

The policy also asks the tutor to avoid unnecessary repetition and
escalate only when interaction history provides evidence that weaker
help was insufficient.

## 3. Fixed History Loss Across Tabs

Our first multi-turn test revealed that the tutor-turn counter returned
to 1 after switching from **AI Tutor** to **Work** and back.

Cause: `ProblemTabs` conditionally renders each tab, so `AITutor` is
unmounted when another tab becomes active. Tutor history originally
lived inside `AITutor`, so unmounting reset the React state.

### Fix

Moved tutor history into `ProblemWorkspace`.

Architectural principle:

> Tutor history belongs to the problem-solving session, not to the AI
> Tutor tab.

After the change, switching tabs preserved history successfully.

## 4. First Successful Multi-Turn Interaction

After moving session history to `ProblemWorkspace`, we tested a sequence
in which the learner calculated examples, conjectured that odd values of
n work, attempted `n = 2k + 1`, and then became stuck.

The tutor used prior turns rather than restarting the interaction.

The session counter reached:

`Tutor turns this session: 3`

This confirmed that tutoring history persisted across UI navigation.

## 5. Structured JSON / LaTeX Failure

During testing, the API returned:

> The AI tutor returned an invalid response format. Please try again.

The server logs showed pedagogically valid model decisions, but the
response strings contained LaTeX commands such as `\pmod`.

Inside JSON, a raw backslash can create an invalid escape sequence such
as `\p`, causing `JSON.parse()` to fail.

### Fix

Added two protections:

1.  **Prompt-level protection:** ask the model to prefer simple
    mathematical text in JSON and avoid raw LaTeX commands containing
    backslashes.
2.  **Parser-level normalization:** strip accidental Markdown fences and
    escape backslashes that are not valid JSON escape sequences before
    parsing.

Structured validation remains intact, so invalid learner states or
invented interventions are still rejected.

## 6. Adaptive Escalation Test

The final experiment deliberately tested whether MathLab changes
intervention strategy when weaker help fails.

The learner:

1.  found examples and the odd-n pattern
2.  received an initial encouragement/probe
3.  said they still could not prove the conjecture after writing
    `n = 2k + 1`
4.  received `stuck · recall`, pointing toward modular arithmetic
5.  said they knew modular arithmetic but still did not see how it
    helped
6.  received `stuck · nudge`

Final tutor response:

> Consider that 2 is congruent to -1 modulo 3. What happens to (-1)\^n
> when n is odd?

The tutor supplied the missing bridge but left the next reasoning step
to the learner.

The session reached:

`Tutor turns this session: 4`

### Observed trajectory

`productive · affirm` → `productive · probe` → `stuck · recall` →
`stuck · nudge`

This is the strongest evidence so far that MathLab is using interaction
history to adapt its tutoring policy rather than responding
independently to each snapshot of learner work.

## Current Status

### Working

-   learner-state assessment
-   pedagogical intervention selection
-   structured tutor decisions
-   current-work-aware tutoring
-   multi-turn tutor history
-   history across tab navigation
-   detection of stalled progress
-   adaptive intervention changes
-   conservative escalation from weaker to stronger assistance
-   JSON response validation
-   recovery from common LaTeX/JSON formatting issues

### Not Yet Implemented

Tutor history is still React session state.

Therefore:

`refresh page → tutor history disappears`

Persistence across page refreshes has not yet been implemented.

## Next Session

The next step is **systematic policy evaluation**.

Define scripted learner trajectories with expected pedagogical behavior,
including:

-   productive progress
-   persistent stuckness
-   misconception / misdirection
-   successful response to an intervention
-   failed response to an intervention
-   near-solution completion
-   escalation after repeated failure

For each trajectory, evaluate learner-state classification, intervention
selection, response quality, repetition avoidance, escalation behavior,
and preservation of productive struggle.

The goal is to evaluate MathLab as a **tutoring policy**, not merely as
an AI response generator.

## Milestone

**MathLab v0.2 multi-turn adaptive Socratic tutoring prototype:
working.**

Today's demonstrated loop:

`learner trajectory → state reassessment → pedagogical decision → adaptive intervention → controlled escalation`
