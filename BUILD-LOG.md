# Build log — `demo.html`

Written while building, not reconstructed afterwards. This is stage 9 of the presentation and the only
part of the exercise that is evidence rather than illustration.

Builder: agent (`claude-opus-5`) driving the repo directly. Human review time: **not yet reported** —
this log is complete only once that number is in it.

---

## 1. The result that contradicts a criterion

**A3 fails on the default assumptions, and it is not close.** Reported rather than fixed, per the stop
condition in `CLAUDE.md`. No number in `DEFAULT_ASSUMPTIONS`, `DEFECTS`, `ASSISTED_STAGES` or
`CONVENTIONAL_PHASES` was changed.

| Path | Total effort, assumed days | vs conventional (64.9) |
|---|---|---|
| Conventional lane | 64.9 | — |
| Triad, probes deleted | 52.7 | −18.8% |
| Triad, probes kept | 65.2 | +0.5% |
| Human-readable, probes deleted | 76.9 | +18.4% |
| Human-readable, probes kept | 89.4 | +37.7% |
| **Thin, probes deleted** | **117.5** | **+81.0%** |
| Thin, probes kept | 130.0 | +100.3% |

A3 asks for the thin path within 10% of the conventional lane. It lands 81% above. A2 holds
(52.7 < 76.9 < 117.5), so the ordering is right; the *spread* is roughly eight times what A3 expects.

Cause, as far as inspection goes. `catchCostMultiplier` is keyed by **assisted** stage semantics —
5 is Specify (×2), 6 is Build (×5), 7 is Test (×10), 8 is production (×25). `conventionalCaughtAt` is a
**phase number in a different list**, where 5 is Build, 6 is QA and 7 is Release. Applying the same
table to both, which is what the machine spec instructs, prices a conventional Build-stage catch at the
assisted Specify-stage rate. Five of the seven conventional defects are caught at phase 5 and so cost
×2 each, which is why the control lane comes out cheap.

Under the other available reading — map each conventional phase to the assisted stage of the same
`kind`, so Build→6, QA→7, Release→8 — the conventional lane totals **89.7** and thin sits **+31.0%**.
Still outside A3, so the ambiguity is not what breaks the criterion; it only narrows the gap. The
implementation follows the machine spec as written ("the same functions with `conventionalCaughtAt`"),
because `SPEC-machine.md` has precedence.

What this means for A3 as a claim: no edit permitted by the specs closes an 81% gap. Either the defect
table's `conventionalCaughtAt` column is too favourable to the control lane (three "missing state"
defects, D4/D5/D6, are all caught at phase 5 there but escape to production on the thin path), or A3 is
asserting something the table was never built to produce. That is a decision for whoever owns the
model, not something to resolve by tuning.

---

## 2. Criteria satisfied by the first generated pass, unmodified

A1, A2, A4, A5, A6, A7, A8, A9, A10, B1, B2, B3, B4, B5, B6, C1, C2, C3, C4, C5, D1, D2, D3, D4, D6,
E3, E4, E5, and the performance budget. Verified by driving the built file in headless Chromium through
all six paths (2 probe choices × 3 spec levels) and asserting on the rendered DOM, not by reading the
source. Evidence:

- **A1** — a full walkthrough repeated after a page reload produced a byte-identical tally. No
  `Math.random` or `Date.now` anywhere in the file.
- **A4** — escaped counts: triad 0, human-readable 0 (probes deleted), thin 5. **A5** — keeping probes
  adds exactly D9, escaping to production, on all three levels (+1 escaped in every case).
- **A9/A10** — editing `defectBaseCost` moved the total 52.7 → 79.1 in the same frame, with
  `stageIndex` and both choices untouched.
- **C4** — emptying a field and entering `-3` both left every figure unchanged, marked the input
  invalid, and put no `NaN` on screen.
- **D1/D3** — a full walkthrough issued zero non-`file:` requests. No storage or network API appears in
  the source.
- **E4** — every rendered text node was measured against its painted background: no contrast failure at
  AA. This required deviating from the reference files; see §5.
- **E5** — no horizontal overflow at 1280×800 or 1920×1080; below 1024px the app is replaced by a
  single "desktop only" line.
- **Performance** — mean full recompute + re-render over 200 iterations: 2.6ms at 1280×800, 2.9ms at
  1920×1080, against a 16ms budget.

**C2, stated precisely rather than as a pass.** The grep over the model functions returns only `0`, in
three places: two `reduce` seeds and the `work`/`review` initialisers. No cost, effort, multiplier or
threshold magnitude appears as a literal. Stage thresholds are derived from the stage table at load
(`SPEC_STAGE`, `BUILD_STAGE`, `PROD_STAGE`, `DECIDE_STAGE`, `LAST_STAGE` are looked up by `kind`/`key`,
so the rework rule reads `caught >= BUILD_STAGE && introducedAt <= SPEC_STAGE` rather than `>= 6` and
`<= 5`). Formatting and layout functions do contain literals — rounding factors, label-width thresholds
— which C2's wording does not cover.

