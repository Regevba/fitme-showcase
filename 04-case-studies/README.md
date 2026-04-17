# Act 4: Case Studies — Measuring What Matters

> Eight features. One complexity model. A longitudinal dataset that proves AI-assisted development gets measurably faster with every iteration.

## Context

These case studies document the evolution of an AI-assisted PM framework across 17 features built over 4 weeks. Unlike typical "we shipped fast" narratives, every claim here is backed by a normalized complexity metric (Complexity Units, or CU) that accounts for task count, work type, and difficulty factors like auth integration, UI complexity, and architectural novelty. The result: apples-to-apples velocity comparisons across features of wildly different scope.

The measurement methodology is described in [How We Normalized Complexity Across 16 Different Features](normalization-model.md), and independently validated in [External Validation](meta-analysis-validation.md).

---

## Summary Table

| # | Case Study | Framework Version | Key Metric | Result |
|---|-----------|-------------------|------------|--------|
| 1 | [The Pilot — Onboarding](01-onboarding-pilot.md) | v2.0 | Baseline velocity | 15.2 min/CU (established the benchmark) |
| 2 | [6 Refactors, 6.5x Speedup](02-framework-evolution.md) | v2.0 - v4.1 | End-to-end improvement | 6.5h down to 1h across 6 identical-scope refactors |
| 3 | [4 Features in 54 Minutes](03-parallel-stress-test.md) | v5.1 | Parallel throughput | 12.4x throughput vs baseline, 0 failures across 35 tests |
| 4 | [Stopped Estimating, Started Measuring](04-measurement-v6.md) | v6.0 | Measurement precision | 7 of 9 metrics moved from estimated to deterministic |
| 5 | [Hardware-Aware Dispatch](05-hadf-v6.1.md) | v6.0+ | Infrastructure shipping | 17 chip profiles + 7 cloud signatures in 120 min |
| 6 | [185 Findings Full-System Audit](06-full-system-audit.md) | v6.1 | Audit coverage | 185 findings, 12 critical, honest self-referential bias report |
| 7 | [The Fastest Feature (+86%)](07-auth-flow-velocity.md) | v5.1 | Peak velocity | 2.1 min/CU — best single-feature result in dataset |
| 8 | [First Feature Under New Architecture](08-ai-engine-architecture.md) | v5.1 | Pattern reuse | 45% cache hit rate, 66% faster than baseline |

---

## The Complexity Unit (CU) Model

Raw metrics like wall time and file count are meaningless without normalization. A 22-task UI refactor with auth integration is fundamentally different from a 4-task backend enhancement.

**Formula:** `CU = Tasks x Work_Type_Weight x (1 + sum(Complexity_Factors))`

- **Work type weights** range from 0.3 (chore) to 1.0 (full-lifecycle feature)
- **Complexity factors** add weight for UI views (+0.15 to +0.45), auth integration (+0.5), runtime testing (+0.4), new models (+0.1 to +0.3), cross-feature scope (+0.2), and design iterations (+0.10 to +0.25 per round)
- **Primary metric:** minutes per CU (lower is better) -- enables direct comparison across features of any size

The model fits a power law with R-squared = 0.87, suggesting the improvement curve has not yet plateaued. Full methodology: [Normalization Model](normalization-model.md).

---

## Supporting Analysis

| Document | What It Covers |
|----------|---------------|
| [Normalization Model](normalization-model.md) | The CU complexity formula, factor definitions, and retroactive normalization of all 16 features |
| [ROI Retrospective](meta-analysis.md) | What-if analysis: had we instrumented measurement from day one, what would we have learned earlier? |
| [External Validation](meta-analysis-validation.md) | Independent review confirming arithmetic consistency, identifying measurement gaps, and recommending next steps |

---

## How to Read These

**If you have 2 minutes:** Read this page. The summary table tells the whole story.

**If you have 10 minutes:** Read [6 Refactors, 6.5x Speedup](02-framework-evolution.md) -- it contains the core thesis with the most data.

**If you have 30 minutes:** Read the full sequence 01 through 08 in order. Each builds on the previous, and the progression from "manual and slow" to "parallel and measured" is the point.

**If you are evaluating methodology:** Start with the [Normalization Model](normalization-model.md) and [External Validation](meta-analysis-validation.md). They contain the statistical caveats and limitations that make the rest of the data trustworthy.
