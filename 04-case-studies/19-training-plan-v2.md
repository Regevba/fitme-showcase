# The Biggest Screen Ran the Fastest

> A 2,135-line monolith — the largest view in the app — refactored in five wall-clock hours with the best complexity-normalized velocity of any v2 pass. The feature that proved file size does not predict wall time once a learning cache is in play.

## Context

Training Plan was by a wide margin the largest surface in the app: 2,135 lines in a single file, 13 nested types inside it, 32 audit findings — more than any other v2 refactor. The README had queued Training with a one-line brief: *"biggest surface in the app — stress-test the per-screen alignment process."*

Training was also the first v2 refactor to run under framework version v4.0, which introduced the L1 learning cache and the reactive data mesh. It therefore carried two jobs at once: deliver a production-grade refactor of the most coupled screen in the app, and serve as the measurement point for whether the cache actually helped on work at scale.

Both jobs shipped. The refactor closed in five wall-clock hours with a 40% first-run cache hit rate, and the complexity-normalized velocity came in at 0.23 hours per 100 lines of v1 — the best number across all six screen refactors, despite Training being the largest file.

---

## The Refactor

| Field | Value |
|--------|--------|
| v1 file size | 2,135 lines (1 file, 13 nested types) |
| v2 footprint | 1,819 lines across 7 files (1 container + 6 extracted views) |
| Audit findings | 32 (8 P0 / 16 P1 / 8 P2) |
| Tractability classification | 20 auto-applicable / 7 needs-decision / 5 needs-new-DS |
| Tasks | 16 in 4 layers |
| Analytics events | 12 (highest count of any screen) |
| Tests | 16 (12 events + screen-prefix + consent + parameter convention + a regression) |
| Design tokens added | 2 |
| Components promoted to the design system | 3 |
| Wall time | ~5 hours |
| L1 cache hit rate (first run) | ~40% |
| Framework version | v4.0 |

Before Training, the screen had zero analytics events. Every user action inside the workout loop was invisible in GA4. The PRD treated this as a full-screen instrumentation buildout — naming convention, consent gating, parameter schema, test coverage — not an afterthought appended to the design spec.

---

## Why the Biggest File Ran the Fastest

Complexity-normalized velocity across the six screen refactors:

| Screen | v1 Lines | Wall Time | h per 100 lines | Framework Version |
|---------|----------|-----------|-----------------|-------------------|
| Onboarding | 1,106 | 6.5h | 0.59 | v2.0 |
| Home | 703 | 1 day | outlier* | v3.0 |
| **Training** | **2,135** | **5.0h** | **0.23** | **v4.0 (L1 cache, 40%)** |
| Nutrition | 487 | 2.0h | 0.41 | v4.1 |
| Stats | 312 | 1.5h | 0.48 | v4.1 |
| Settings | 289 | 1.0h | 0.35 | v4.1 |

*Home was an outlier on wall time because it was simultaneously establishing the v2 file convention, spawning three sub-features, and integrating external tools for the first time. Its raw refactor-only time was consistent with the trend.

Training — the largest file — achieved the best complexity-normalized speed. This is the data point that cannot be explained by practitioner learning. The practitioner got faster with each refactor, but Training was only the third, well before learning effects plateau. The only remaining explanation is the L1 cache: audit patterns and design-token mappings primed from Home v2 made Training's research phase run far faster than raw file size would predict.

The three v4.1 screens that followed show the L2 shared cache stabilizing at 55-70% hit rate with a flat normalized velocity around 0.35-0.48. That is what a mature cache looks like — no longer improving, but no longer being rebuilt from scratch either.

---

## Two-Axis Audit Classification Enabled Parallel Dispatch

The audit tagged every finding on two independent axes:

- **Severity** — P0 (blocks ship) / P1 (should fix) / P2 (defer).
- **Tractability** — auto-applicable (just change the code) / needs-decision (user call required) / needs-new-token (DS evolution) / needs-new-component (DS evolution).

