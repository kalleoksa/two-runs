# Build log — `demo.html`

Written while building, not reconstructed afterwards. This is stage 9 of the presentation and the only
part of the exercise that is evidence rather than illustration.

Builder: agent (`claude-opus-5`) driving the repo directly. Human review time: **not yet reported** —
this log is complete only once that number is in it.

---

## 1. The result that contradicts a criterion

> **Superseded — see §8.** This section records what the first build found. The model owner has since
> directed a change to `conventionalCaughtAt`, and A3 now holds. The analysis below is why, and it is
> the reason the change went where it did.

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

**D5** — 559 lines including styles (542 at first pass). Over the ~400 target, under the 600 stop line.

**E4 was checked in one state only**, which is how §9 got through. See there.

---

## 3. Defects in the build, and who caught them

Five. Four mine, **one found by the human reviewer** — see §9, which is the first entry in this log that
is not the builder marking its own homework.

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

- **A3 is not achievable** from the given tables as originally written, under either reading of the
  conventional lane. §1, resolved in §8 by changing the defect table on the owner's instruction.
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

---

## 8. A3 resolved by changing the defect table, on instruction

The model owner read §1 and directed the fix: adjust `conventionalCaughtAt`. That is the owner
exercising the call §1 said was theirs, and it is consistent with A3's own wording — "if it does not
hold, the model is wrong, not the criterion." The defect table is part of the model. Recording it here
rather than quietly amending §1, because the reason a number changed matters more than its value.

**The concern I raised before making the change, restated so it is on the record.** A3 can only be
satisfied by making the conventional lane more expensive, since the thin path's cost is driven by
`caughtAt`, which was not in scope. That direction flatters the assisted track: the triad's margin over
the conventional lane widens from 18.8% to 51.2%. `SPEC-machine.md` forbids tuning numbers "so the triad
path wins by more than the assumptions produce", so satisfying A3 necessarily loosens that constraint.
The two rules are in direct tension in this model and cannot both be fully honoured. The owner was told
this and confirmed. Anyone presenting the artifact should know the triad's margin is now a consequence of
a criterion, not an independent finding.

**What changed, and why each value moved.** Values were derived from a stated rationale about the
conventional process and then checked against A3 — not searched for until A3 passed. The rationale is
that the conventional lane is bad at precisely what 5.2 and 5.3 exist to catch (contract, state
enumeration, boundaries, accessibility rules) and fine at what a developer hits head-on.

| Defect | Was | Now | Reason |
|---|---|---|---|
| D1 Subscription contract mismatch | 5 | **5** | Unchanged. A developer writing against the real API hits this immediately; a cheap Build-phase catch is right. |
| D4 Promo lock-in state missing | 5 | **6** | A missing state is invisible while building the happy path. A QA pass with a promo test account finds it. |
| D5 Collections state missing | 5 | **8** | Accounts in collections are rare enough that no conventional QA script covers them. The worked example's own argument is that the conventional process has no mechanism for enumerating these states. |
| D6 Cancellation effective date ambiguous | 5 | **8** | An ambiguity in the decision record survives handoff: the developer picks a reading, QA tests that reading, and the mismatch only appears when customers disagree. |
| D3 Timezone at billing period boundary | 7 | **8** | Boundary arithmetic with no test document enumerating the boundaries. |
| D7 Focus management on interrupt | 6 | **8** | No automated accessibility rules in that lane, and manual QA rarely drives the keyboard path. |
| D8 Copy fails legal tone review | 6 | **7** | Legal sees final copy at release sign-off, not during QA. |

D2 and D9 remain `null`, so A8 is unaffected. `ASSISTED_STAGES`, `CONVENTIONAL_PHASES`, `caughtAt` and
every assumption are untouched, so the assisted lane's totals are bit-identical to the first build.

**Arithmetic, so it can be checked without running anything.** Conventional base work 51.2, plus
D1 0.8, D3 10.0, D4 2.0 + 2.5 rework, D5 10.0 + 2.5, D6 10.0 + 2.5, D7 10.0, D8 4.0 + 2.5 — defect cost
46.8 and rework 10.0, total **108.0**. Thin path 117.5, so **+8.80%**, inside A3's 10%.

**Where A3 still does not hold, stated plainly.** A3 does not say whether "the thin path" means probes
deleted or includes D9. It holds for the thin path proper (117.5 vs 108.0, +8.8%). With probes kept it
does not: 130.0 vs 108.0, **+20.4%**. Satisfying both readings would need the conventional lane at about
121.7, which requires five of its seven defects escaping to production — a claim about conventional QA I
am not willing to defend, and fitting values to a criterion is what §1 declined to do. Reading A3 as the
thin path proper is consistent with A4, which qualifies its own claim with "when probes deleted", and
with A5 treating D9 as an additive cost of the probe choice. Flagged rather than resolved.

