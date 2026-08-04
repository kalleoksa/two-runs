# 5.3 — Testable spec

**A3 was deleted.** It asserted that the thin path lands within 10% of the conventional lane — a
conclusion written as a test. Satisfying it requires the control arm to ship four or five of its seven
defects, so it could only ever be met by fitting the model to it, which is what happened once and is
recorded in `BUILD-LOG.md` §8 and §14. Removed by its author rather than demoted, because a test that
asserts a conclusion is the failure this document set exists to argue against.

Criteria are written as invariants rather than fixed totals, because the assumptions are user-editable and any hardcoded number would be wrong the moment someone changes one. Invariants must hold for the default assumptions and for any assumption set where the multipliers keep their stated ordering.

---

## A. Model correctness

| # | Criterion |
|---|---|
| A1 | Determinism: same `probesKept` + `specLevel` + assumptions produces identical totals across reloads. No `Math.random`, no `Date.now` in any computation path. |
| A2 | Total effort ordering: `triad < human < thin`. |
| A4 | Escaped defect count: `triad === 0`, `human === 0` when probes deleted, `thin > 0`. |
| A5 | Keeping probes adds exactly one defect (`D9`), escaping to production, on every spec level. |
| A6 | Rework loops fire only where `caughtStage >= 6 && introducedAt <= 5`. Count is displayed and matches the arrows drawn. |
| A7 | Every defect appears in exactly one place in the stage 9 record, with introduced stage, caught stage, and cost. |
| A8 | `D2` and `D9` are absent from the conventional lane, and the UI says why where they appear. |
| A9 | Editing any assumption recomputes all downstream totals in the same frame. No stale values anywhere on screen. |
| A10 | Changing an assumption mid-run does not alter `stageIndex` or either choice. |

## B. Walkthrough behaviour

| # | Criterion |
|---|---|
| B1 | Next is blocked at stage 4 until the probe choice is made, and at stage 5 until the spec level is chosen, with a visible reason. |
| B2 | Back preserves both choices; Reset clears both and returns to stage 0 while keeping edited assumptions. |
| B3 | Defects introduced at stage 3 are not visible in the tally or the lane until the stage where they are caught. |
| B4 | Choice prompts contain no evaluative language. A reader must not be able to infer the recommended option from the button labels or the surrounding copy. |
| B5 | The conventional lane's **strip** is fully rendered from load and shows the whole run at every moment. Its **narrative and figures** depend on position in the walkthrough and on nothing else — never on either choice. Amended when the tally began showing both runs at the equivalent point: the original wording said "never changes", which conflated two promises and became untrue of one of them. The substance is unchanged — no part of the conventional lane responds to `probesKept` or `specLevel` — and G3 is the check. |
| B6 | Stage 9 record lists defects for the path actually taken, including any caught cheaply, not only the expensive ones. |

## C. Assumptions panel

| # | Criterion |
|---|---|
| C1 | Open on first load. |
| C2 | Every number used in any computation is present and editable. Grep test: no numeric literal in the model functions that is not sourced from `state.assumptions` or the stage/defect tables. |
| C3 | The word "assumed" appears on screen at all times, or within one interaction of any displayed figure. |
| C4 | Negative and non-numeric input is rejected; the last valid value is retained; no `NaN` reaches the display. |
| C5 | Reset assumptions restores defaults without disturbing run state. |

## D. Build constraints

| # | Criterion |
|---|---|
| D1 | Single file. Opens from the filesystem with no server and no network access. Verified with devtools offline. |
| D2 | No framework, no CDN reference, no build artifact. |
| D3 | No `localStorage`, `sessionStorage`, `indexedDB`, `fetch`, or `XMLHttpRequest` anywhere in the source. |
| D4 | No `<form>` element. |
| D5 | Under 800 lines including styles. Over that, stop and report rather than continuing. (Said 600 until this was noticed: `CLAUDE.md` and `SPEC-machine.md` had been raised to 700 and this line was not, so the three documents disagreed about the one constraint written in all three. Now 800 in all three.) |
| D6 | Colours and type come only from the token list in `SPEC-machine.md`. No hex value outside that list except the named segment colours. |

## E. Quality floor

| # | Criterion |
|---|---|
| E1 | Keyboard: all controls reachable by Tab, operable by Enter or Space, with a visible focus ring. |
| E2 | Stage advance is announced to assistive technology via a live region. |
| E3 | Respects `prefers-reduced-motion`; with it set, no transitions run. |
| E4 | Text contrast meets WCAG AA against `--paper` and `--panel`. |
| E5 | Renders correctly at 1280×800 and 1920×1080. Below 1024px wide it may show a single "desktop only" message rather than attempting to reflow. |

## F. Comprehension

Added after the owner reported that the artifact was too technical to follow — the fourth finding of that
shape (`BUILD-LOG.md` §9, §10, §13), and the fourth that every criterion above passed straight through.
A—E check that the figures are right and the controls behave. None of them ask whether the screen means
anything to someone who has not read this document set. These do.

