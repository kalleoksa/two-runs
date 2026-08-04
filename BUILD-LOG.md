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

**D5** — 612 lines including styles (542 at first pass). Over the ~400 target, under the 700 stop line
(raised from 600 by the owner; see §15).

**E4 was checked in one state only**, which is how §9 got through. See there.

---

## 3. Defects in the build, and who caught them

Six. Four mine, **two found by the human reviewer** — see §9 and §10, the first entries in this log that
are not the builder marking its own homework.

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

> **Reverted — see §14.** These values shipped and were later reverted to the spec table. The reasoning
> below stands as the record of why they were chosen; the decision to make them was the prohibited move.

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

---

## 10. Found by the human reviewer: the conventional lane's rework arrows pointed nowhere

Reported as comprehension trouble — "this tool is a bit hard to understand, I don't get the idea behind
conventional run reworks" — which turned out to be the correct reading of a picture that was lying.

**The bug.** `introducedAt` is a single value per defect, written in **assisted-stage** terms. The arrow
renderer used it as an index into whichever lane it was drawing, so in the conventional lane it selected
a phase by number from a list that numbers entirely different things:

| Arrow said | Meant, in assisted terms | Actually pointed at |
|---|---|---|
| D4 rework to 3 | 3 Concept & probe | 3 Design to spec |
| D6 rework to 4 | 4 Decide | 4 Handoff |
| D8 rework to 5 | 5 Specify | 5 Build |

The rework *count* and *cost* were right, so nothing I had asserted failed. Only the arrows were wrong,
and since both lanes render small bare numbers with no indication that they use separate numbering, a
reader had no way to detect it. The reviewer could tell something was incoherent but not what.

**The spec gap underneath it.** `SPEC-machine.md` provides `conventionalCaughtAt` but no
`conventionalIntroducedAt`. There is no data for where a defect is introduced in the conventional lane.
`CLAUDE.md` says that where both specs are silent I should stop and ask rather than decide — and this is
precisely a case where I did not. I silently reused the assisted number as if it were a conventional
one. That is the third invented answer in this log, and the only one invented without noticing.

**Resolution, chosen by the model owner** from three options put to them: map the assisted `introducedAt`
to the conventional phase sharing its `kind`. Concept & probe and Decide both map to 2 Concept, Specify
to 3 Design to spec, Build to 5 Build. Every `introducedAt` in the table maps cleanly. This invents one
structural rule rather than seven numbers, and it is checkable by inspection. The two rejected options
were dropping the arrows from that lane entirely (invents nothing, but the conventional lane stops
showing where its rework goes) and adding a seventh column to the defect table (most precise, seven new
claims to defend).

Arrows in **both** lanes now name their destination — "D4 reopens 2 Concept" rather than "D4 rework to 3"
— which removes the cross-lane numbering ambiguity that hid the bug. Verified by measuring each arrow's
left edge against the segment rectangles in both lanes: every arrow now lands on the block it names.

**What this says about the artifact as a presentation object.** Two of the three reviewer-facing problems
found so far (§9, §10) were invisible to a passing criteria suite and obvious to a person looking at the
screen for thirty seconds. Neither is in `SPEC-tests.md`, and neither could be. The 5.3 document covers
the model thoroughly and the rendered picture barely at all — it constrains numbers, not whether the
drawing of them means anything.

---

## 11. Two terms the artifact never defined

Not a defect, and not in either spec. Added on the owner's instruction after they asked, in consecutive
rounds, what "rework loops" and "escaped" meant. `CLAUDE.md` lists wanting to add something absent from
both specs as a stop-and-ask, so it was put to them rather than assumed.

Two questions in a row about load-bearing vocabulary is the useful signal here. Both words carry the
model's whole argument — the cost of a defect is a function of where it was caught, and those two terms
name the two ends of that scale — and neither appeared anywhere on the page. A presenter would have had
to define them aloud every time, which means the artifact was not carrying its own weight. `SPEC-human.md`
specifies three screen regions and their contents; it never asks whether a first-time reader can decode
what is in them, and no criterion in `SPEC-tests.md` could have failed here.

Added: a two-line definition under the lanes, and the Escaped tally tile now reads "escaped to production"
rather than "found in production".

