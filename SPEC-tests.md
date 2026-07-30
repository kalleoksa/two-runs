# 5.3 — Testable spec

Criteria are written as invariants rather than fixed totals, because the assumptions are user-editable and any hardcoded number would be wrong the moment someone changes one. Invariants must hold for the default assumptions and for any assumption set where the multipliers keep their stated ordering.

---

## A. Model correctness

| # | Criterion |
|---|---|
| A1 | Determinism: same `probesKept` + `specLevel` + assumptions produces identical totals across reloads. No `Math.random`, no `Date.now` in any computation path. |
| A2 | Total effort ordering: `triad < human < thin`. |
| A3 | The thin path total lands within 10% of the conventional lane total. This is the argument of the whole artifact; if it does not hold, the model is wrong, not the criterion. |
| A4 | Escaped defect count: `triad === 0`, `human === 0` when probes deleted, `thin > 0`. |
| A5 | Keeping probes adds exactly one defect (`D9`) on every spec level. Where it is caught depends on the spec level chosen — the fuller the spec, the earlier it surfaces — so the two decisions interact rather than sum. `D9` escapes to production on the thin path only. |
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
| B5 | The conventional lane is fully rendered from load and never changes in response to viewer choices. |
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
| D5 | Under 600 lines including styles. Over that, stop and report rather than continuing. |
| D6 | Colours and type come only from the token list in `SPEC-machine.md`. No hex value outside that list except the named segment colours. |

## E. Quality floor

| # | Criterion |
|---|---|
| E1 | Keyboard: all controls reachable by Tab, operable by Enter or Space, with a visible focus ring. |
| E2 | Stage advance is announced to assistive technology via a live region. |
| E3 | Respects `prefers-reduced-motion`; with it set, no transitions run. |
| E4 | Text contrast meets WCAG AA against `--paper` and `--panel`. |
| E5 | Renders correctly at 1280×800 and 1920×1080. Below 1024px wide it may show a single "desktop only" message rather than attempting to reflow. |

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