| # | Criterion |
|---|---|
| F1 | No term appears on screen that a reader outside the team would not know and cannot find defined on screen. Grep list: `5.1`, `5.2`, `5.3`, triad, probe, escaped, rework, effort day, spec level, enum, handoff, orchestrate, guard metric. A hit is a pass only if the same screen defines it. |
| F2 | Every defect, wherever it is shown, carries one plain-language sentence naming who notices and what happens to them, alongside its id and engineering label. Present tense, so the sentence reads correctly whether the defect was caught at its stage or shipped. |
| F3 | Each of the two runs is described in one sentence saying what that process actually does, next to its strip, and the strips are explained before the reader meets them. |
| F4 | Stage 9 states in sentences, before the table, what was chosen, what came back, and how the totals compare — and reads equally flat when the assisted run is the more expensive one. |
| F5 | No copy states a magnitude. Every figure is one assumption edit away from changing, so copy names the assumption, never its current value. (§11 found this class; it applies to all narrative copy, not only the two definitions it was found in.) |
| F6 | B4 holds across all copy, not only the button labels: no label, prompt, description, or consequence sentence lets a reader infer which option is recommended, and the options stay parallel in length and register. |

## G. The two runs side by side

Added when the conventional run gained a narrative of its own, so the differences between the two
processes could be read rather than inferred from bar widths.

| # | Criterion |
|---|---|
| G1 | `ALIGN` covers every assisted stage and every conventional phase exactly once. Handoff is included and attributed to Specify. No phase is orphaned and no stage has an empty column. |
| G2 | The conventional figures are monotonic across the walkthrough and, at stage 9, equal the whole-run totals exactly — effort, calendar, review, found, escaped and loops alike. Anything less means part of that run is not being counted. |
| G3 | The conventional column's rendered text is byte-identical across all six choice paths at every stage. This is the automatable half of B5 and it is a script, not a reading. |
| G4 | Each column states where its narrative comes from, on screen, under the column heading. The conventional column is marked as a reconstruction wherever it appears. |
| G5 | Conventional escapes appear at Release, not at Retrospective — in the story column and in the tally alike. If the two regions disagree about when a problem reached a customer, one of them is lying. |
| G6 | No copy in either column states a count, a duration or a magnitude the model computes. The panels describe how the work goes; the numbers come from the model and appear once. |

## H. The run sheet

Added with `checklist.html`, which is the first file in this set allowed to give instructions. These
criteria exist because that permission is narrow and easy to widen by accident.

| # | Criterion |
|---|---|
| H1 | Every item on the sheet either cites the problem it prevents, or is marked *authored*. No item stands without provenance. |
| H2 | No figure anywhere in `checklist.html`: no days, hours, multipliers, effort counts or percentages. Stage numbers and problem ids are identifiers and do not count. Grep, not judgement. |
| H3 | Stage numbers and names match `ASSISTED_STAGES` exactly, and every cited problem id and label tag matches `DEFECTS`. Checked by script — two files naming the same things is the same drift risk that `conventionalIntroducedAt` and `ALIGN` were written out to avoid. |
| H4 | Prints without splitting a stage block, a decision box or a closing section across a page. |
| H5 | No `<script>`, no network reference, no storage. Opens from the filesystem. |
| H6 | The demo's crossover line recomputes with the assumptions and agrees with the totals at every setting — including the case where all six paths sit above the conventional total, and the case where none do. |
| H7 | The demo advises the reader nowhere, and the run sheet carries no figures. Precisely: `demo.html` may describe what a stage *consists of* in the imperative mood the process documents use — "agree what problem is being solved", "write the decision down" — because that describes the work rather than instructing the viewer. What it must never do is tell the reader what they should do about their own choices: no second-person prescription — "you should", "make sure", "be sure to", "we recommend", "you need to", "best practice" — and no evaluation of the option they picked. ("Always" and "never" are not markers: both appear descriptively throughout, as in "a flow that was never designed for them", and grepping for them only produces noise.) The stage 9 block reports what the model catches and where, and stops there. |

H1, H2, H3, H5 and H6 are scripts. H4 is a print-to-PDF and a read. H7 is a reading with a grep behind it —
for second-person prescriptions, not for imperative verbs, which the first draft of this criterion did and
which flagged all ten stage descriptions. The same disqualification applies as F1: the person best placed
to judge the boundary has not read these documents.

Neither F1 nor F6 can be automated into a pass or fail. The check for both is to read the page top to
bottom as though presenting to someone who has never seen the doc set, and list every sentence that would
need explaining aloud. An empty list is the pass condition.

---

## Failure states

| Condition | Required behaviour |
|---|---|
| Assumption field emptied | Retain last valid value in the model; show the field as invalid; do not compute with zero or `NaN`. |
| Multiplier ordering broken by an edit (e.g. `thin.build` set below `triad.build`) | Compute and display the result honestly, and show a note that the ordering assumption no longer holds so A2 and A3 will not. Do not silently correct the input. |
| Assumption set that makes the assisted path slower than conventional | Display it. This is a valid, informative result, not an error. |
| Next pressed while blocked | Inline reason next to the control. No modal, no alert. |
| Any thrown exception | Render a plain message naming the stage and choice state. Never leave a half-rendered screen. |

## Performance budget

Full recompute and re-render under 16ms on a mid-range laptop. The model is a few dozen arithmetic operations; if a profiler shows otherwise, something is being recomputed in a loop.

## Verification of the process, not the artifact

To be recorded during the build and used in stage 9 of the presentation:

- Which of these criteria were satisfied by the first generated pass, unmodified.
- Which required correction, and what the correction was.
- Whether anything was produced that passed its own tests while being wrong — the `D2` failure mode, occurring for real.
- Actual review time spent, against the estimate.

A build that reports no defects should be treated as an incomplete record rather than a clean run.