One thing worth noting about writing the copy. Both definitions were first drafted with claims that
would have gone stale the moment a viewer edited an assumption — "charged at the largest multiplier in
the table" is true of the defaults and false as soon as someone changes them. Rewritten to name the
assumption instead of its current value ("the assumed production multiplier", "the assumed rework loop
cost"). On a screen whose entire premise is that every number is editable, any copy that states a
magnitude is a latent bug. That constraint is not written down anywhere in the specs either.

Verified: 577 lines, still under the 600 stop line. Full criteria suite re-run with no regressions,
contrast clean including the new text, "assumed" now appears 21 times on screen.

### §9 and §10 confirmed fixed on a real device

The reviewer sent a screenshot of the deployed page showing the selected choice button holding its dark
fill on iPad Safari, and the conventional lane's arrows reading "D4 reopens 2 Concept" and landing on the
Concept block. Both fixes are confirmed by observation rather than by my inference from matching bytes —
which matters, because the session's egress policy blocks the Pages domain, so every claim I have made
about the live page has been about the file rather than the page.

---

## 12. The first challenge to what the model argues, not to how it looks

> **Reverted — see §14.** `D9` is back to `{triad:8, human:8, thin:8}`. The argument recorded here was
> accepted and is still, in my view, correct; it was reverted because it breaks A5 and removes the
> consequence of the stage-4 decision. The conflict is unresolved, not settled.

Everything in §1 through §11 was about execution: wrong arrows, invisible buttons, an unachievable
criterion. This entry is different. The reviewer challenged a premise, was right, and the model changed.

**The argument, as they made it.** Keeping the prototype brings no value if the full triad already holds
all the necessary information — and there is a case that starting from scratch is *better* with agents
against a good spec, not merely equivalent.

**Why it holds.** If 5.2 specifies the input contract, expected outputs, component names, repo
conventions and what must not be touched, the prototype's informational value is already captured. What
remains is code, and inherited code is not obviously an asset when the binding constraint is review
rather than generation: reviewing code generated against a spec is bounded by the spec, while reviewing a
modified prototype means also auditing everything the prototype already did, which nobody wrote down. A
prototype is a pile of undocumented decisions, which is the thing 5.2 exists to remove. And its structure
constrains the build even where the spec disagrees, invisibly in a diff.

**What I had said before, and withdraw.** In answering an earlier question I offered the caveat that
"reusing a high-fidelity prototype plausibly saves some build effort, and the model gives it no credit",
and framed the flat 12.5-day penalty as the model being one-sided. That caveat was the weaker position
and I should not have offered it as a balance. The reviewer's argument is the stronger one.

**What neither of us can supply.** The reviewer noted that some sources argue for starting from scratch
with current models. I could not produce those sources, and did not try to reason as though I had: the
mechanism above is an argument, not evidence. That is the artifact's own footnote applying to the people
building it — a simulation of AI benefits, built with AI, argues in a circle.

### The consequence: the model was wrong in the opposite direction

The argument does not just remove a phantom upside. It says the *downside must vary with the spec level*,
and `D9` flatly refused to. Its `caughtAt` was `{triad:8, human:8, thin:8}` — keeping the prototype cost
exactly 12.5 days whether you had written a full triad or thin annotations. By the logic of the rest of
the artifact that cannot be right: against a triad the prototype is redundant, while against thin
annotations the prototype *is* the contract.

Changed to `{triad:6, human:7, thin:8}`, on the owner's instruction. Each value has a reason:

| Level | Caught at | Why |
|---|---|---|
| Full triad | **6 Build** | 5.2 names components, conventions and what must not be touched, so prototype code that violates them is visible in build review. |
| Human-readable | **7 Test** | 5.1 describes flows and states but no contract and no scenarios, so the mismatch surfaces when a person tests behaviour against it. |
| Thin | **8 production** | Nothing exists to check the prototype against; its decisions are the contract, so nothing catches it. |

**Effect.** The cost of keeping the prototype now ranges 4.5 → 6.5 → 12.5 days, a 2.8× spread, and the
two viewer decisions **interact** rather than sum. That is a materially more interesting claim than the
flat penalty, and it is the first time the model says something that could not be read off either
decision alone.

| Path | Probes deleted | Probes kept | Cost of keeping |
|---|---|---|---|
| Full triad | 52.7 | 57.2 | +4.5 |
| Human-readable | 76.9 | 83.4 | +6.5 |
| Thin | 117.5 | 130.0 | +12.5 |

### Two spec documents changed, which is worth stating plainly

This is the first change that could not be made in `demo.html` alone.

- **`SPEC-machine.md`** — `D9`'s `caughtAt` updated, plus a paragraph explaining why it varies. The
  defect table is the authoritative contract; leaving it stale would have made the spec fiction.
- **`SPEC-tests.md`** — **A5 as written was false** after this change. It said "escaping to production, on
  every spec level". Reworded to state that the catch stage depends on the spec level and that `D9`
  escapes on the thin path only.

`A4` was left alone: "`triad === 0`, `human === 0` when probes deleted, `thin > 0`" is still true, and is
now satisfied *unconditionally* for triad and human rather than only when probes are deleted. The "when
probes deleted" qualifier existed because of the old flat `D9` and is now vestigial. Not edited, because
it is not wrong.

Verified after the change: A2 holds on both probe branches (57.2 < 83.4 < 130.0 and 52.7 < 76.9 < 117.5),
A3 unchanged at +8.80%, A6 rework fires correctly on all three levels with `D9` reopening stage 3, A7 one
row per defect, A8 unaffected. 577 lines. Contrast clean, keyboard clean, 2.9ms recompute.

**The honest caveat on the new numbers.** 6, 7 and 8 are three invented values with a stated rationale
and no evidence, exactly like every other number in this file. What changed is not that the model became
true — it became *interesting*, and its shape now matches an argument someone made out loud and can be
disagreed with. That is a better position than a flat penalty nobody could interrogate, and it is still
not measurement.

---

## 13. The artifact never said what the project was

Reported by the reviewer: *"we need to have a story of how cancellation flow will be designed and
developed. This is a bit too abstract to understand without the context."* Checked before answering, and
they were describing a real hole rather than asking for polish.

`SPEC-machine.md` opens by defining the artifact as a simulation "comparing two delivery processes **on
one brief**". The brief appeared nowhere on screen. A grep for the project's own subject matter — cancel,
subscription, refund, churn — returned two hits, and both were incidental, inside defect labels. A viewer
met "D4 Promo lock-in state missing" with no way to know what product has promos, why cancellation was
being built, or who was building it. Every number on the page was attached to a project the page never
named.

Nothing in `SPEC-tests.md` could have caught this. The criteria check that figures are right, that
controls block correctly, that contrast passes. None of them ask whether the screen means anything to
someone seeing it for the first time. That is now the third reviewer finding of this shape (§9, §10, §13),
and the pattern is worth stating: **every defect a criterion caught was mine to make, and every defect
about whether the thing communicates came from a person looking at it.**

**Fixed by reuse, not by writing.** Both pieces of copy already existed in
`reference/ai-worked-example.html`, which `CLAUDE.md` names as a source to match rather than redesign:

- A brief box above the lanes — project, why, team, and the "not why" line stating that AI is not the
  reason for the project. Styling lifted from the reference rather than invented.
- A fourth field per stage, rendered as "On this brief" between the generic stage description and its
  output. The generic description says what the stage *is*; the new one says what happened *here* —
  8,000 tickets clustered and hand-checked at Research, the 40-scenario edge sweep at Concept, the
  retention-offer decision and its reason at Decide, proration written by hand at Build.

The payoff is that the defect table stops being arbitrary. "Promo lock-in" and "account in collections"
now appear in the stage 3 narrative as scenarios the sweep produced, so when D4 and D5 surface later the
viewer has already met them.

### Against the line budget, and close to it

**596 lines. The stop condition is 600.** Four lines of headroom, and this entry exists partly to say
that out loud: the file is now effectively at its ceiling, and the next content addition of any size
needs the budget revisited rather than quietly absorbed. `SPEC-machine.md` targets ~400 lines and says
that past 600 the model has grown beyond the spec — it has not, but the *presentation* has, which is a
distinction the constraint does not draw.

The cost was kept down by extending the existing `COPY` table with a fourth element rather than adding a
structure, and by reusing the `.out` block for the narrative instead of styling a new one. One line went
to `.out+.out{margin-top:8px}` after a screenshot showed the two blocks merging into what read as a
single box.

Model untouched: all six paths produce identical totals to before, determinism holds, contrast clean,
2.9ms recompute.

**A test-script error worth recording, since it is the same failure in miniature.** My first screenshot
script clicked Next five times before making the probe choice, and timed out on a disabled button. The
app was right and my script was wrong — B1 blocking exactly as specified. I misread it as a fault for a
moment before reading the log. Third time in this build that I have suspected the artifact before
suspecting my own check of it.

---

## 14. The reviewer filed a bug that was not one, and the log is why it did not get "fixed"

The reviewer reported the conventional lane's 4 escaped defects as a calculation bug — hypothesising that
`conventionalCaughtAt: null` was being read as "escaped" — and asked for it to be fixed first and written
up as the most valuable entry in this file, an instance of the D2 failure mode occurring for real.

It was not a bug. Audited before touching anything, because writing up a defect that did not happen
would itself have been the D2 failure:

- `activeDefects` filters `conventionalCaughtAt !== null` for the conventional lane. D2 and D9 are
  **absent** from it, not escaped. A8's "does not occur in the conventional run" annotations are intact.
- Escape is `caught === PROD_STAGE`, and the four escapes were D3, D5, D6 and D7 carrying
  `conventionalCaughtAt: 8` **in the table** — the §8 amendment, made on the owner's instruction.

The reviewer was reviewing the artifact against the spec table they wrote, without this log. They
identified the mismatch correctly and diagnosed the mechanism wrongly. On being shown the audit they
withdrew it and said the audit-before-complying was the right behaviour. Recording that plainly: the
mitigation that worked here was not a test. It was a written record of a decision and its reason, which
is the thing stage 9 exists to produce and the thing this whole exercise argues for.

### The substantive question, answered on its merits

The owner asked, separately from the bug report, whether a 50% escape rate is defensible for the
conventional run — noting their original zero was probably too optimistic, since pre-AI processes shipped
defects too. Reasoning per defect, without looking at the total:

| Defect | Defensible conventional catch | Why |
|---|---|---|
| D1 contract mismatch | 5 Build | A developer writing against the real API hits it immediately. |
| D3 timezone at boundary | 7 Release | Billing boundaries are standard QA territory for a subscription product. |
| D4 promo lock-in missing | 6 QA | Invisible while building the happy path; a promo test account finds it. |
| D5 collections missing | **8 escaped** | No conventional QA script covers accounts in collections unless someone already suspected the state. |
| D6 effective date ambiguous | 7 Release | Dev picks a reading and QA tests that reading, so it surfaces at sign-off at the earliest. |
| D7 focus management | **8 escaped** | No automated accessibility rules, and manual QA rarely drives keyboard-only paths. |
| D8 legal tone | 7 Release | Legal sees final copy at release sign-off. |

**Two escapes out of seven, not four and not zero.** Four (57%) describes a team that ships most of what
it finds. Zero describes one that catches everything, which the owner has themselves disowned.

### What that does to the headline, which is the real finding

| Conventional lane | Total | Escapes | Headline (triad, probes deleted) | Thin vs conventional | A3 |
|---|---|---|---|---|---|
| Spec as written | 64.9 | 0 | −18.8% | +81.0% | fail |
| **Reasoned per defect** | **96.0** | **2** | **−45.1%** | +22.4% | fail |
| Reasoned, D6 also ships | 102.0 | 3 | −48.3% | +15.2% | fail |
| The §8 amendment | 108.0 | 4 | −51.2% | +8.8% | pass |

Two things fall out, and both are more useful than the demo working.

**A3 is unachievable by any defensible control arm.** It requires the conventional total to land between
106.8 and 130.6. Each escape is worth 10.0–12.5 days against a 51-day baseline, so satisfying A3 takes
four or five escapes — a control arm that ships most of its known defects. A3 does not merely happen to
fail; it *demands* a conventional lane nobody would defend. It should be dropped, or demoted from an
invariant to an observation. The §1 suspicion that A3 asserts something the table cannot produce is now
arithmetic rather than suspicion.

**The headline is hypersensitive to a number nobody has measured.** −18.8% at zero escapes, −45.1% at
two. The single largest lever on the demo's top-line claim is how many defects you let the control arm
ship, multiplied by an assumed ×25. Any presentation of the headline that does not say this out loud is
overclaiming, and the honest framing of the artifact is that it demonstrates that sensitivity rather than
a result.

### What shipped, and why

**The spec table, unamended.** Conventional 64.9, zero escapes, A3 failing at +81%, `D9` back to
`{triad:8, human:8, thin:8}`. `SPEC-machine.md` and `SPEC-tests.md` are restored to their original text.
Reasons, in order:

1. The §8 amendment was made to make A3 pass. That is the move the `CLAUDE.md` stop condition prohibits
   by name, whoever authorised it. Reverting is what should have happened at §1.
2. A3 failing is the finding, and it is a better one than A3 passing ever was.
3. It agrees with `reference/ai-worked-example.html`, which puts the gain at "about a fifth". The
   two-escape version does not, and the cross-document conflict the reviewer flagged is real: at −45%
   the demo and the worked example describe different projects.

**The two-escape table is the recommendation, not the shipped state.** It is more honest about the
conventional process than either the spec's zero or the amendment's four, and adopting it means moving
the worked example's headline to match. That is an owner decision with a document change attached, and it
is not mine to take unprompted.

### Also fixed, from the same review

- **"Defects open" was a mislabel.** They are surfaced and resolved. Now "Defects surfaced", with the
  sub-label "found and resolved by stage N". Escapes remain a separate tile.
- **Comma decimals in the assumption inputs.** A real hazard, not just an inconsistency: the handler used
  `parseFloat`, and `parseFloat("0,4")` is `0`, so a comma-locale entry of "0,4" would have silently set
  the assumption to zero rather than rejecting it — C4 violated in a way no test in this file caught,
  because every test typed periods. Now uses `valueAsNumber`, which is locale-correct, and the inputs
  carry `lang="en"`. Verified under en-GB, fi-FI and de-DE: field shows `0.4`, typed values apply, no
  `NaN`. Not verified on iOS Safari, where the reviewer saw it — display formatting there is the
  browser's and may still differ.
- **Mixed-numbering rework arrows.** The `conventionalIntroducedAt` gap is a genuine omission in
  `SPEC-machine.md`, acknowledged by its author. Rather than keep the §10 kind-mapping — a guess, however
  defensible — the conventional lane no longer draws rework destinations at all. It still counts its
  loops; the legend says why nothing is drawn. Proposed values for the field, for approval, all of which
  happen to equal what the kind mapping produced, so adopting them changes no totals and only makes the
  claim explicit: D1→3, D3→5, D4→2, D5→2, D6→2, D7→5, D8→3. The rework *predicate* should keep using the
  shared `introducedAt`, or the thresholds need a conventional equivalent too.

### Not addressed

**5.1 asked for three regions on one screen; this is a scrolling document.** The reviewer is right that
this weakens the assumptions panel, which is a scroll away from every figure it drives. Fixing it is a
layout rewrite, and the file is at **598 lines against a 600 stop condition** — there is no room to
attempt it. Logged as an open deviation from 5.1 rather than quietly accepted.

---

## 15. Removing the claim rather than defending it

The owner's call, and the best structural decision made about this artifact: **ship with no headline
percentage.** The rationale is exact — the number was one unmeasured integer times an assumed ×25, and
§14 measured its range at −18.8% to −51.2%. A figure that volatile is not a result. Both lane totals now
appear side by side in the tally and the stage 9 record states them without a delta, and the record says
in as many words that the difference is withheld because it moves more than twenty points on the escape
count alone.

This also dissolves the cross-document conflict without either document conceding: neither has to assert
a figure the other must match. And it is consistent with the rest of the set, which refuses multipliers
everywhere else — the artifact was the one place a headline number had crept back in.

**The two levers are now adjacent to the tally**: production multiplier and defect base cost, editable
in place, synced with their copies in the assumptions panel. Together they price an escape. Moving the
multiplier from 25 to 10 takes the thin path from 117.5 to 87.5 in front of the viewer, which is a better
demonstration of the model's sensitivity than any sentence about it.

### A3 deleted

Removed from `SPEC-tests.md` by its author, with a note in that document explaining why. Their words: *"I
wrote a test asserting a conclusion. That's the 8X slide in test form, and it's the root cause of the
whole §8 episode."* Deleted rather than demoted, because demoting it would have left the conclusion in
the document with a softer label.

Worth stating what this costs: `SPEC-tests.md` no longer contains any criterion about the relationship
between the two lanes. That is correct — there is no invariant there to assert — but it means nothing in
the test document now checks the artifact's central comparison. The comparison is an output to be read,
not a claim to be verified, and the criteria set should be understood that way.

### The line cap was binding on the wrong thing

Raised 600 → 700 in `CLAUDE.md` and `SPEC-machine.md`, with the reason recorded in both: the cap existed
to stop scope creep in the model, and was instead blocking a presentation fix the artifact needed. Now
612 lines. The model has not grown; the model is smaller than it was two days ago.

### Reconciliation asked for, and what the documents actually say

The owner asked me to reconcile the reasoned conventional table against "the conventional document"
before adopting it, citing three specifics. Checked against `reference/`, which contains only
`ai-process-spine.html` and `ai-worked-example.html` — **there is no conventional-run document in the
input set.** Both describe the AI-assisted run.

| Claim | What the file says |
|---|---|
| A production defect at a billing-period boundary, untested timezone | *"Two failed on timezone handling at billing-period boundaries"* — stage 7 **Test**, fixed. No production defect is named anywhere. |
| Eleven unconsidered states surfacing during build | No occurrence of "eleven". *"Four unconsidered scenarios surfaced at **concept stage**"*; 40 generated, 34 surviving. |
| D7 found in QA and accepted as a known limitation | *"Suite in CI, two defects fixed, one accepted and documented."* The category is real. Which defect is never stated. |

Two of the three are not in the documents, and all three describe the assisted lane. Setting
`conventionalCaughtAt` from them would take evidence about one process and apply it to the other — the
same category error as §10's arrows, one level up. Declined, and reported instead.

The assisted column already agrees with the worked example without changes: D3 `triad:7` **is** the
timezone catch in Test, and D5 `triad:3` **is** a state surfacing at concept. On D5 the document argues
against the proposed revision outright — *"A coded prototype cannot tell you about accounts in
collections or promo lock-in — you have to already suspect those states to build them"* — which is the
reason a conventional run ships it.

**Adopted as reasoned in §14**, unreconciled: conventional 96.0, two escapes (D5 collections, D7 focus
management), four rework loops. Two cells remain disputed and are the owner's to settle — D3, where their
instinct that a conventional run ships a timezone defect is defensible even though the cited evidence was
not, and D5, where the document contradicts the proposed change.

### `conventionalIntroducedAt` added

Approved as proposed: D1→3, D3→5, D4→2, D5→2, D6→2, D7→5, D8→3. In `SPEC-machine.md` with a paragraph
explaining why the field exists. Used for the arrow destination only; the rework predicate keeps the
shared `introducedAt`, or its thresholds would need a conventional equivalent too. Conventional rework
arrows are drawn again, now from data rather than from §10's inferred mapping. Totals unchanged, as
predicted — the values equal what the kind mapping produced, so the only difference is that the claim is
now explicit and signed off.

### The modelling gap: there is no "found and accepted"

Logged, not built, per instruction. Every defect in the model is either caught-and-fixed or escaped.
There is no third outcome for *found, priced, and knowingly shipped* — which is the most common real
disposition for accessibility defects and minor issues, and which the worked example itself records:
"two defects fixed, one accepted and documented".

Why it matters more than the escape count it was raised alongside: a model with only two outcomes must
score every discovery as a cost avoided. A process that finds a defect and accepts it pays the finding
cost and none of the fixing cost, and gets none of the escape penalty either. Absent that category, **any
simulation flatters whichever process finds things earlier**, because finding is modelled as unambiguously
good. That bias runs in the assisted lane's favour throughout this artifact, and it is not visible in any
figure on screen.

Not built: it needs a third disposition on every defect, a cost for accepting, and a decision about
whether accepted defects count toward "surfaced". That is a model change, at the point where the model
was supposed to stop growing.

### Reclassified: the most valuable entry in this log

At the owner's direction, and I agree with the reasoning. **§14's `parseFloat("0,4") === 0` finding is
promoted above §3.2.**

`parseFloat` on a comma-decimal locale entry returns `0` rather than failing. An assumption typed as
"0,4" would have been silently set to zero — not rejected, not flagged, no `NaN` on screen, C4 reporting
clean. Every automated check in this build typed periods, so every one of them passed. The bug was
invisible to the entire test suite because of an unexamined property of the person who wrote the tests.

That is a better instance of the failure class than §3.2. The tally bug was a layout mistake no criterion
covered. This one **passed a criterion that was specifically written to catch it** — C4 exists to ensure
invalid input is rejected and no bad value reaches the model, and it reported success while a silent
zeroing path sat open. Internally consistent, self-verified, wrong, and wrong in a way that quietly
alters the numbers the whole artifact exists to show.

It was also found in the tooling — by running the thing under three locales — rather than argued about in
a document. Fixed with `valueAsNumber` and `lang="en"`, verified under en-GB, fi-FI and de-DE. Still
unverified on iOS Safari, where the reviewer originally saw comma formatting.

---

## 16. The artifact was written for the people who wrote the specs

Reported by the owner: *"Data is fine for now, but problem is that from story and presentation
perspective this is too technical. The story needs to be easily understandable and relatable by any
stakeholder: business, designer, developer."*

This is the fourth finding of the shape §13 named, and the pattern it stated has now held four times for
four: **every defect a criterion caught was mine to make, and every defect about whether the thing
communicates came from a person looking at it.** Nothing in A–E can fail on an unreadable page. All 32
criteria passed on the version being complained about.

### Where the technicality came from, which is not where I expected

I assumed I had written insider copy. I had not — I had faithfully reused it. `CLAUDE.md` names
`reference/ai-process-spine.html` as the source for stage wording and says to reuse it rather than write
new descriptions. That reference copy reads *"Prompt-to-prototype, flows and states, edge-case sweeps,
heuristic passes"* and *"Rule-checkable first: types, lint, unit, accessibility rules"*. It is written for
someone who has read the whole document set. Reusing it verbatim was the instruction, and following the
instruction produced the defect.

So the fix required going against a standing instruction, which `CLAUDE.md` makes a stop-and-ask. Put to
the owner as one of four decisions, all four answered before any code changed:

| Decision | Answer |
|---|---|
| Stage copy: rewrite plainly, or keep reference wording and add a plain line beside it | Rewrite. Keep every claim, drop the shorthand. |
| Choice button labels, which `SPEC-machine.md` fixes verbatim | Rename, and edit the spec to match. |
| Line cap, then at 612 of 700 | Raised to 800. |
| Presenter narrating, or a link read alone | Read alone. The page carries its own argument. |

`CLAUDE.md` amended: the reference governs the visual treatment, the stage names and the substance of
each stage — not the wording. `SPEC-human.md` gained a section on being read without a presenter.
`SPEC-machine.md` gained a paragraph saying the button labels are presentation and the
`thin`/`human`/`triad` keys are the contract, so the next person to reword them does not think they are
touching the model.

### The model was not touched, and that is checked rather than asserted

Totals for all six paths were captured before the first edit by evaluating the model section out of the
file in node, and diffed after every round of changes. Byte-identical throughout: assisted 52.7 / 76.9 /
117.5 with probes deleted, 65.2 / 89.4 / 130.0 with probes kept, conventional 96.0. No change to
`DEFECTS`, the stage tables, `DEFAULT_ASSUMPTIONS`, or any function in the model section. No new state
field, no new control.

### The one that passed its own check while being wrong

The `D2` failure mode again, in my own copy, and the most useful entry here.

Each defect gained a plain-language consequence sentence. I wrote them in the past tense, because I was
writing them while thinking about the stage 8 reveal, where the defects have shipped:

> D3 — *"A customer cancelling late on the last evening of their billing period **was charged** for
> another month."*

That sentence is used at whatever stage the defect surfaces. On the triad path D3 is caught at stage 7,
fixed, and no customer is ever charged anything — and the panel read as though one had been. The copy was
internally consistent, grammatical, checked against the defect table, and it **overstated the cost of a
process that had just worked correctly**. It made the artifact argue for its own conclusion by accident,
which is precisely what the forbidden list exists to prevent.

No criterion covered it. F2 as I first drafted it required a plain sentence per defect and would have
reported a pass. I found it by screenshotting the stage 7 panel on the triad path and reading it — not by
reading source, and not by any check I had written. Every sentence is now present tense (*"is charged"*),
and the block heading carries the disposition: *"Caught at this stage · what it would have done"* against
*"Nobody caught these before release · what customers met"*. F2 was amended afterwards to require the
present tense and to say why.

The general lesson, which is the same one as §15's comma: the checks I write encode what I was already
thinking about. This one encoded the reveal and missed the other eight stages.

### Invented — not in any spec

- **Nine consequence sentences**, one per defect (`PLAIN`, keyed by id). Invented illustration. Held to
  the existing table: no sentence states a magnitude, no cost comes from anywhere but the model, and
  `DEFECTS` is untouched so the labels remain the spec's. Two are worth flagging as authored claims
  rather than derived ones — D2's *"invented a status that does not exist"* and D9's *"a stand-in the
  build inherited rather than replaced"* — because they name a mechanism the defect table only labels.
- **Plain names for the three spec levels**, used in the stage 9 summary sentence: *notes on the
  designs*, *a write-up for the team*, *documents for the team, the tools and the tests*.
- **A definition of "probe" in the legend.** Found by the F1 grep: the stage name *Concept & probe* is
  model data fixed by `SPEC-machine.md`, so it is on screen from load, while the word was only explained
  in the stage 3 description — which a reader does not reach until stage 3. Defined next to the strips
  instead.
- **Two CSS classes**, `.gl` for the plain gloss under each strip and `.sum` for the stage 9 summary
  paragraph. No new colours: contrast recomputed anyway, 5.26 and 15.60 against paper, both AA.
- **Plain group headings in the assumptions panel** — *"How much more a problem costs, depending on when
  it is found"* for what was *"Catch cost multiplier, by stage caught"*. The panel is not collapsed, not
  moved, and not shortened; every field is still there.
- **A section F in `SPEC-tests.md`**, six criteria, because none of A–E could have failed here.

### Which of F1–F6 passed on the first pass, unmodified

Honest answer: the question is malformed for this entry, because F1–F6 were written in the same pass as
the copy they check. Recorded as it actually went:

| # | Outcome |
|---|---|
| F1 | Failed on first pass. The grep against *rendered* text — not source — found `probe` and `handoff` on screen undefined, and `enum` inside a retained engineering label. Fixed for `probe`; `handoff` is defined by the conventional strip's gloss ("a design signed off and handed over to be built"); `enum` sits under a plain sentence that explains it, and is deliberately kept so a developer recognises the defect. |
| F2 | Passed structurally, failed on tense. See above. |
| F3 | Passed. Then improved: the "how to read a strip" line was below both strips on the first pass, so a reader met the strips before the explanation. Moved above. |
| F4 | Passed. Then two wording bugs found by reading output across all six paths — *"0 problems reached customers"* where *"no problems"* was wanted, and a hardcoded *"Two of the nine happen only on this route"* which is false when the prototypes are deleted and D9 is not in the run. The second is an F5 breach in miniature: a magnitude stated in prose, correct on one path and wrong on another. Now computed. |
| F5 | Passed on the copy written in this pass, having been the rule I was watching for. |
| F6 | One correction: the third spec-level button label read *"Written for the team, the tools and the tests"*, wrapped onto two lines, and rendered visibly larger than the other two. Parallel length is part of neutrality when the buttons sit side by side. Shortened to *"Written for team, tools and tests"*. |

### Also corrected, from the same reading

- `SPEC-tests.md` D5 still said **600 lines** while `CLAUDE.md` and `SPEC-machine.md` said 700. The one
  constraint written into all three documents disagreed in all three, and had done since the 600→700
  raise in §14. All three now say 800.
- The ordering-broken warning still promised that the thin path would land near the conventional lane —
  a claim from A3, which §15 deleted. Rewritten.
- `Out` as the label on every stage's output block. Replaced with *"What this stage produces"*, which is
  what `SPEC-human.md` calls it anyway.
- The `NO_CONVENTIONAL` notes said *"a generated-output failure mode with no human equivalent"*. Same
  point, now in words: *"it is a way generated code fails, and a person writing the same code does not
  fail that way."*

### Verified

660 lines, against the 800 cap. Model totals identical across all six paths. Full criteria suite
re-checked: A1–A10, B1–B6, C1–C5, D1–D6, E1–E5, F1–F6. Driven under Chromium at 1280×800 and 1920×1080,
all six paths to stage 9, no console or page errors, no `NaN`/`undefined`/`Infinity` in rendered text.
"Assumed" appears 22 times on screen. Invalid input re-checked under en-GB, fi-FI and de-DE after the
panel copy changed — empty and negative both flagged, last valid value retained, §15's fix intact.
Contrast recomputed for every text pairing: lowest is 5.26.

### What was not checked

- **iOS Safari and any real device.** Same gap as §11 and §15. Everything above is Chromium in this
  container; the session's egress policy still blocks the Pages domain, so every claim here is about the
  file, not the deployed page.
- **Whether a stakeholder can actually follow it.** This is the only thing the entry is about and it is
  the one thing I cannot verify. I read the page as though presenting it, listed the sentences that would
  need explaining aloud, and fixed them until the list was empty — but I have read the whole document
  set, which is exactly the disqualification. Four rounds of this now say the check is a person who has
  not: give it to someone in business or engineering who has never seen these files, watch where they
  stop, and record it here.
- **Screen reader.** The live region and labels are unchanged, but the new copy has not been heard.
- **Human review time.** Still not reported. This log stays incomplete until it is.

---

## 17. Both runs, told at once

Requested by the owner: *"let's roll both stories at the same time side by side? If we also can see all
the steps as a story in conventional way it's easier to understand the differences."*

Fair, and the gap was structural rather than cosmetic. §16 made one route readable. The other was a grey
strip with a total, so every difference between the two processes had to be inferred from bar widths. The
artifact's whole claim is about *where* problems get found, and it was asking the viewer to take that on
trust.

### The blocker, which is the reason this is not just typing

**There is no conventional-run document in the input set.** §14 established that when the owner asked for
a reconciliation; I re-verified it. `reference/` holds `ai-process-spine.html` and `ai-worked-example.html`
and both describe the assisted run. §14 also set the rule that a previous attempt broke: evidence about
the assisted lane must not be transplanted onto the conventional one.

So a conventional narrative is authored. Put to the owner as four decisions before any code changed; all
four answered. What the documents *do* support, used for substance and never for duration:

| Anchor, quoted | Phase |
|---|---|
| *"clustered … in an afternoon. Previously two weeks of sampling."* | 1 Discovery |
| *"you have to already suspect those states to build them … previously not available at any price"* | 2 Concept |
| *"scaffolded from tokens and 5.2 in two days for what was previously a week"* | 5 Build |
| *"a template field that is empty nearly everywhere"* | 8 Retrospective |

**Their durations are deliberately not used.** "Two weeks of sampling" against a Discovery phase the spec
prices at 7 person-days is a contradiction waiting to be spotted by a reader, and the reference's own
conventional split (15/22/6/35/14/8) is on a different scale again from the spec's phase table. So the
copy carries the anchors' substance — read by hand and sampled rather than counted; the states you already
suspect are the states you design — and states no duration anywhere. That is F5 applied to authored
narrative rather than to computed figures.

Invented outright: **3 Design to spec, 4 Handoff, 6 QA phase, 7 Release.** Every panel in that column
says so on screen, under the column heading, not in a footnote. The assisted column now says where its
copy comes from too — otherwise a reader would reasonably assume parity between a reconstruction and a
worked example.

### The alignment is data, not a runtime guess

`ALIGN` maps each assisted stage to the conventional phase doing the same kind of work. It could have been
derived from the shared `kind` field at render time. It is written out instead, for the reason §14 added
`conventionalIntroducedAt`: when two lists number different things, an implicit mapping is a claim nobody
signed off. `SPEC-machine.md` carries the table.

Phase 4 Handoff has no assisted counterpart at all, and sits with Specify — which turns out to be the
best screen in the artifact. Left: write the decision down for three readers. Right: finish the design,
sign it off, hand it over, and answer questions in chat afterwards. Stated as what each route does, with
no adjective doing work in either direction.

### The near-miss, and it is the most valuable entry in this round

The number `8` means two things. In the defect table, `caughtAt: 8` means escaped to production. In the
phase lists, row 8 is `Release & measure` on the assisted route — a real stage where escapes belong — and
`Retrospective` on the conventional one, which is not where a customer meets anything. The source already
carried a comment about this from §10's era, which is the only reason I looked.

I handled it in the story column: conventional escapes display on phase 7 Release, so both routes reveal
what customers met at the same step. Then I built the paired tally from the aligned partial run, which
counts phases up to 7 — and the escapes are stored on 8. **At stage 8 the tally would have read "0 reached
customers" directly beside a panel naming two customers' problems.**

Not caught by a criterion. G5 did not exist yet, and G2 as first drafted only checked the stage 9 total —
which was correct, because by stage 9 phase 8 is included and everything reconciles. The bug lived
entirely in the middle of the walkthrough and disappeared at both ends. I found it by reading the two
regions against each other while writing the plan's verification section, before either was built.

Fixed with `convUpTo()`, which adds the escaped defects' cost from Release while leaving the
retrospective's own 0.2 days where it belongs. Both regions now answer the same question the same way.
G5 was written afterwards to make the rule checkable, and G2 was strengthened from "the total matches" to
"monotonic across the walkthrough and exact at the end".

The pattern worth naming: **the last two rounds' worst defects were both consistency failures between two
places on one screen**, not arithmetic failures. §16's tense bug made a caught defect read as shipped;
this one would have made a shipped defect read as caught. The model was right both times.

### What the paired tally exposed, which nobody asked for

At stages 0, 1 and 2 the conventional column reads 7.0 person-days while the assisted run climbs 2.0 →
4.0 → 9.0. **The AI-assisted run is behind at stage 2** — its three discovery stages cost more than the
single Discovery phase they replace. That was always true in the model and was invisible while the only
comparison on screen was whole-run against whole-run. Showing both at the same point made the artifact
less flattering to the process it demonstrates, which is the correct direction for it to move, and it is
left exactly as the model produces it.

### Invented — not in any spec

- **Eight phases of conventional narrative**, four fields each (`CONV_COPY`), provenance above.
- Two authored details worth flagging because they read like data and are not: the Handoff walkthrough is
  "two hours" with "three questions in the room", and Discovery's themes are "the ones a reader noticed".
  Neither is computed; neither contradicts a figure.
- **`ALIGN`, `alignedPhaseMax()`, `stagesInPhase()`, `convUpTo()`, `convPhase()`** and the refactor of
  the stage panel into `stagePanel()` / `foundBlocks()` so both routes render through one path — which is
  also why the conventional column cannot drift from the assisted one in styling or in disposition
  wording.
- `foundBlocks()` now splits caught from escaped into two blocks. The old single heading was fine while
  only stage 8 held escapes; the conventional Release phase holds both at once, and one heading cannot
  describe both honestly.
- **Section G in `SPEC-tests.md`**, six criteria. **B5 amended** — it said the conventional lane "never
  changes", which conflated the strip with the figures and became untrue of the figures once the owner
  chose to show both runs at the equivalent point. The substance is unchanged and G3 is the check.

### Which of G1–G6 passed first pass

| # | Outcome |
|---|---|
| G1 | Passed. Verified by script rather than by reading the table: all ten stages covered, all eight phases covered, Handoff on stage 5, mapping monotonic. |
| G2 | Failed as first written, in the sense that it was too weak to catch the escape bug. Strengthened, then passed: monotonic across all ten stages, exact at 96.0 / 59.5 / 26.4 / 5 found / 2 escaped / 4 loops. |
| G3 | Passed. Automated — the conventional column's rendered text captured at all ten stages across all six choice paths and diffed. Identical. |
| G4 | Passed after one correction: the provenance line existed only on the conventional column, so the two columns' panels started at different heights and a reader would have assumed the unlabelled one was sourced. Both columns now carry one. |
| G5 | Did not exist until the near-miss above created it. |
| G6 | Passed. Grep for counts and durations in copy returns only the brief's own substance (a 12-month promotion), sourced narrative detail, and the record summary's computed pluralisation. |

### Also corrected

- The stage 5 choice prompt did not say whose decision it was. Harmless in a single-column layout,
  ambiguous the moment a second route appeared beside it. Both prompts now open "Your decision on the
  AI-assisted run".
- The shared-phase note ("stages 0, 1 and 2 all happen inside this one phase") sat above the panel and
  pushed the right column down. Moved inside, under the ownership line.

### Verified

731 lines against the 800 cap. Model totals byte-identical across all six paths, diffed against the
baseline captured before the first edit of §16. Full suite: A1–A10, B1–B6, C1–C5, D1–D6, E1–E5, F1–F6,
G1–G6. Two columns at 1280×800 and 1920×1080, 583.5px each, no horizontal overflow at either. Ten stages
screenshotted and read. No console or page errors on any of the six paths. Invalid input still rejected
under en-GB, fi-FI and de-DE. "Assumed" now appears 24 times on screen. No new colours, so E4 stands.

**A script error, and the first time the pattern did not repeat.** My G2 script clicked Next before making
the stage 4 choice and timed out on a disabled button — the same mistake as §13, B1 blocking exactly as
specified. §13 records that I suspected the artifact first; this time I read the disabled attribute in the
error output and fixed the script without opening the source. Recorded because three earlier entries
recorded the opposite.

### What was not checked

- **Whether two columns is one column too many for a stakeholder.** The layout stacks below 1240px, which
  I have looked at, but the request came from wanting the comparison to be easier — and it is possible
  that twice the reading is harder, not easier, however well aligned. Same disqualification as §16: I have
  read the document set and cannot judge this.
- Any real device, iOS Safari, or the deployed page. Egress to the Pages domain is still blocked here.
- Screen reader on the paired layout. The two columns are separate `div`s with heading text, in source
  order left then right, which should read as two sequential sections — unverified by ear.
- Whether the four invented phases describe anything a team would recognise. They are mine, they are
  marked as mine, and the person to correct them is someone who has delivered this way.
- Human review time. Still not reported.

---

## 18. A run sheet, and the crossover said out loud

First feedback from someone outside the project — the check §16 and §17 both asked for and could not
perform. It arrived in Finnish and is worth quoting rather than paraphrasing:

> *"Tää on tosi hyvä! Selkeä toi rinnakkain juoksutus ja hyvät huomiot eri vaiheissa. Saako tästä lopulta
> irti jonkinlaisen check listin, jolla pystyisi varmistamaan noita vaiheita ja valintoja? Erityisesti, kun
> tekee ensimmäistä kertaa, se voisi olla hyödyllistä."*
>
> Asked what kind: *"Jonkunlainen työkalu prosessin juoksuttamiseen?"* — *"Joo, jotain sellaista, että ota
> tämä huomioon, niin säästät aikaa/vaivaa seuraavassa vaiheessa tms."*
>
> *"Tää tosi hyvin tekee siitä näkyvää, että valinnoilla on vaikutusta seuraavaan vaiheeseen ja
> työmäärään. Ja 'valitsemalla väärin' saattaa AI versiosta tulla kalliimpi/hitaampi kuin ns. vanhalla
> tavalla työskentelemällä."*

Two things in that, and both are firsts.

**The side-by-side layout passed the test I said I could not run.** §17 recorded that I could not judge
whether two columns made the comparison easier or merely doubled the reading. A reader outside the document
set says clearly: *selkeä* — clear, with the per-stage observations landing. That closes the open question
from §17 in the affirmative, and it is the only entry in this log where an open question was closed by
someone other than me.

**And the artifact's hardest finding was reached unaided.** *"Valitsemalla väärin saattaa AI versiosta
tulla kalliimpi/hitaampi"* is exactly what §14 and §15 fought over and what the paired tally exposed in
§17 — a reader got there from the totals with nobody pointing at it. That is the strongest evidence in this
log that the thing communicates, stronger than any criterion could be, because it was not prompted.

### What the ask actually was, and the rule it broke

Not a summary. A *tool for running the process* — imperative, usable on a real project, of the form "do
this now and the next stage costs less". The artifact could not become that without breaking its own rules:
`SPEC-machine.md`'s forbidden list bars copy that recommends anything, and B4 requires that a reader cannot
infer which option is preferred. A hedged checklist is not a checklist.

Put to the owner as four decisions, all four answered before any code changed. The one that mattered: the
run sheet becomes a **separate file with its own rule.** `checklist.html` may instruct; `demo.html` may
not. The boundary runs between the two files rather than inside one, and it is written into `CLAUDE.md`,
`SPEC-human.md` and `SPEC-machine.md` so the next change does not quietly blur it — with H7 added to make
it checkable: *the demo gives no instructions and the run sheet carries no figures.*

### `checklist.html`

Thirty items across ten stages, two decision boxes, no script at all. Every item either cites the problem
it prevents by the id `demo.html` uses, or is marked **authored** — practice the model does not encode, and
therefore the first thing a reader should argue with. The split came out at 14 defect citations across 30
items; the other 16 are authored, which is a fair account of how much of "how to work well" this model
actually contains.

Three decisions in it worth recording:

- **No figures at all.** Not even the ones the model computes. A sheet gets printed and read months later
  by someone who was not there when an assumption was edited, so any number on it is a latent lie. This is
  §11's rule taken to its conclusion — and it made the sheet better, because "expect the catches to move
  later" is more useful to a first-timer than a multiplier.
- **Stage 0 says it prevents nothing.** Nothing in the defect list is prevented by writing a hypothesis
  down. The honest note is that the stage buys the ability to stop, which the model does not price. Padding
  it would have been easy and would have been the first lie on the page.
- **The decision boxes are conditionals, not verdicts.** "If you keep a prototype, then list every stand-in
  in it and replace them before release." A real team may legitimately keep one; the sheet has to be usable
  by them. The only flat assertion is that keeping one *without* a list of what in it was fake is not
  defensible.

It also carries a gap the model has and the sheet should not: stage 7 tells the reader to record a problem
they accepted rather than fixed, and says outright that the demo's model has no category for that — §14's
"found and accepted", still logged and unbuilt. The sheet is where that omission stops being invisible.

### The crossover, in the demo, computed

`crossover()` compares all six paths against the conventional total on every render and names the ones
above it. Under the defaults, two of six — notes-only with the prototypes deleted, and notes-only with one
kept. Phrased as what it is: *whether AI is faster here is not a property of AI; it is a property of these
numbers, and you can change all of them.*

Hardcoding "two of six" would have been the same latent bug as §11's magnitudes and §16's "two of the
nine", so it is computed. Verified under three regimes: defaults (2 of 6), every specification cost raised
until all six cross (reads "every one of the six"), and escapes made cheap until none do.

### Found in my own output, twice, by reading it

Neither was caught by a criterion. Both were caught by reading rendered text against the model, which is
now the third round in a row where that is the method that works.

1. **The stage 9 block credited the wrong spec level.** For each problem that reached customers it named
   the level that catches it earlier — picked with a `reduce` that returned the *strictly earliest* match in
   key order, which is the fullest level. `D2`, `D3` and `D7` are caught at the same stage by the team
   write-up as by all three documents, and the block was crediting all three. It overstated what the reader
   would have to do, in the direction that flatters the process the artifact demonstrates. Fixed to report
   the **thinnest level that achieves the earliest catch**, which is both accurate and less flattering.
2. **The level names were about to exist twice.** The record summary had them inline; the new block needed
   the same three strings. Two copies of the same prose in one file is the drift risk that
   `conventionalIntroducedAt` (§14), `ALIGN` (§17) and H3 all exist to prevent. Hoisted to `LVL_PLAIN`
   before the second copy was written rather than after.

### Invented — not in any spec

- The thirty checklist items, 16 of them authored practice with nothing behind them but judgement. The
  three most load-bearing: book the review time as calendar time before the build starts; keep the money
  arithmetic human; ask which states nobody in the room has met, then look in the data rather than in the
  team's memory.
- `crossover()`, `checklistBlock()`, `LVL_PLAIN`, the `.ckb` block, `a{color:var(--cheap)}` — the first
  link the demo has ever contained.
- Section H in `SPEC-tests.md`, seven criteria, of which H7 is the only one that polices a boundary
  between two files rather than a property of one.

### A criterion that was wrong on first writing

H7 — *the demo gives no instructions* — failed on its own first run, and the fault was the criterion rather
than the artifact. Grepping for imperative verbs flagged all ten stage descriptions, because they are
written in the mood the process documents use: *agree what problem is being solved*, *draw the edges of the
job*, *write the decision down*. Those describe what a stage consists of. They do not tell the viewer what
to do about their own choices, which is the boundary that actually matters.

Sharpened to grep for second-person prescription instead — "you should", "make sure", "we recommend", "you
need to" — and re-run: none in the demo. Two further passes were needed even then, because "always" and
"never" turned out to be noise rather than signal: *"a flow that was never designed for them"*, *"never in
response to either of your choices"*. All descriptive. Recorded because it is the third criterion in this
build (after A3, then G2) that had to be rewritten once it met the thing it was supposed to check, and
because a criterion that fires on correct work is worse than no criterion — it trains you to ignore it.

### Verified

`demo.html` 783 lines against the 800 cap — **17 lines of headroom, and the next content addition of any
size needs the budget revisited rather than absorbed.** `checklist.html` 311 lines. Model totals
byte-identical across all six paths, diffed against the baseline captured before §16. Full suite: A, B, C,
D, E, F, G, H. H1: 30 of 30 items carry provenance. H2: grep for days, hours, multipliers and percentages
in the sheet returns nothing. H3 by script: stage numbers and names match `ASSISTED_STAGES` exactly, all 14
cited ids and label tags match `DEFECTS`, and every one of the nine problems is cited somewhere. H5: no
script, no network, no storage. H6 under three assumption regimes. No console or page errors.

**H4 is a partial pass and the reason is worth stating.** No PDF renderer exists in this container, so I
could not read the printed pages. Chromium's own text extraction returned only ligature glyph names from
the subset fonts. Instead: print media emulated at A4 content width, every block measured, tallest is 409px
against a 1016px printable page, and `break-inside:avoid` confirmed as computed rather than merely written.
That is a geometric argument that nothing *can* split, not an observation that nothing *did*.

### What was not checked

- **The printed sheet, on paper.** See above. The geometry says it holds; nobody has held it.
- **Whether the authored items are any good.** Sixteen items of practice advice with the model behind none
  of them. They are marked, which is honesty rather than validation. The person to correct them has
  delivered a project this way and has not read these documents.
- Any real device, iOS Safari, or either deployed page.
- Screen reader on the run sheet — the checkboxes are CSS pseudo-elements on list items, which read as a
  plain list, and that is probably right, but it is unverified by ear.
- Human review time. Still not reported, eighteen sections in.
