# Act 4: Case Studies — The Framework Evolution, Told in Data

> 13 framework milestones plus 12 engineering deep dives, followed by an audit-level mirror of every case study upstream. One complexity model. A linear timeline that traces how an AI-assisted PM framework evolved from a 6.5-hour pilot to hardware-aware dispatch — with honest measurements at every step.

**Two layers in this folder, by design.**

- **#01–#25 (curated):** rewritten for narrative clarity, ~35% shorter than upstream, self-contained for outside readers.
- **#26–#47 + `meta-analysis/`:** verbatim mirror of `FitTracker2/docs/case-studies/` for audit-level fidelity. Filenames track upstream slugs; content is unedited.

Curation for the public site (fitme-story.vercel.app) draws from this superset; the showcase repo is the comprehensive corpus.

## Context

These case studies document the evolution of an AI-assisted PM framework across 19 features built over 4 weeks. Unlike typical "we shipped fast" narratives, every claim here is backed by a normalized complexity metric (Complexity Units, or CU) that accounts for task count, work type, and difficulty factors like auth integration, UI complexity, and architectural novelty. The result: apples-to-apples velocity comparisons across features of wildly different scope.

**Read them in order.** The numbering follows the framework's version timeline — v2.0 through v7.0. Each study builds on the previous, and the progression from "manual and slow" to "parallel, measured, and hardware-aware" is the point.

The measurement methodology is described in [How We Normalized Complexity Across 16 Different Features](normalization-model.md), and independently validated in [External Validation](meta-analysis-validation.md).

---

## Timeline

| # | Case Study | Version | Story Beat | Key Result |
|---|-----------|---------|------------|------------|
| 1 | [The Pilot — Onboarding](01-onboarding-pilot.md) | v2.0 | The beginning | 15.2 min/CU baseline established |
| 2 | [6 Refactors, 6.5x Speedup](02-framework-evolution.md) | v2.0→v4.3 | Scale | 6.5h → 1h across 6 identical-scope refactors |
| 3 | [Testing AI Output Quality](03-eval-driven-development.md) | v4.4 | Quality | 29 eval cases, 6-phase lifecycle, +71% vs baseline |
| 4 | [Most Complex Feature at Refactor Speed](04-user-profile.md) | v4.4 | Discipline | 2x file count, same 2h wall time, first mandatory case study |
| 5 | [Designing Software Like a Chip](05-soc-on-software.md) | v5.0 | Hardware inspiration | 63% overhead reduction, free context nearly doubled |
| 6 | [The Fastest Feature (+86%)](06-auth-flow-velocity.md) | v5.1 | Peak velocity | 2.1 min/CU — best single-feature result in dataset |
| 7 | [First Feature Under New Architecture](07-ai-engine-architecture.md) | v5.1 | Breadth | 45% cache hit rate, 66% faster than baseline |
| 8 | [4 Features in 54 Minutes](08-parallel-stress-test.md) | v5.1 | Parallelism | 12.4x throughput, 35 tests, 0 failures |
| 9 | [What Breaks at Scale — and the Fix](09-dispatch-intelligence.md) | v5.2 | Learning | Tool usage -48%, variance -84%, review gate catches real bugs |
| 10 | [From Luck to Design](10-parallel-write-safety.md) | v5.2 | Safety | Deterministic file isolation, self-improving region detection |
| 11 | [Stopped Estimating, Started Measuring](11-measurement-v6.md) | v6.0 | Self-awareness | 7 of 9 metrics moved from estimated to deterministic |
| 12 | [Hardware-Aware Dispatch](12-hadf.md) | v7.0 | Hardware | 17 chip profiles + 7 cloud signatures in 120 min |
| 13 | [185 Findings Full-System Audit](13-full-system-audit.md) | v7.0 | Audit | 185 findings, 12 critical, honest self-referential bias report |

---

## Engineering Deep Dives

Six case studies that take a single decision, bug, or component and examine it in depth — complementing the version-timeline arc above with engineering-level detail.

