# CLAUDE.md

Working instructions for this repository. Read this first, then the specs.

## What this is

A single deliverable: `demo.html` in the repo root. A deterministic, client-side simulation comparing two delivery processes on one brief. It exists to be presented, and it is itself the worked output of the process it demonstrates — which is why the build log at the end of this file matters as much as the artifact.

## Reading order

1. `SPEC-machine.md` — the contract. State shape, model data, computation, interaction table, forbidden list. This is authoritative.
2. `SPEC-human.md` — intent and screen structure. Read it to understand why, and to resolve anything the machine spec leaves open.
3. `SPEC-tests.md` — the definition of done. Every criterion is checkable. Work through them explicitly before declaring the build complete.

Reference files, for matching rather than reading closely:

- `reference/ai-worked-example.html` — visual reference for the segment bars, stage blocks, and typographic treatment. Match it; do not redesign it.
- `reference/ai-process-spine.html` — source for stage names, ownership lines, and what each stage produces.

The reference governs the visual treatment, the stage names, and the substance of each stage — not the
wording of the descriptions. That wording was reused verbatim until the owner reported the artifact read
as too technical for a stakeholder to follow, and it was the reference copy, written for people who had
read the whole document set, that made it so. Stage descriptions are now written for a first-time reader.
Keep every claim the reference makes; do not keep its shorthand.

Nothing else in the wider document set is an input to this build.

## Precedence

`SPEC-machine.md` beats `SPEC-human.md` beats the reference files beats your judgement. Where the machine spec is silent and the human spec implies an answer, follow the human spec. Where both are silent, stop and ask rather than deciding — the model is small enough that every number in it is a claim someone has to defend.

## Stop conditions

Stop and report rather than continuing, in each of these cases:

- The file passes 800 lines including styles. (Raised from 600 to 700, then 700 to 800, by the owner: the
  cap existed to stop scope creep in the model, and was twice blocking presentation the artifact needed
  rather than doing that job. The model is smaller than it was at 600 lines.)
- The model produces a result that contradicts criteria A2 or A3 in `SPEC-tests.md`. Report the numbers; do not adjust the assumptions to make the criteria pass.
- Any part of the model requires a number that is not in `DEFAULT_ASSUMPTIONS` or the stage and defect tables.
- The interaction spec is ambiguous about what a control does.
- You find yourself wanting to add a feature that is not in either spec.

## How to work

Build in this order, verifying each part before moving on: model functions first, then the two lanes, then the tally, then the two decision points, then the assumptions panel, then the stage 9 record, then the quality floor items in section E.

Keep the model as pure functions taking `(state, assumptions)` and returning computed values. One `render()` reading from state. No incremental accumulation of totals anywhere — everything recomputes, because the assumptions panel changes inputs mid-run.

Do not write tests as files. `SPEC-tests.md` is checked by hand and by inspection; C2 in particular is a grep over the source.

## What not to do

The forbidden list in `SPEC-machine.md` is not advisory. Specifically:

- Do not tune any number to improve the assisted track's showing.
- Do not add copy that celebrates one process or disparages the other. Neutral throughout, including in variable names and comments.
- Do not de-emphasise or collapse the assumptions panel.
- Do not add persistence, analytics, network calls, or model calls.
- Do not change the defect table because a result looks bad. An uncomfortable result is the finding.

## Build log — maintain as you go

Append to `BUILD-LOG.md` while working, not afterwards from memory. This file becomes stage 9 of the presentation and is the only part of the exercise that is evidence rather than illustration.

Record:

- Which criteria in `SPEC-tests.md` passed on the first generated pass, unmodified.
- Which needed correction, what the correction was, and who caught it — you or the human reviewer.
- Anything that passed its own tests while being wrong. This is the `D2` failure mode occurring for real, and it is the most valuable entry in the log.
- Anything invented that was not in the specs: a value, an enum, a behaviour, a component name.
- Points where the specs were ambiguous or wrong, with what was missing.
- Actual review time, if the human reports it.

A log reporting a clean run is an incomplete log. If nothing went wrong, say what was not checked.

## Definition of done

`demo.html` exists in the repo root, opens from the filesystem with no server, satisfies every criterion in `SPEC-tests.md` or documents why not, and `BUILD-LOG.md` contains entries against every section above.
