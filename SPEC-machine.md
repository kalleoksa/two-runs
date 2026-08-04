# 5.2 — Machine-readable spec

## Intent

Build `demo.html`: a single-file, deterministic, client-side simulation comparing two delivery processes on one brief. Viewer advances stages and makes two choices; the model computes cost, defect detection, and rework from editable assumptions. No randomness, no network, no persistence.

## Constraints

- One file. Inline `<style>` and `<script>`. No build step, no bundler, no framework, no CDN imports.
- Vanilla JS. One state object, one `render()`, pure functions for the model.
- No `localStorage`, `sessionStorage`, or any browser storage API.
- No `<form>` elements. Event handlers only.
- Target ~400 lines. If it exceeds 800, stop and ask. The cap is against scope creep in the model, not
  against presentation the artifact needs; raised from 600 to 700, and from 700 to 800, for that reason
  both times. The model has not grown in either raise.
- Deterministic: identical inputs produce identical outputs on every run.

## Design tokens — use exactly these

```
--paper:#EDEEF0  --panel:#F7F8F9  --ink:#14161A  --ink-soft:#5C6270  --rule:#C8CCD2
--cheap:#2F4BA0  --dear:#9C3B2E
--mono: ui-monospace,"SFMono-Regular","IBM Plex Mono",Menlo,monospace
--sans: ui-sans-serif,system-ui,"Helvetica Neue",Arial,sans-serif
```

Segment colours, by stage kind: `discover #7C8798`, `concept #5C6270`, `spec var(--dear)`, `handoff #B08A3E`, `build var(--cheap)`, `test #4A6FB5`, `release #9AA3B0`, `record #14161A`.

Labels, eyebrows, and all numerals use `--mono` uppercase with `letter-spacing:.1em`. Body copy uses `--sans`. Borders are 1px `--rule`. No border radius, no shadows, no gradients.

## State

```js
state = {
  track: 'assisted',          // fixed; conventional lane is precomputed
  stageIndex: 0,              // 0..9
  probesKept: null,           // null until stage 4 decided
  specLevel: null,            // null | 'thin' | 'human' | 'triad'
  assumptions: { ...DEFAULT_ASSUMPTIONS },  // user-editable
  assumptionsOpen: true
}
```

`probesKept` and `specLevel` are the only inputs. Everything else is derived. Advancing past stage 4 or 5 with the corresponding choice still `null` is invalid — block it and show why.

## Model data

```js
const ASSISTED_STAGES = [
  { n:0, key:'value',   name:'Value hypothesis', kind:'discover', effort:2   },
  { n:1, key:'scope',   name:'Scope',            kind:'discover', effort:2   },
  { n:2, key:'research', name:'Research',        kind:'discover', effort:5   },
  { n:3, key:'probe',   name:'Concept & probe',  kind:'concept',  effort:8   },
  { n:4, key:'decide',  name:'Decide',           kind:'concept',  effort:1   },
  { n:5, key:'spec',    name:'Specify',          kind:'spec',     effort:5   },
  { n:6, key:'build',   name:'Build',            kind:'build',    effort:9   },
  { n:7, key:'test',    name:'Test',             kind:'test',     effort:4   },
  { n:8, key:'release', name:'Release & measure',kind:'release',  effort:3   },
  { n:9, key:'record',  name:'Record',           kind:'record',   effort:0.5 }
];

const CONVENTIONAL_PHASES = [
  { n:1, name:'Discovery',      kind:'discover', effort:7   },
  { n:2, name:'Concept',        kind:'concept',  effort:10  },
  { n:3, name:'Design to spec', kind:'spec',     effort:5   },
  { n:4, name:'Handoff',        kind:'handoff',  effort:2   },
  { n:5, name:'Build',          kind:'build',    effort:15  },
  { n:6, name:'QA phase',       kind:'test',     effort:7   },
  { n:7, name:'Release',        kind:'release',  effort:5   },
  { n:8, name:'Retrospective',  kind:'record',   effort:0.2 }
];
```

Defects. `caughtAt` maps spec level to the assisted stage where the defect becomes detectable. `conventionalCaughtAt` is the equivalent phase in the control lane. Stage/phase `8` means it escaped to production.

```js
const DEFECTS = [
  { id:'D1', label:'Subscription contract mismatch',    introducedAt:5,
    caughtAt:{ triad:5, human:6, thin:7 }, conventionalCaughtAt: 5, conventionalIntroducedAt:3 },
  { id:'D2', label:'Invented plan status enum',          introducedAt:6,
    caughtAt:{ triad:6, human:6, thin:8 }, conventionalCaughtAt:null },
  { id:'D3', label:'Timezone at billing period boundary',introducedAt:6,
    caughtAt:{ triad:7, human:7, thin:8 }, conventionalCaughtAt: 7, conventionalIntroducedAt:5 },
  { id:'D4', label:'Promo lock-in state missing',        introducedAt:3,
    caughtAt:{ triad:3, human:6, thin:7 }, conventionalCaughtAt: 6, conventionalIntroducedAt:2 },
  { id:'D5', label:'Collections state missing',          introducedAt:3,
    caughtAt:{ triad:3, human:6, thin:8 }, conventionalCaughtAt: 8, conventionalIntroducedAt:2 },
  { id:'D6', label:'Cancellation effective date ambiguous', introducedAt:4,
    caughtAt:{ triad:5, human:7, thin:8 }, conventionalCaughtAt: 7, conventionalIntroducedAt:2 },
  { id:'D7', label:'Focus management on interrupt',      introducedAt:6,
    caughtAt:{ triad:7, human:7, thin:8 }, conventionalCaughtAt: 8, conventionalIntroducedAt:5 },
  { id:'D8', label:'Copy fails legal tone review',       introducedAt:5,
    caughtAt:{ triad:5, human:7, thin:7 }, conventionalCaughtAt: 7, conventionalIntroducedAt:3 },
  // conditional: only present when probesKept === true
  { id:'D9', label:'Prototype fidelity debt shipped',    introducedAt:3,
    caughtAt:{ triad:8, human:8, thin:8 }, conventionalCaughtAt:null,
    requires:'probesKept' }
];
```

