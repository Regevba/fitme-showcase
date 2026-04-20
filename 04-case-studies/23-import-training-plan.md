# The Feature That Shipped But Was Never Visible

> A CSV / JSON / Markdown parser, a 37-entry exercise mapper, a three-tier confidence scorer, 23 unit tests — all shipped in two days under a parallel stress test. Three days later the entry-point views were annotated `HISTORICAL —` as dead code. A cautionary record of what "phase complete" means when the gate fires on a green build rather than on a reachable screen.

## Context

Import Training Plan was one of four features chosen for the v5.1 parallel-stress-test run. The experiment: advance four PM workflows simultaneously to see whether the framework could produce clean builds under concurrent load. It could — the stress-test log records *"3rd consecutive clean build under parallel load"* on the phase that landed this feature's core.

The feature's PRD was ambitious. Import from CSV, from JSON, from paste-to-parse flows catching ChatGPT / Claude / Gemini output. A 3-tier confidence scorer for mapping arbitrary exercise names to FitMe's 87-exercise library. A preview screen with green-check / orange-pencil / red-warning indicators. Six analytics events under an `import_` prefix. Success criteria: 80% import success rate, 90% mapping accuracy, <5-minute time-to-first-imported-workout.

Two days after PRD approval the state file recorded `phase=complete`, note: *"All tasks implemented. BUILD SUCCEEDED. Feature complete."* There was no pull request. There was no branch. Commits landed on `main`.

Three days after that, a UI audit caught what the green build had missed: the two views that were supposed to be the entry point — `ImportSourcePickerView` and `ImportPreviewView` — were never instantiated by any tab, sheet, or navigation destination. Finding UI-015 in Sprint D of the meta-analysis audit. The fix, committed on 2026-04-17, was to leave the files in the build target but annotate them with `HISTORICAL —` headers. The production import flow, if it is ever revived, will need a new entry point.

---

## What Actually Shipped

| Layer | Status | Evidence |
|---|---|---|
| `ImportParser` protocol + CSV / JSON / Markdown implementations | Reachable, exercised | 174 lines, 23 unit tests |
| `ExerciseMapper` with 37 aliases, 3-tier confidence | Reachable, exercised | 88 lines, 4 mapper tests pass |
| `ImportOrchestrator` — `@MainActor` state machine (.idle → .parsing → .mapping → .preview → .success) | Reachable, exercised | 60 lines, 4 orchestrator tests pass |
| `ImportSourcePickerView` — 2x2 source grid + paste field | **Archived as HISTORICAL** | Never wired into a tab |
| `ImportPreviewView` — confidence icons, summary bar | **Archived as HISTORICAL** | Only referenced by the archived picker |
| 6 `import_*` analytics event constants | Defined, **never fired** | `AnalyticsProvider.swift` lines 232-242 |
| `ImportOrchestrator.confirmImport()` persist to `TrainingProgramData` | **Not implemented** — transitions state only |
| Prompt preservation (IT-11) | Not implemented |
| Progressive alias learning (IT-12) | Not implemented — `aliases` is `let`, not `var` |
| PDF text extraction (IT-13) | Not implemented |

The bottom three rows are P1 deferrals, explicitly scoped out of the first push. The top five are what the funnel produced. The shape is a working, tested, unreachable feature.

---

## The Gate That Didn't Fire

The PM workflow's completion gate fired on two signals: *all tasks implemented* and *BUILD SUCCEEDED*. Both were true. What the gate didn't check was: *is this feature reachable from any user-facing surface?*

Later features in the same project — the M-series that ran a month later — were more conservative. M-1 (Settings decomposition), M-2 (MealEntrySheet extraction), M-3 (design system completion), M-4 (XCUITest infrastructure) each required merged pull requests with visible entry points before the state machine would accept `phase=complete`. Import Training Plan is the clearest pre-M-series case that earned the retrofit.

The v6.0 measurement framework that landed shortly after added eval-coverage gates partly in response to this pattern. *Shipped code* and *tested code* are different from *reached code*, and the framework now knows it.

---

## Why the Archival Was the Right Call

The audit's fix on finding UI-015 was not to delete the orphan views but to annotate them. The file header reads:

> HISTORICAL — never wired into a tab/sheet. Audit UI-015 (2026-04-16) identified this view + ImportPreviewView as dead code. Retained for reference; the production import flow (if revived) should use ImportOrchestrator from a new entry point built on current design system.

Retaining the views as reference material is cheaper than deletion for one reason: the parser + mapper + orchestrator stack *is* durable. It is decoupled from SwiftUI. It has 23 tests. It honors the 2026-04-08 analytics naming rule. When a revival happens — and the feature is still ranked HIGH priority in the PRD — the entry point is the only thing that needs fresh design. The parser does not need to be rebuilt. The mapper does not need to be re-seeded. The state machine does not need to be re-verified.

Revival is three changes:
1. Remove the `HISTORICAL` headers.
2. Wire a Training-tab menu item that presents `ImportSourcePickerView()`.
3. Make `ImportOrchestrator.confirmImport()` actually persist to `TrainingProgramData` (currently it only transitions state).

---

## The Useful Pattern

Import Training Plan is the project's cleanest record of one truth: a framework can produce working code under parallel load, maintain green builds, and still ship features that nobody can use. The gates that catch the wrong thing pass; the gates that catch the right thing have to be added later.

The 2026-04-13 "every feature gets a case study" rule exists partly because of cases like this. Without the rule, the state file would say `complete` and the memory would move on. With the rule, the record survives long enough for the lesson — *reachability is a gate*, not an assumption — to be available the next time a feature funnel runs.

---

## Key Takeaways

- **Phase-complete is not feature-live.** The state machine accepted `complete` on BUILD SUCCEEDED + tasks implemented. No check existed for "user can reach this." Post-audit gates were added to later features in response.
- **The parser layer is the durable artifact.** 23 tests, 607 production lines, `@MainActor` state machine — all still in the build target and salvageable. The archived views are UI reference material for a future revival, not waste.
- **Archival beats deletion when the underlying infrastructure works.** `HISTORICAL —` headers let dead code coexist with live code and tell the revival story without the git-archaeology cost.
- **Analytics constants without call sites are a taxonomy win, not a funnel win.** The `import_` event names are reserved, but the funnel is empty. Revival should wire call sites in the same PR.
- **Parallel-dispatch wins produce different kinds of gaps than serial dispatch does.** Four PRDs concurrent + three clean builds under load is a real framework win. It is also why nothing noticed the wiring gap until a separate audit pass ran.
