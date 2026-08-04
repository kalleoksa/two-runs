# 5.1 — Human-readable spec

**Artifact:** `demo.html` — a single-file interactive walkthrough of two delivery processes running on the same brief.
**Audience:** the team, and whoever presents this.
**Reader of this document:** humans. See `SPEC-machine.md` for the model contract and `SPEC-tests.md` for acceptance criteria.

---

## Purpose

Let a viewer cause the outcome rather than be told it. Two decisions are handed to them — whether to discard probes, and how completely to specify — and the consequences accumulate visibly across the remaining stages. The stop rule ("if 5.2 and 5.3 cannot be written, stage 6 does not start") should end up feeling like something they discovered, not a slogan they were shown.

Secondary purpose, equally important: the demo must be able to lose. If a viewer picks thin specs and the totals come out roughly level with the conventional run, the model is working. Nothing here should be tuned to make AI look good.

## The screen

One page, no scrolling between steps. Three regions.

**Top — two lanes.** The conventional run sits above as a fixed reference, its phases pre-filled and greyed. The assisted run sits below and fills in as the viewer advances. Both lanes are horizontal strips of stage blocks, growing left to right, using the same segment colours as the existing effort bars so the pair reads as continuous with the documents.

**Middle — the current step, both runs at once.** Two equal columns: the assisted stage on the left, the conventional phase doing the same kind of work on the right. Each carries its name, what happens, who owns it, what it produces, what happened on this brief, and the problems that route finds at that point. Reading them across is what makes the difference between the two processes legible rather than something to infer from bar widths — and where the two lists do not line up one-to-one, that mismatch is the interesting part and is stated: three assisted stages sit inside Discovery, and Handoff has no assisted counterpart at all.

Each column says where its narrative comes from, directly under the column heading rather than in a footnote. The conventional column is a reconstruction — no record of that run exists — and a reader must not have to guess which column is evidence and which is written. At the two decision points the choice appears below both columns, attributed to the assisted run, because the other route has no such decision to make.

**Bottom — the running tally.** Effort, calendar days, review hours, defects open, defects escaped to production, rework loops — for both runs at the equivalent point, so the score answers the same question the two columns above it do. Numbers change as stages complete; changes should be legible rather than animated for effect. The conventional figures move with position in the walkthrough and with nothing else, never with either choice, and the strip above always shows that run whole; both statements belong on screen, because "fixed reference" on its own is no longer precise enough to be true. The tiles are labelled for a reader rather than for us — team time, elapsed time, senior review, problems found, reached customers, stages reopened — with the modelling term kept underneath each, so *escaped* and *rework loop* still appear and still mean what the legend says.

## The walkthrough

Ten stages, advanced one at a time by the viewer. Back and reset are always available; nothing is irreversible.

Stages 0 through 3 run without interaction. Stage 3 introduces latent defects the viewer cannot see yet — this is deliberate, and the reveal later is the point of the whole thing.

**Decision one, at stage 4.** *Keep the prototype, or delete it?* Keeping it looks reasonable and free. It carries fidelity debt that surfaces in production. The prompt must not signal which choice is correct.

**Decision two, at stage 5.** *How completely do you specify?* Three options: thin annotations, human-readable only, or the full triad. Each is described neutrally in terms of what gets written, not in terms of outcome.

Stages 6 through 9 play out the consequences. Defects surface at the stage where each becomes detectable given the spec level chosen. When one surfaces that was introduced before stage 5, a rework arrow fires back to its origin stage in the lane and cost is added. Escaped defects appear at stage 8 as production findings.

Stage 9 prints the run record: every defect with where it was introduced, where it was caught, what it cost, and the final comparison against the conventional lane.

## Assumptions must be visible

An **Assumptions** panel, open by default on first load, listing every number the model uses: stage base efforts, spec-level multipliers, defect cost multipliers by catch stage, review-hour rates, rework cost. All editable, with a reset.

Every number is labelled **assumed**. There is no data behind any of them. A viewer who thinks the outputs are measurements has been misled by us, which is the exact failure of the slide this whole set replaces.

## Tone

Neutral throughout. No celebration when the triad path performs well, no scolding when the thin path does badly. The record at stage 9 lists what came back wrong regardless of path chosen, because both paths produce defects — the difference is only where they are caught.

## Written to be read without a presenter

The artifact is met as a link, by one person at a time, with nobody standing next to it to explain what a
word means. It therefore has to carry its own argument, and that has consequences the earlier version of
this document did not draw:

- **Every load-bearing term is defined on screen.** Not only *escaped* and *rework loop*, which was the
  §11 fix, but anything a reader outside the team would stumble on: what each of the two processes
  actually is, what a strip means, what the spec triad is without reference to `5.1` / `5.2` / `5.3`.
- **Stage descriptions are written for a first-time reader, not lifted from `reference/`.** The reference
  files remain the source for the visual treatment, the stage names, the ownership lines and what each
  stage produces. Their *wording* was written for people who had read the whole document set, and reusing
  it verbatim is what made the demo unreadable to a stakeholder. `CLAUDE.md` is amended to match.
- **Every defect carries a plain-language consequence** alongside its id and engineering label: one
  sentence naming who notices and what happens to them. Written in the present tense, because the same
  sentence has to read correctly whether the problem was caught at its stage or shipped.
- **The run ends in sentences.** Stage 9 states in plain words what was chosen, what came back, and how
  the totals compare — before the table, and stated flat in whichever direction the numbers point.

The pull this creates is toward drama, which the tone rule above forbids. Plain is not the same as
persuasive: the escaped-defect sentences must not be written in a more alarming register than the
caught-defect ones, and no sentence may state a magnitude, because every magnitude on this page is one
edit away from being false.

## Out of scope

No model calls, no backend, no persistence, no build step. No multi-scenario library. No editing the stage or defect model in the interface — that is done in the source. Desktop only; this is a presentation artifact.

## Optional, only if the core works (P2)

A third decision at stage 6: delegate the billing logic or write it by hand. Delegating is faster and introduces one high-cost defect that only surfaces in production. It demonstrates the verification-cost rule directly. Build it last or not at all.
