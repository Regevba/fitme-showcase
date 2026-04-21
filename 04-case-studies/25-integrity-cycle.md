# The Drift That Detected Itself

> A single deep audit uncovered seven features whose state files had been silently wrong for days — median four, worst eleven. The fix for those seven was a one-time batch. The fix for *the pattern* became version 6.2: a 72-hour recurring audit that checks every feature's self-report against reality, commits a snapshot ledger to git, and auto-opens an issue on any regression.

## Context

A PM framework worth the name treats its own state files as source of truth. Features carry their phase (research / prd / tasks / implementation / testing / review / merge / documentation / complete), their task list, their case-study linkage, their shipping evidence — all in a `state.json` that downstream skills and dashboards read to decide what to do next.

The silent failure mode: what happens when the code ships but nobody updates the state file? Nothing — until someone asks. By the time an audit catches the drift, the pile has grown.

The 2026-04-20 open-items audit caught seven features in that exact state. Some had drifted for eleven days. The batch reconcile for those seven was a few hours of work. The question the audit left open was how to not get back into this position next month.

v6.2 is the answer.

## What It Does

Every 72 hours, a GitHub Actions workflow runs an audit script against every feature's state file. The script looks for eight kinds of drift:

| Code | Trigger |
|---|---|
| `PHASE_LIE` | Top-level phase says `complete` but sub-phases say `pending` |
| `TASK_LIE` | Top-level says terminal but tasks are still `pending` / `in_progress` |
| `NO_CS_LINK` | Terminal phase but no case-study linkage |
| `V2_FILE_MISSING` | State declares a file path that doesn't exist on disk |
| `PARTIAL_SHIP_TERMINAL` | `partial_ship: true` alongside a terminal phase |
| `NO_STATE` / `INVALID_JSON` / `NO_PHASE` | Structural failures |

For each cycle: produce a snapshot JSON, diff vs the previous snapshot, commit the new snapshot to main. On any regression — a feature or case study disappearing, or a new finding being introduced — auto-open a labeled GitHub issue.

## Why 72 Hours, Not 24 or Weekly

Every cadence choice in a recurring audit has a cost shape. Too fast: the ledger fills with identical snapshots that have nothing to detect. Too slow: drift accumulates past the point where incremental fix is easy.

The 2026-04-20 data set the cadence:

- **Median drift before detection: 4 days.** A 72h cycle catches the median case one cycle after it happens.
- **Max drift: 11 days.** A weekly cycle would still let that case sit for 4 days past the first cycle. 72h means at most 5 days before detection even for the worst case.
- **Accumulation rate: ~1 drift event per 2 days.** A 72h cycle flags ~1-2 drifted features per run, not a pile of 7.

72 hours is the cadence where all three curves stabilize.

## The Cycle Is a Smoke Detector, Not a Fire Preventer

The integrity cycle's scope is deliberately narrow: detect drift between what state.json claims and what the rest of the repo actually is. It does not prevent drift — Phase 0 (the PM workflow's health check at feature kickoff) still owns that. It does not verify the correctness of the features themselves — a state.json can describe a broken product with perfect internal consistency.

What the cycle does that nothing else in the framework did: run on a 72-hour wall-clock, every time, whether or not any feature was dispatched.

That's the version-bump. Every prior capability in the framework ran when a feature was dispatched — skill-on-demand loading, cache compression, batch dispatch, hardware-aware routing. The integrity cycle runs because 72 hours elapsed. It's the first self-observation loop whose trigger isn't a feature action.

## Bypass Markers — the Signal-to-Noise Dial

Without two bypass markers, the initial baseline would be ~15 findings — half of them false positives from legacy vocabulary in pre-rule features or from consolidated features that deliberately don't have dedicated case studies. That would make every cycle's report indistinguishable from the last one.

The markers are:

- **Pre-PM-workflow backfill** — for features that shipped before PM workflow enforcement existed. Their sub-phase statuses use legacy vocabulary (`pre-pm-workflow`, `backfilled`, `shipped`) which isn't in the current schema and isn't a lie. Ten features carry this marker.
- **Roundup** — for features covered by a consolidation case study where sub-phase granularity isn't meaningful. Six features carry this marker.

With them, the 2026-04-20 baseline is **zero findings** — and any future finding is genuinely new drift worth paging on. The markers are the signal-to-noise dial. Adding a new exemption is explicit and defaults to "no"; the bar is "here's why this class of finding is legitimately unflaggable."

## Regression Gates

Three events trigger an auto-opened issue labeled `integrity-cycle` + `regression`:

1. A feature present in the previous snapshot is absent now — someone deleted a feature directory
2. A case study present before is absent now
3. A new `(feature, finding_code)` pair appears that wasn't in the previous snapshot — drift re-emerged, or a new feature shipped without linkage

Non-regression changes do not trigger an issue: new features, new case studies, state.json content changes that remain consistent, findings that got resolved. The asymmetry is deliberate. The cycle watches the *introduction rate* of drift, not the current drift backlog — because the backlog is fixed once per audit cycle, while introduction is a rate that needs continuous attention.

## Why Commit Snapshots to Main, Not to Artifacts

Two ways to store snapshots: (a) commit them to main as ordinary files, (b) upload them as GitHub Actions artifacts. The framework picks (a).

Reasons:
- Artifacts expire. Git history doesn't.
- Anyone can read `cat .claude/integrity/snapshots/2026-04-21T04-00-00Z.json` without the GitHub API.
- `git log` on the snapshots directory gives the cycle history with commit dates attached.
- The integrity cycle's own committed state is itself integrity-checkable by future cycles.

Cost: ~50KB per snapshot × ~10/month = ~6MB/year. Negligible in a repo with this much case-study markdown.

## Key Takeaways

- **A one-time audit that finds a systemic drift pattern should convert to an automated cycle, not stay as a one-off.** Without the cycle, the 2026-04-20 findings would recur under new feature names in the next audit window.
- **Cadence follows data, not intuition.** The 72h rhythm was chosen because three independent drift metrics (median, max, accumulation rate) converged on it. Other projects would find a different cadence from their own data.
- **Framework-level regressions should auto-open issues, not fail CI.** CI gates merges; integrity cycles gate steady state. Different authority loops — a state.json drift isn't a merge blocker, it's a hygiene event.
- **Version bumps should require a capability change, not a point improvement.** v6.2 adds a new class of behavior (recurring self-observation) whose trigger isn't a feature action. That's the bar — "is this a new kind of thing?" — not "is this a better version of an existing thing?"
- **The cycle found itself first.** The audit that motivated v6.2 was the same kind of artifact v6.2 now produces every 72 hours. Every cycle after the first one reads the previous cycle's snapshot as part of its diff — the detector is its own input. The framework has started observing itself.