**D5** — 542 lines including styles. Over the ~400 target, under the 600 stop line.

---

## 3. Defects in the build, and who caught them

Four. All mine; the human reviewer has not been through it yet.

1. **Crash on load: `a.specMultipliers[null].reviewRate`.** The review-hours branch dereferenced the
   spec multipliers for build/test stages before a spec level existed, so the page died at stage 0 —
   before the choice that supplies the level. Caught by opening the file in a browser, not by
   inspection or by reasoning about the code.

   Worth recording: what I actually saw was the spec's own required behaviour working. The failure
   state "any thrown exception → render a plain message naming the stage and choice state" printed
   *"The model could not be rendered. Stage 0, probes undecided, spec unchosen. Cannot read properties
   of undefined (reading 'reviewRate')"*. The message named the exact precondition I had missed. A
   criterion written for the viewer's benefit paid for itself on the builder instead.

2. **The D2 failure mode, occurring for real.** `drawTally` emitted six `.tile` elements into
   `<div id="tally">` — which never had `class="tally"`, so the six-column grid rule matched nothing and
   the tally rendered as a single vertical column a full screen tall. Every assertion I had written
   against the tally passed, because they all read `.tile .big` text content, and the *numbers were
   correct*. Nothing in `SPEC-tests.md` constrains the tally's layout either. Internally consistent,
   self-verified, and wrong — caught only by taking a screenshot and looking at it.

   This is the entry the log exists for. The check that found it was a human-style one (look at the
   thing), and it found a defect that the model, its tests, and the criteria all agreed was absent.

3. **Keyboard focus destroyed on every advance (E1).** `render()` replaces `#stage.innerHTML`, which
   discards the focused button and drops focus to `<body>`. Tab and Enter worked once; the second
   activation went nowhere. My first test run reported "Space on Next → stage 1" and I read it as a
   Playwright quirk before checking — the correct reading was that a keyboard user has to re-Tab from
   the top of the page after every single stage. Fixed by capturing a focus key before the redraw and
   restoring it after, falling back to the first enabled control in the stage panel when the previous
   one is gone or disabled (which is what happens on arriving at stage 4, where Next is blocked).

   Note the tension: "one `render()`" as an architecture and "all controls operable by keyboard" as a
   floor do not compose for free. The spec asks for both and says nothing about the interaction.

4. **Segment labels clipped at both ends.** Centre-aligned labels in narrow proportional blocks were
   trimmed on both sides, rendering "Concept & probe" as "ACCEPT & PROBIC". Changed to left-aligned,
   and the stage name and figure are now dropped below width thresholds rather than sliced. Caught by
   screenshot. Two invented thresholds; see §4.

---

## 4. Invented — not in any spec

Every item here is a decision I made where the specs were silent, and every one is a claim someone
should either ratify or overrule.

- **Lane scaling (`scaleMax`).** Nothing says how two lanes of different totals share a horizontal
  scale, and B5 forbids the conventional lane changing in response to choices — so a scale that grows
  with the assisted run is not available. I fixed the scale at the maximum of the conventional total and
  the worst-case assisted total across all three levels with probes kept. It depends only on
  assumptions, so it is stable across the walk and re-derives when an assumption is edited.
- **Stage base efforts are editable, by mutating the stage tables.** `SPEC-human.md` says the panel
  lists stage base efforts and all of it is editable; `DEFAULT_ASSUMPTIONS` in `SPEC-machine.md` does
  not contain them, and machine has precedence on state shape. Rather than invent assumption keys, the
  18 stage/phase efforts are edited in place in the tables, with `Reset assumptions` restoring a
  snapshot taken at load. Both readings are satisfied; neither is satisfied exactly as written.
- **Light vs dark segment label colour**, chosen per stage kind, and `var(--panel)` in place of the
  reference's `#fff`. The reference files put white text on `#7C8798`, `#9AA3B0` and `#B08A3E`, which
  is 2.9:1 — E4 fails if the reference is matched literally. Those three take `--ink` instead. `#fff`
  would also be a hex outside the token list, which D6 forbids, so `--panel` stands in for white.
- **`filter:grayscale(.5)` for "pre-filled and greyed".** Mechanism invented; verified not to break AA
  on any segment.
- **"Defects open" defined as** surfaced at or before the current stage and not escaped, with that
  definition printed under the figure. `SPEC-human.md` names the row and never says what "open" means
  in a model where a defect is found and paid for in the same stage.
- **Where the triad's cheap catches are costed.** D4 and D5 are caught at stage 3 on the triad path,
  but the spec level is not chosen until stage 5, so they cannot surface when the viewer passes stage 3.
  They appear at stage 5 with their cost attributed to stage 3, and the record says so. The
  alternative — costing them where they appear — would have hidden the fact that the triad's value is a
  stage-3 catch.
- **Review hours use the multiplied build/test effort, excluding defect and rework cost.** "sum over
  build/test stages of `effort * ...`" does not say which effort. The conventional lane gets no review
  rate at all, on the machine spec's "no spec multipliers".