`conventionalIntroducedAt` is the conventional phase a defect originates in, added because
`introducedAt` is written in assisted-stage numbers and the two lists number different things — without
it a rework arrow in the control lane points at a phase chosen by an unrelated index. It is used for the
arrow destination only; the rework predicate keeps using the shared `introducedAt`.

`conventionalCaughtAt: null` means the defect does not occur in the conventional run at all — D2 is a generated-output failure mode with no human equivalent, D9 requires a high-fidelity probe. State this in the UI where those defects appear; it is a point in the conventional method's favour and must not be hidden.

```js
const DEFAULT_ASSUMPTIONS = {
  specMultipliers: {            // applied to build and test stage effort
    thin:  { build:1.6, test:1.4, reviewRate:1.8 },
    human: { build:1.25, test:1.15, reviewRate:1.3 },
    triad: { build:1.0, test:1.0, reviewRate:1.0 }
  },
  specEffort: { thin:1.5, human:3, triad:5 },   // replaces stage 5 base effort
  defectBaseCost: 0.4,                          // person-days
  catchCostMultiplier: { 3:1, 4:1, 5:2, 6:5, 7:10, 8:25 },
  reworkLoopCost: 2.5,          // person-days, when caught>=6 and introducedAt<=5
  reviewHoursPerBuildDay: 1.2,
  calendarFromEffort: 0.62      // effort days -> calendar days, team parallelism
};
```

## Computation

```
defectCost(d, level)   = defectBaseCost * catchCostMultiplier[caughtStage]
reworkFires(d, level)  = caughtStage >= 6 && d.introducedAt <= 5
stageEffort(stage)     = base effort, except:
                           stage 5 -> specEffort[level]
                           stage 6 -> effort * specMultipliers[level].build
                           stage 7 -> effort * specMultipliers[level].test
                         plus defectCost for every defect caught at this stage
                         plus reworkLoopCost for every rework that fires here
reviewHours            = sum over build/test stages of
                           effort * reviewHoursPerBuildDay * reviewRate[level]
calendarDays           = totalEffort * calendarFromEffort
escaped                = defects whose caughtStage === 8
```

The conventional lane uses the same functions with `conventionalCaughtAt`, no spec multipliers, and its own phase list. Compute it once at load.

Totals must be recomputed from assumptions on every render. No caching, no incremental accumulation — the assumptions panel changes numbers mid-run and everything downstream must follow.

## Interaction contract

| Control | Behaviour |
|---|---|
| Next stage | `stageIndex++`. Blocked at 4 until `probesKept !== null`, at 5 until `specLevel !== null`. |
| Back | `stageIndex--`. Does not clear choices. |
| Reset | Full reset including choices; assumptions retained. |
| Reset assumptions | Restores `DEFAULT_ASSUMPTIONS` only. |
| Probe choice | Two buttons, neutral labels: `Keep the prototype` / `Delete the prototypes`. No hint of correctness. |
| Spec choice | Three buttons: `Notes on the designs` / `Written up for the team` / `Written for team, tools and tests`. Described by what gets written, never by outcome. |

Button labels are presentation and may be reworded for a reader who has not read this document set; the
`thin` / `human` / `triad` keys are the contract and do not change with them. The labels above were
rewritten from `Thin annotations` / `Human-readable only` / `Full triad`, which named the spec triad by
its internal numbering and meant nothing to a first-time reader. Any replacement must still describe
what gets written rather than what results, and the three must stay parallel in length and register —
one option reading longer or heavier than the others is itself a hint of correctness.
| Assumption inputs | `type="number"`, `step` appropriate to magnitude, immediate recompute on change. Reject negatives and non-numeric; keep last valid value. |

## Forbidden

- Do not tune the numbers so the triad path wins by more than the assumptions produce.
- Do not add copy that celebrates the assisted track or disparages the conventional one.
- Do not hide, collapse by default, or de-emphasise the assumptions panel.
- Do not present any computed figure without the word "assumed" reachable from the same screen.
- Do not add persistence, analytics, network calls, or model calls of any kind.
- Do not change `DEFECTS` or the stage lists to improve the demo's outcome. If the model produces an uncomfortable result, that is the result.

## Known limitations — state these in the UI footer

Every number is invented. The defect table is a reconstruction, not observation. Both lanes assume existing tests, a design system, and a documented domain; without those the assisted lane would be slower, and the model does not represent that case. A simulation of AI benefits, built with AI, argues in a circle — only a recorded run on real work settles it.
