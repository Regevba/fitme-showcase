# 185 Findings, 6 Sprints, 100% In-Project Closure

> A 4-layer audit surfaced 185 findings across 6 domains in a single sweep. A 6-sprint remediation program closed 183 of them in five days, deferred 2 on documented external blockers, and produced a case study per sprint along the way. The first time the framework was used to close findings from its own audit.

## Context

On 2026-04-16 a 4-layer meta-analysis audit produced 185 findings across 6 domains — UI, Backend, AI, Design System, Framework, Tests. The audit itself was a framework milestone: parallel domain dispatch, risk-weighted deep dives, cross-reference against 18 prior case studies, and an external validation layer that served as the independent anchor. What happened next is the rarer artifact: a full remediation program that took those 185 findings and drove them to documented closure inside five days.

Six sprints ran in sequence — one bulk remediation (50 findings in a single sweep), four multi-PR decomposition sprints (M-3 / M-1 / M-2 / M-4), and one parallel backend/security track (Path A). Every sprint shipped a case study alongside its code. Every finding ended up either closed in-repo or classified as deferred on a documented external blocker. Nothing was silently dropped.

The final arithmetic: **183 of 185 closed in-repo. 2 deferred on named external blockers. 100% in-project closure rate.**

---

## The 185 Findings

The audit ran 4 layers in sequence:

| Layer | Purpose | Output |
|---|---|---|
| 1 — Surface Sweep | 6 parallel domain agents | 140 findings |
| 2 — Deep Dive | 3 risk-weighted batches | 45 findings (incl. 3 discovery-level systemic patterns) |
| 3 — Cross-Reference | Compare against 18 prior case studies | 9 recurring patterns, 2 regressions, 4 predictions, 5 new categories |
| 4 — External Validation | External automated checks + external human review | validated layers 1-3, surfaced no new unique items |

**Distribution by severity:** 12 critical, 49 high, 90 medium, 34 low.

The top discovery — DEEP-AI-015 — was a *fabrication-over-nil systemic pattern* in AI adapters: 12 separate sites where missing fields were being replaced with plausible-looking fabricated values instead of surfacing `nil` or an explicit unknown. That is not a bug in one place. It is a cross-cutting pattern that required a cross-cutting fix.

---

## Six Sprints, Six Shapes

Each sprint ran a different shape because each addressed a different kind of work.

| # | Sprint | Shape | Findings Closed | Cumulative |
|---|---|---|---|---|
| 1 | Post-stress-test bulk remediation | 21 PRs sequenced, serial dispatch after a concurrent-wave abort | 50 | 127 → 177 (95.7%) |
| 2 | M-3 Design System + Dark Mode | 4 PRs decomposed (3a/b/c/d), case study in closing PR | 3 | 177 → 180 (97.3%) |
| 3 | M-1 SettingsView Decomposition | 4 PRs decomposed, 1,170 → 294 lines | 1 (UI-002) | 180 → 181 (97.8%) |
| 4 | M-2 MealEntrySheet Decomposition | 5 PRs decomposed, 1,104 → 140 lines, 17 @State vars → 0 | 1 (UI-004) | 181 → 182 (98.4%) |
| 5 | M-4 XCUITest Infrastructure | 3 PRs + plan addendum (first M-series with a mid-flight abort and retry) | 1 (TEST-025) | 182 → 183 (98.9%) |
| 6 | Path A (parallel backend + security track) | 3 PRs closing 6 of 8 findings; 2 deferred | 6 | (counted above) |

**The M-series shape** — `M-{n}a` (riskiest / largest sub-change) → `M-{n}b` + `M-{n}c` (low-risk extensions) → `M-{n}d` (case study + closing PR) — emerged during Sprint 2 and was reused for Sprints 3, 4, and 5. It is now the default shape for any multi-PR decomposition-style remediation on this project.

---

## What Got Deferred — and Why It Matters

Two findings remain open:

- **BE-024** — an Edge Function remediation that requires its own multi-PR chain; ships when the Edge Function owner opens the chain.
- **DEEP-SYNC-010** — a CloudKit → Supabase image bridge that requires a production data migration; ships in a dedicated data-migration session.