- **Focus restoration after render** (§3.3), and **label-width thresholds** of 6% and 2.5% of the scale
  for dropping the stage name and the figure from a segment (§3.4).
- **All UI copy**: the two probe descriptions, the three spec-level descriptions, the two blocked-Next
  reasons, the `NO_CONVENTIONAL` sentences A8 requires, the ordering-broken warning, tally sub-labels.
  Stage names, ownership lines and "Out" lines are reused from `reference/ai-process-spine.html` and the
  worked example, as instructed. The invented copy was written to B4 — neutral, described by what gets
  written rather than by outcome — but B4 is a judgement, and a reader should check it rather than take
  my word for it.

**Not built:** the P2 third decision at stage 6 (delegate the billing logic or write it by hand).
`SPEC-human.md` marks it optional and "build it last or not at all". Skipped — with A3 unresolved,
adding a fourth path to the model would be adding to something whose central claim does not hold.

---

## 5. Where the specs were ambiguous or wrong

- **A3 is not achievable** from the given tables under either reading of the conventional lane. §1.
- **`catchCostMultiplier` is keyed to one lane's stage semantics and applied to both.** §1. This is the
  substantive spec defect found in this build.
- **"Stage/phase 8 means it escaped to production"** contradicts `CONVENTIONAL_PHASES`, where `n:8` is
  Retrospective. Moot in practice — no conventional defect is caught at 8 — but a defect at phase 8
  would be simultaneously "escaped to production" and "caught at the retrospective".
- **`catchCostMultiplier` has no keys for stages 0–2** or for conventional phases 1–2. Nothing in the
  tables reaches them, so the model indexes directly and would produce `NaN` if a future defect were
  caught there. Left as-is: adding a fallback would have put a numeric literal in a model function and
  broken C2, and inventing keys would have added numbers the specs do not have.
- **Assumptions panel scope**: human spec says stage base efforts are in it and editable, machine spec's
  state shape has no room for them. §4.
- **"One `render()`" vs E1/E2.** Rebuilding `innerHTML` is incompatible with retaining keyboard focus
  unless focus is explicitly restored. Neither spec mentions it.
- **E4 vs the reference files.** "Match it; do not redesign it" and "text contrast meets WCAG AA" are in
  direct conflict on three of the eight segment colours. Resolved toward E4.
- **D2's label** is "Invented plan status enum"; the worked example's narrative calls it a subscription
  status. Cosmetic, unchanged.
- **A6 says the displayed loop count "matches the arrows drawn".** Arrows can only be drawn between
  stages that are on screen, so during the walk the count and the arrows agree only because no rework
  fires before both its endpoints are revealed. It holds, but by circumstance rather than by design.

---

## 6. What was not checked

A log reporting a clean run is an incomplete log, so here is what is missing rather than passing.

- **E2 was checked structurally, not for real.** The live region exists, is `aria-live="polite"`, and
  its text updates to "Stage N of 9, <name>." on every advance. No screen reader was run. Whether the
  announcement is *useful* — or whether it collides with the stage panel being replaced wholesale — is
  unverified.
- **B4 (no evaluative language) cannot be tested by a machine** and was not reviewed by a second
  person. I wrote the copy and I am asserting it is neutral, which is the weakest evidence in this file.
  The 5.2 and 5.3 descriptions in particular name document numbers a viewer of the spine deck may
  already read as the recommended answer.
- **B6** was verified by reading the stage 9 table on the thin path (all 8 defects listed, cheap ones
  included) but not asserted programmatically across every path.
- **No human has opened the file.** Everything above is the builder checking its own work, which is
  the failure mode §3.2 documents.
- **No accessibility rule engine** (axe or equivalent) was run — the contrast check was hand-written and
  covers contrast only, not roles, names or reading order.
- **The 5.3 criteria themselves were not reviewed** for whether they are the right criteria. A3 is the
  one that failed loudly; a criterion that is simply too weak would not have announced itself.

---

## 7. Verification of the process, not the artifact

The exercise's own claim is that the spec triad makes downstream work checkable. On this build:

- The two defects that a written criterion caught (the crash, via the required failure-state behaviour;
  the focus loss, via E1) were caught **because someone had written down what correct looked like** —
  before the code existed, by a different party than the one building.
- The one defect that mattered most visually (§3.2, the tally) was caught by **none** of it. The model
  was right, the assertions were right, the criteria did not cover it, and it took a human-shaped act —
  render it and look — to find. The generated tests were consistent with the generated code, which is
  the D2 failure mode exactly, occurring inside a demo built to illustrate the D2 failure mode.
- The criterion that failed (A3) failed on **arithmetic that could have been checked before any UI was
  written**, and was. Fifteen minutes of running the model in isolation, ahead of the build, is what
  turned "the demo's central claim does not hold" from a late surprise into a first-day finding. That
  is the stop rule working, and it is the cheapest thing in this log.