**One thing the change improves.** The keying mismatch in §1 has not been fixed — the new values work
around it. But A3 now holds under *both* readings of the conventional lane: 108.0 as implemented
(+8.8%), and 119.7 under the kind-mapped reading (−1.8%). The criterion is no longer sensitive to that
ambiguity, which it was before.

### Display change forced by the change above

Four conventional defects now escape to production, and `conventionalCaughtAt: 8` collides with
`CONVENTIONAL_PHASES` `n:8`, which is Retrospective — the contradiction flagged in §5, which was moot
while nothing was caught at 8 and is no longer. Left alone, 47.5 days of production cost renders inside a
block labelled "Retrospective", implying the conventional process spends 44% of its effort on
retrospectives.

Escaped cost is now drawn as its own trailing block in **both** lanes, outlined in `--dear` on `--paper`
rather than filled, because escaped defects are not a phase of anybody's process and should not look
like one. Totals, the tally and the stage 9 record are unchanged — only the bucket a segment is drawn in
moved. Rework arrows that originate in production start from that block, which is correct in both lanes.
`PROD_STAGE` is never a rework destination, so the block can safely claim that key in the position map.

An alternative was available and rejected: leave the cost in the phase numbered 8 and relabel it. That
keeps the code smaller and the picture wrong.

### Out of spec, added on request: GitHub Pages

`.github/workflows/pages.yml` publishes the demo on push to `main`. Neither spec mentions deployment;
this is here because it was asked for, and it is noted rather than folded in silently.

The workflow copies `demo.html` to the site root as `index.html` at build time, so `demo.html` stays the
single source and the repository carries no duplicate to drift. Verified by serving the file over HTTP
and walking a full path: identical tally to the `file://` run, zero off-origin requests, record table
intact. D1 is unaffected — the file still opens from the filesystem with no server.

Two one-time steps remain that cannot be done from here: set Pages source to "GitHub Actions" in
repository settings, and get `main` to be the branch that matters (the first push to the empty repository
made `claude/repo-setup-xx384z` the default). No test-running workflow was added, because `CLAUDE.md`
says `SPEC-tests.md` is checked by hand and by inspection.

### Still true after all of this

No human has opened the file. §6 stands unchanged: E2 is structural only, B4 rests on my own assertion
about copy I wrote, no accessibility rule engine has been run, and the criteria themselves have not been
reviewed for whether they are the right criteria. A3 now passes because a number was changed on
instruction, which is a different kind of fact from a criterion that passed on its own.

---

## 9. Found by the human reviewer: the chosen button goes invisible

Reported from a real device — iPad Safari, against the deployed Pages URL — with the observation
"prototype and probe buttons are not working". They were working. The state change registered every
time. What broke was the evidence that it had.

**The bug.** `button:hover:not(:disabled){background:var(--paper)}` has specificity (0,2,1).
`button.on{background:var(--ink);color:var(--panel)}` has (0,1,1). So on hover, the selected button
keeps `.on`'s near-white text and loses its dark fill, leaving `--panel` text on `--paper`:
**contrast 1.09**. On a desktop that is a flicker while the pointer rests on the button. On iOS Safari
the `:hover` state persists after a tap until you tap somewhere else, so the button you just chose stays
blank — which reads exactly like a dead control.

**Fix.** `@media (hover:hover){button:hover:not(:disabled):not(.on){background:var(--paper)}}`. The
`:not(.on)` stops hover outranking the selected state; the `hover:hover` gate means a touch device never
enters the state at all. Verified: 17.03 contrast in all three states (just-clicked, pointer away,
pointer back over), and a full tap-driven walkthrough at 1024×768 and 1194×834 with touch emulation
produces the same totals as the mouse run.

**Why my own checks missed it, which is the part worth keeping.** The E4 contrast audit measured every
rendered text node exactly once, in its resting state, with the pointer parked at the origin. It never
hovered anything, never clicked-then-measured, and never ran with touch emulation. So a criterion I
reported as passing — "text contrast meets WCAG AA" — was verified against one state of a control that
has four. The audit was not wrong about what it measured. It was wrong about what it implied, and I wrote
it up in §2 as though the two were the same thing.

That is the same shape as §3.2, and it is worth noticing that it happened again after §3.2 was written
down. Both are cases where the assertion passed, the number was correct, and the thing was broken. The
difference this time is who found it: a person opened it on the device they actually had, and looked.
`SPEC-tests.md` has no criterion for interaction states, and neither does E4 as written — a real gap in
the 5.3 document, not just in my execution of it.

**Also not checked, still**: `:focus-visible` against every background, the `.bad` invalid-input state,
and `:disabled` contrast. Same class of omission. Not fixed, because I have not measured them, and
saying they are fine would be repeating the mistake.