Severity alone tells you what matters. Tractability tells you what is dispatchable without further user input. Together they enabled parallel dispatch on 20 of 32 findings from the moment the PRD landed — no blocking on decisions, no waiting for design system additions.

The foundation layer (T1-T8) ran concurrently on independent subtasks: token additions, component scaffolding, the UX foundations walk, the accessibility audit, the state-coverage plan. This was the first time the framework dispatched more than four concurrent subagent tasks and recombined their results into the assembly phase without merge conflicts.

---

## Planning Velocity, Not Wall Time, Is the Framework Metric

Raw wall time is dominated by file size, which is a feature property, not a framework property. Planning velocity — audit findings surfaced per hour during research — is the cleanest signal of framework capability:

| Screen | Framework Version | Findings per Hour |
|---------|-------------------|-------------------|
| Onboarding | v2.0 | 3.7 |
| **Training** | **v4.0** | **6.4** |
| Nutrition | v4.1 | 11.5 |
| Stats | v4.1 | 13.5 |
| Settings | v4.1 | 16.0 |

Training's 6.4 findings/h represents a 1.7x jump from Onboarding — the first real payoff from the cache and parallel-dispatch infrastructure. The v4.1 screens that followed doubled it again as L2 kicked in.

---

## Kill Criteria at the Granularity of the Most-Fired Event

Training's kill criterion is: *per-exercise event drop-off greater than 15% versus `training_workout_start`, across weekly / 30-day / 90-day cohorts.* This sounds innocuous but is aggressive by design: because `training_set_completed` fires many times per workout, a 15% regression across weekly cohorts is detectable within days, not quarters.

The rule the feature validated: kill criteria should be set at the granularity of the most-fired event on the screen. Kill criteria tied to rare events — weekly conversions, session completions — take too long to trigger and provide no early warning. Kill criteria tied to the most granular event inside the core loop surface regressions while they are still small.

---

## Component Promotion as a Design System Evolution Path

Three of Training's six extracted views were promoted from feature-local to app-wide: `StatusDropdown` (a flexible activity type picker), `MilestoneModal` (weekly goal progress), and `SetCompletionIndicator` (visual set state). None of them were designed top-down. They were feature-local components the UX audit flagged as *"this should be reused elsewhere"* — and became part of the design system on the way out.

The DS evolution path this validated — feature-local first, promoted when the second consumer appears — is cheaper than the top-down path where the design system predicts what every feature will need. Prediction is expensive and often wrong; promotion is cheap and always calibrated to actual use.

---

## Deferred by Design

Two downstream features were spun out rather than absorbed into Training's scope:

- **Rest day positive-experience redesign** — own PM cycle.
- **Advanced data fusion and AI engine exercise recommendations** — own PM cycle.

Eight P2 audit findings were deferred to a v2.1 follow-up pass. Training shipped with a clean scope; none of the deferred work stalled the pilot run.

---

## Key Takeaways

- **File size does not predict wall time under a mature framework.** Training was 3x larger than Home and shipped in 5 hours versus Home's full day. The L1 cache is what closed that gap.
- **Two-axis audit classification (severity × tractability) is what enables parallel dispatch.** Without the tractability axis, every finding is a candidate for blocking on user input. With it, auto-applicable findings can dispatch immediately.
- **A full-screen analytics buildout is a distinct PRD shape from an incremental one.** Going from zero events to twelve required a naming convention, a consent model, a parameter schema, and a test matrix — not a bolt-on to a design spec.
- **Component promotion is opportunistic, not planned.** Three components were lifted into the design system during Training. None were designed for reuse up front; all were flagged by the UX audit as reusable at the moment they were extracted.
- **Complexity-normalized metrics make framework claims defensible.** "Training ran in 5 hours" is a feature observation. "Training shipped at 0.23 h/100 lines, beating every other v2 pass on normalized velocity despite being the largest file" is a framework claim — and it is the one the cache actually earned.
