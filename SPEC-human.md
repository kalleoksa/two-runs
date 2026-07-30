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

**Middle — the current stage.** Name, what happens, who owns the decision, what it produces. At the two decision points this region becomes the choice instead.

**Bottom — the running tally.** Effort, calendar days, review hours, defects open, defects escaped to production, rework loops. Numbers change as stages complete; changes should be legible rather than animated for effect.

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

## Out of scope

No model calls, no backend, no persistence, no build step. No multi-scenario library. No editing the stage or defect model in the interface — that is done in the source. Desktop only; this is a presentation artifact.

## Optional, only if the core works (P2)

A third decision at stage 6: delegate the billing logic or write it by hand. Delegating is faster and introduces one high-cost defect that only surfaces in production. It demonstrates the verification-cost rule directly. Build it last or not at all.