| # | Case Study | Story Beat | Why It Earns Its Own Study |
|---|-----------|------------|-----------------------------|
| 14 | [Framework Story Site Meta-Build](14-framework-story-site.md) | Meta-recursion | The site describing the framework, built by the framework it describes |
| 15 | [SSR Regression Post-Mortem](15-ssr-regression.md) | Debugging | Blank-main production bug, root cause, fix, and guardrail added |
| 16 | [DispatchReplay Component](16-dispatchreplay.md) | Design-to-code | How a scroll-interactive trace-replay component was specified and built |
| 17 | [Lego as PM-Flow Metaphor](17-lego-pmflow.md) | Pedagogy | The metaphor that unlocked the mental model for the framework's ecosystem |
| 18 | [Two Project-Wide Rules Born in One Refactor](18-home-today-screen.md) | Rules emerge from features | How Home v2 codified the V2 Rule and the screen-prefixed analytics convention |
| 19 | [The Biggest Screen Ran the Fastest](19-training-plan-v2.md) | Scale | A 2,135-line monolith shipped in 5 hours with the best complexity-normalized velocity of any v2 pass |
| 20 | [185 Findings, 100% In-Project Closure](20-audit-remediation-program.md) | Audit → Remediation | The first time the framework was used to close findings from its own audit — 6 sprints, 183/185 closed, 2 deferred on documented external blockers |
| 21 | [Smart Reminders — Four Features in 12 Hours](21-smart-reminders.md) | Stress test | 5-guard notification scheduler shipped as 1 of 4 features during the v5.1 parallel stress test; surfaced a PM-hygiene gap in stress-test mode |
| 22 | [Shipped Without a Door](22-push-notifications.md) | Shipped-not-live | The priming view shipped tested and merged, but no entry point was ever wired — the feature that documents "complete" ≠ "reachable" |
| 23 | [Phase-Complete Is Not Feature-Live](23-import-training-plan.md) | Archival vs deletion | Parser stack + 23 tests shipped cleanly; entry-point views annotated HISTORICAL three days later. Archive rather than delete — preserves the revival path |
| 24 | [Six Features That Didn't Earn a Case Study](24-backlog-features-roundup.md) | Honest accounting | When the case-study-every-feature rule meets six features that either pre-dated it or were research-only — a three-condition threshold emerges |
| 25 | [The Drift That Detected Itself](25-integrity-cycle.md) | v7.1 self-observation | A one-time audit uncovered 7 state-file lies spanning up to 11 days. v7.1 converts that audit into a 72-hour recurring cycle — the first framework capability whose trigger is wall-clock elapsed, not a feature action |

---

## The Complexity Unit (CU) Model

Raw metrics like wall time and file count are meaningless without normalization. A 22-task UI refactor with auth integration is fundamentally different from a 4-task backend enhancement.

**Formula:** `CU = Tasks x Work_Type_Weight x (1 + sum(Complexity_Factors))`

- **Work type weights** range from 0.3 (chore) to 1.0 (full-lifecycle feature)
- **Complexity factors** add weight for UI views (+0.15 to +0.45), auth integration (+0.5), runtime testing (+0.4), new models (+0.1 to +0.3), cross-feature scope (+0.2), and design iterations (+0.10 to +0.25 per round)
- **Primary metric:** minutes per CU (lower is better) — enables direct comparison across features of any size

The model fits a power law with R-squared = 0.87, suggesting the improvement curve has not yet plateaued. Full methodology: [Normalization Model](normalization-model.md).

---

## Supporting Analysis

| Document | What It Covers |
|----------|---------------|
| [Normalization Model](normalization-model.md) | The CU complexity formula, factor definitions, and retroactive normalization of all 16 features |
| [ROI Retrospective](meta-analysis.md) | What-if analysis: had we instrumented measurement from day one, what would we have learned earlier? |
| [External Validation](meta-analysis-validation.md) | Independent review confirming arithmetic consistency, identifying measurement gaps, and recommending next steps |
| [Data Quality Tiers](data-quality-tiers.md) | T1 (Instrumented) / T2 (Declared) / T3 (Narrative) labeling convention. Every quantitative claim in a case study, PRD, or meta-analysis must carry one |
| [Case Study Template](case-study-template.md) | Canonical template for new case studies (frontmatter, sections, tier tags) |
| [PM-Workflow Skill](pm-workflow-skill.md) | The skill that orchestrates the 9-phase lifecycle |
| [FitTracker Evolution Walkthrough](fittracker-evolution-walkthrough.md) | Chronological narrative of the codebase's evolution |
| [UI-Audit Baseline Burndown](ui-audit-baseline-burndown.md) | Hard-gate burndown of the 27 P0 design-system findings |

---

## Audit-Level Mirror (#26–#47)

Verbatim mirror of `FitTracker2/docs/case-studies/*.md` for case studies that don't have a curated #01–#25 entry. Ordered by git creation date.

### UCC + Operations (April 13)

| # | Case Study | Topic |
|---|-----------|-------|
| 26 | [Cleanup + Operations Control Room](26-cleanup-control-room.md) | Maintenance + UCC-adjacent ops surface |
| 27 | [Control Center IA Refresh](27-control-center-alignment-ia-refresh.md) | UCC information-architecture realignment |

### Audit Remediation Track (April 17–20)

| # | Case Study | Topic |
|---|-----------|-------|
| 28 | [Audit Remediation (early)](28-audit-remediation.md) | Early-stage remediation pass that preceded the 185-findings program |
| 29 | [Orchid AI Accelerator](29-orchid-ai-accelerator.md) | 26K+ design-space runs; foundation for v7.0 HADF |
| 30 | [Audit v2 — Concurrent Stress Test](30-audit-v2-concurrent-stress-test.md) | Concurrent dispatch stress test; surfaced framework bugs F1–F9 |
| 31 | [Audit v2 — G1 UI](31-audit-v2-g1-ui.md) | Group 1: UI surface findings |
| 32 | [Audit v2 — G2 Tests](32-audit-v2-g2-tests.md) | Group 2: test infrastructure findings |
| 33 | [Audit v2 — G3 AI](33-audit-v2-g3-ai.md) | Group 3: AI engine findings |
| 34 | [Audit v2 — G4 Backend](34-audit-v2-g4-backend.md) | Group 4: backend findings |
| 35 | [Audit v2 — G5 Design System](35-audit-v2-g5-ds.md) | Group 5: design-system findings |
| 36 | [Audit v2 — G6 Config](36-audit-v2-g6-config.md) | Group 6: config / build findings |
| 37 | [M-1 Settings Decomposition](37-m-1-settings-decomposition.md) | 1170-line monolith → 294 lines + 8 new files |
| 38 | [M-2 MealEntrySheet Decomposition](38-m-2-mealentrysheet-decomposition.md) | 1104 → 140 lines (~87% reduction); 17 @State vars → 0 |
| 39 | [M-3 Design System Completion](39-m-3-design-system-completion.md) | Token pipeline 60% → 100%; 29 → 38 dark-mode colorsets |
| 40 | [Post-Stress-Test Audit Remediation](40-post-stress-test-audit-remediation.md) | 21 PRs (#96–#116), 50 audit findings closed, F1–F9 documented |
| 41 | [M-4 XCUITest Infrastructure](41-m-4-xcuitest-infrastructure.md) | First M-series feature with attempt-1 abort + plan-addendum recovery |
| 42 | [Original README Redesign](42-original-readme-redesign.md) | Top-level README rewrite for project framing |

### Data Integrity Trilogy (April 24–27)

| # | Case Study | Topic |
|---|-----------|-------|
| 43 | [Data Integrity Framework v7.5](43-data-integrity-framework-v7-5.md) | Eight cooperating defenses born from the Gemini audit |
| 44 | [Mechanical Enforcement v7.6](44-mechanical-enforcement-v7-6.md) | 7 Class B → A promotions; per-PR review bot + weekly cron |
| 46 | [Validity Closure v7.7](46-framework-v7-7-validity-closure.md) | 5 new gates + advisory; closes A1–A5 + B1–B2 + C1 |

### Paused / In-Progress (April 27)

| # | Case Study | Topic |
|---|-----------|-------|
| 45 | [Auth Polish v2](45-auth-polish-v2.md) | Paused 2026-04-28 post-A1 (stub) |
| 47 | [Onboarding v2 Retroactive](47-onboarding-v2-retroactive.md) | Retroactive PM-workflow application to onboarding v2 (stub) |

### Meta-Analysis Subfolder

Verbatim mirror of `FitTracker2/docs/case-studies/meta-analysis/`:

| Document | What It Covers |
|----------|---------------|
| [README](meta-analysis/README.md) | Subfolder index |
| [Independent Audit — Gemini 2.5 Pro](meta-analysis/independent-audit-2026-04-21-gemini.md) | Source trigger for the v7.5 framework bump |
| [Meta-Analysis 2026-04-21](meta-analysis/meta-analysis-2026-04-21.md) | Cross-feature synthesis |
| [Meta-Analysis Validation 2026-04-16](meta-analysis/meta-analysis-validation-2026-04-16.md) | Verbatim source of the curated [meta-analysis-validation.md](meta-analysis-validation.md) |
| [What If V6 From Day One](meta-analysis/what-if-v6-from-day-one-2026-04-16.md) | Verbatim source of the curated [meta-analysis.md](meta-analysis.md) |
| [V7.5 Advancement Report](meta-analysis/v7-5-advancement-report.md) | Tier-by-tier closure status |
| [Tier-Tag Checker Baseline](meta-analysis/tier-tag-checker-baseline.md) | Baseline measurement of T1/T2/T3 compliance |
| [Unclosable Gaps](meta-analysis/unclosable-gaps.md) | The 5 mechanical gaps that cannot be closed by automation |

---

## How to Read These

**If you have 2 minutes:** Read this page. The timeline table tells the whole story.

**If you have 10 minutes:** Read [6 Refactors, 6.5x Speedup](02-framework-evolution.md) — it contains the core thesis with the most data.

**If you have 30 minutes:** Read the full sequence 01 through 13 in order. Each builds on the previous, and the progression from pilot to hardware-aware dispatch is the narrative arc.

**If you are evaluating methodology:** Start with the [Normalization Model](normalization-model.md) and [External Validation](meta-analysis-validation.md). They contain the statistical caveats and limitations that make the rest of the data trustworthy.