Neither is abandoned. Both have documented closure paths, named external blockers, and explicit unblock triggers. The program treats *deferred on documented external blocker* as a terminal audit state distinct from *closed* and distinct from *open*. Every audit accounting now has this as a third bucket.

This matters because "183 / 185 in-project closure" can be lied about easily: drop 2 findings silently and the number looks the same. Classifying each deferred finding with a reason a reviewer can audit is what keeps the accounting honest.

---

## Nine Framework Bugs Surfaced by the Program

Running a remediation program at this scale is itself a stress test of the framework. Nine framework-level bugs were surfaced by the work:

| Bug | Surface | Resolution |
|---|---|---|
| F1 | Concurrent-dispatch worktree permission gap | Wave 1 aborted; serial pivot |
| F2 | Plan-doc contract drift | Patched; validated 10 of 10 agents |
| F3 | Spec-to-plan link validation | Patched; validated 10 of 10 agents |
| F4 | Audit-findings.json live-sync contract | Patched; validated 10 of 10 agents |
| F5 | Case-study-monitoring concurrent-session conflict | Wave 2 aborted; concurrent blocked until upstream |
| F6 – F9 | Four concurrent-dispatch permission-propagation bugs | Unpatched; concurrent dispatch remains blocked at the framework layer |

**Pattern:** F1, F5, F8, F9 are all *concurrent-dispatch* bugs. The program's original plan was parallel sub-agent sprints. It pivoted to serial execution not as a fallback but as the working pattern — and completed successfully. Concurrent dispatch stays blocked pending F8 / F9 upstream fixes.

**F2, F3, F4 were validated at 10 of 10 agents** — their contracts are production-ready and have survived concurrent validation. The plan-doc, spec-to-plan, and audit-findings live-sync are all load-bearing parts of this program's accounting.

---

## The Self-Referential Problem, Handled Honestly

This is the first time the framework was used to close findings produced by its own audit. The auditor and the remediator are the same system. The self-referential bias is real and documented in the audit report itself: the audit cannot detect what the framework cannot perceive.

The counter-balance is Layer 4 — external validation. External automated checks (build, lint, static analysis, external tests) and external human review (documented in the `meta-analysis-validation-2026-04-16.md` companion doc) serve as the independent anchor that internal layers 1-3 cannot provide. Without Layer 4 the audit becomes a confidence machine: it finds what it expects to find and misses what it can't see.

Six UX principles came out suspiciously all-green in the internal audit. The audit report flagged them explicitly as suspicious rather than claiming them as wins. This is what honest self-audit looks like.

---

## Key Takeaways

- **Bulk + decomposition + parallel track is the right mix for a large remediation.** No single sprint shape would have covered all 50+ affected PRs. One sprint handled the bulk; four handled coordinated multi-PR work; one handled a parallel backend/security track.
- **A remediation program is a framework stress test.** 100% in-project closure is only meaningful because the program kept running *through* nine framework bugs — documenting them, pivoting around them, and shipping. The bugs did not stop the program; they became inputs to it.
- **Concurrent case study writing is cheaper than retrospective case study writing.** Once M-3 established the pattern, every subsequent sprint produced its case study alongside the code. Retrospective case studies on older sprints took days; concurrent ones took hours.
- **External-blocker classification keeps audit accounting honest.** Two deferred findings is not two findings dropped. The difference — a named blocker, a documented closure path, an explicit expected-unblock trigger — is what lets 183 / 185 be called 100% in-project closure without lying.
- **Layer 4 external validation is load-bearing, not decorative.** Internal audit layers cannot detect what the framework itself cannot perceive. Layer 4 is the only layer that can find those blind spots.
- **Stacked PRs on a single decomposition need a merge-serialization discipline.** M-2 surfaced the "stacked PR misfire" where merging an intermediate PR destabilizes the downstream ones. Either merge the full stack atomically or rebase downstream PRs after each intermediate merge — but choose one explicitly.
