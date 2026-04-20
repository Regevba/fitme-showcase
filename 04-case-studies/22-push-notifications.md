# The Feature That Shipped Without a Door

> Push Notifications — the first feature to complete a full v5.2 PM lifecycle end-to-end, and the case study that taught the project the difference between "shipped" and "reachable."

## Context

Push Notifications was the pilot run of the v5.2 PM workflow. Research, PRD, tasks, UX spec, implementation, tests, code review, merge — every phase was executed, every gate passed, every task marked done. `state.json` closed with the note: *"First feature to complete full v5.2 PM lifecycle."*

Three weeks later, an audit found that nobody using the app had ever seen the feature. The priming view — the one surface that would have asked the user for notification permission — had been built, reviewed, tested, and merged, but never wired into navigation. No entry point. No door.

This is the case study of how that happened, and the project-wide lesson it produced.

---

## The Work

| Field | Value |
|---|---|
| Scope | Local notifications only (`UNUserNotificationCenter`) — APNs deferred to Phase 2 |
| Tasks | 12 (T1 → T12) |
| Production code | 400 LOC across 5 files |
| Tests | 18 across 2 test files |
| Analytics events | 10, all `notification_` prefixed |
| Success metrics | 5 defined (opt-in, tap-through, ack rate, disable rate, DAU lift) |
| Code review findings | 11 (2 critical, 4 important, 5 suggestions) |
| Duration | 3.5 calendar days, research through merge |

The substrate that shipped is real. A `NotificationService` singleton wraps `UNUserNotificationCenter`, enforces quiet hours (10 PM → 7 AM), gates per-type preferences, and caps the daily notification count at 2. A `NotificationPreferencesStore` persists four toggles in UserDefaults. A content builder produces three notification types — workout reminder, readiness alert, recovery nudge — each with a deep-link `userInfo` payload. A `DeepLinkHandler` routes `fitme://` URLs to the right tab. All of it works; all of it is tested.

The one thing that does not work is the user ever getting there.

---

## What Went Right

The code review phase caught two real product defects before merge. C1: the frequency cap was specified, instrumented, and stored — but `scheduleNotification` never checked it. A cap that couldn't cap. C2: per-type disable toggles existed — but `scheduleNotification` never read them. A disable that wouldn't disable. Both were fixed in a single follow-up commit (`531b196`) and re-approved the same day.

This is what Phase 6 review is for, and it worked. Two defects that would have reached production were caught and corrected before the state file flipped to `complete`.

The analytics taxonomy was also executed cleanly: 10 events, all prefixed with `notification_`, all consent-gated via `ConsentManager`, all with documented parameters. When the events eventually fire, the dashboards will work without rework.

---

## What Went Wrong

The PRD specified that the priming view would be triggered "after first completed workout (primary) or via Settings toggle (secondary)." Neither trigger was implemented. Task T8 — "wire into app lifecycle (schedule on launch)" — landed the scheduling wiring but not the presentation wiring. The distinction was implicit in the task language and got lost in execution.

Two deferrals compounded. First, the Settings preferences UI was deferred to a later feature (Smart Reminders), on the reasonable assumption that it was secondary. Second, the first-workout trigger was not explicitly listed as a task, on the reasonable assumption that it was obvious. Each deferral looked local. Together, they closed off every path to the view.

Three weeks later, an audit opened finding UI-016: the file now carries a `HISTORICAL — never wired into navigation` header. The scheduling substrate works. The consent surface does not reach the user. Zero of the five PRD success metrics have data to evaluate.

---

## The Rule That Came Out of It

Every future UI-bearing feature splits its wiring work into two separate, independently-verifiable tasks:

1. **Wired into scheduling / logic** — the subsystem can do its job if asked.
2. **Wired into navigation** — at least one live entry point presents the surface from a real user flow.

The test plan must include at least one test that instantiates the new view *from its intended entry point*, not just from a unit test or a preview. A feature that cannot be reached cannot be said to have shipped, no matter how many tasks were marked done.

A second lesson ran in parallel: direct-to-main commit chains are not safe for large features. CLAUDE.md already said this — any feature with > 5 files or new services needs a `feature/{name}` branch and a PR. Push Notifications violated that rule (5 new files, 2 new services, direct commits on main). A PR review would likely have surfaced the reachability gap during the merge discussion, not three weeks later in an audit.

---

## Why This One Matters

Push Notifications is the case study every team has but rarely publishes: the feature that passed every gate and still missed its user. The v5.2 workflow is not being blamed here — it did exactly what it was designed to do, including catching two real defects in Phase 6. But the workflow's gates can only check what they're pointed at. "Is this file in the build target?" ≠ "Is this view presented from anywhere?"

The lesson is durable:

- **Task names hide assumptions.** "Wire into app lifecycle" can mean scheduling, presentation, or both. If it's ambiguous, split it.
- **Deferrals compose.** Two reasonable, local deferrals can close off every path to a feature simultaneously. Always check "is there at least one live entry point left?"
- **Analytics without traffic is silent.** 10 events specified, instrumented, consent-gated — and not one has fired, because the one surface that would trigger them is unreachable.
- **The audit loop is the safety net.** UI-016 is closed now, with a header on the file, a test file salvaged via Sprint F (TEST-013), and this case study as the durable record. The framework caught what the feature cycle missed.

---

## What's Shipped Today

The substrate is ready. When the Smart Reminders feature ships the Settings preferences UI, the priming view (or its successor) gets its first live entry point. When that happens, every line of the 10-event analytics taxonomy will start producing real data, and the 2-week post-launch review that was specified in the PRD — and has never been run — will finally have something to review.

Until then, Push Notifications remains a pilot: the first feature to complete a full v5.2 lifecycle, and the first to demonstrate that a complete lifecycle is not sufficient for a complete feature. Both halves of that sentence matter.
