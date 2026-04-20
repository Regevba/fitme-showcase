# The Feature That Shipped in Twelve Hours and Never Got Its Own PR

> Six reminder types, a reusable guest-lock overlay, a frequency-cap engine, and a full PRD — designed, specified, and implemented during a parallel stress test that advanced four features simultaneously. The feature landed cleanly and forgot to write its own story. This is that story, reconstructed from primary evidence four days later.

## Context

Smart Reminders was the app's first proactive surface. Until then, FitMe was reactive — it had no way to reach the user until the user opened it. Competitor analysis across Whoop, Oura, MyFitnessPal, and Noom pointed at a single design choice: the apps that succeed at notifications are **state-aware**, not schedule-based. Whoop and Oura fire on biometric thresholds. MyFitnessPal fires on clock time and drives disable rates. FitMe chose the Whoop/Oura model.

The feature ran through the full PM lifecycle — Research, PRD, Tasks, UX, Implementation — in twelve hours, inside the v5.1 Adaptive Batch stress test that was simultaneously advancing three other features. The state file shows six clean transitions from init to complete. No dedicated feature branch. No single merged PR. The work arrived through commits labelled by stress-test phase, and the build stayed green the whole way.

---

## What Shipped

| Field | Value |
|--------|--------|
| Reminder types | 6 (HealthKit Connect, Account Registration, Nutrition Gap, Training Day, Rest Day, Engagement) |
| Lifecycle span | Research → Complete in ~12 hours (2026-04-15 → 2026-04-16) |
| PRD | 349 lines, 20 requirements across P0 / P1 / P2, five-column metrics table |
| Tasks | 14 planned, critical path of 7 |
| Production files | 4 — enum, scheduler, trigger evaluator, locked-feature overlay |
| Tests | 17 cases across 2 files, added 2 days later during audit sprint J |
| Frequency guards | 5 (quiet hours, global daily cap, per-type daily cap, per-type lifetime cap, 4-hour minimum interval) |
| Framework version | v5.1 (Adaptive Batch) |
| Concurrent features | 3 (Smart Reminders was 1 of 4 advanced simultaneously) |

---

## The Engine Behind the Notifications

The scheduler is a singleton with one public entry point — `scheduleIfAllowed(type:body:)` — and five private guards that run in order before any notification reaches the system. The first guard checks the hour: 10 PM to 7 AM is quiet by policy. The second reads today's date-scoped send count and rejects if the global daily cap has been hit. The third reads a per-type daily cap (every type is capped at 1/day). The fourth reads a per-type lifetime counter — three of the six types will stop forever after three fires. The fifth enforces a 4-hour minimum interval between any two reminders of any type.

Each guard is a single guard-statement in the shipped code. The scheduler doesn't surface which guard rejected — it just silently no-ops. Notifications are best-effort: if the system won't add the request, the user sees nothing and the app moves on. The analytics layer records the `reminder_suppressed` event with a reason code so the PM can still tell why a reminder didn't fire.

The design choice that made this possible was splitting "training day" and "rest day" into two separate enum cases rather than two branches of one case. The scheduler treats each as its own lifetime-capped series. The enum's frequency-cap table becomes flat — three types at lifetime 3, three at lifetime unlimited — and the test for that table is a single non-nested loop.

---

## The Overlay That Wasn't Really About Reminders

The PRD specified a Locked Feature Overlay — the card that would appear when a guest user tapped an account-gated surface like AI coaching or cross-device sync. On paper it was a Smart Reminders sub-component: a piece of guest-conversion UX bundled with the Account Registration reminder type.

The shipped file isn't coupled to reminders at all. `featureIcon`, `featureTitle`, `benefitText`, `onCreateAccount`, `onDismiss` — five inputs, zero assumptions about why the feature is locked. The overlay uses design-system tokens throughout (`AppColor.Overlay.scrim`, `AppColor.Accent.primary`, `AppRadius.card`, `AppSize.ctaHeight`), has a labelled accessibility container, and is ready for any future monetization or entitlement gate to mount it.

This is the kind of outcome the Smart Reminders PRD surfaced almost by accident: a reusable component that happened to get specified inside a reminder-focused document but has no structural dependency on the feature that commissioned it. The right place to write the overlay's behaviour was the Smart Reminders PRD — the right place for the overlay file to live was `Views/Shared/`.

---

## The Test Coverage That Documented Its Own Gap

Two days after the feature closed, an audit sprint added 17 test cases. The enum tests are trivial — one per contract: six cases exist, every title is non-empty, every deep link starts with `fitme://`, lifetime caps are exactly 3 for the three attention-bounded types and nil for the other three, raw values are snake_case, Codable round-trips preserve identity.

The scheduler tests are more interesting because the scheduler is mostly untestable without a mock for the iOS notification centre. The test author wrote the testable surface (singleton identity, cancellation idempotence, UserDefaults key contract, quiet-hours math) and then wrote a comment inside the test file documenting what couldn't be tested and what mock infrastructure would unlock it.

This is the right pattern. The honest note — "The full scheduling path goes through UNUserNotificationCenter, which isn't reliably testable without a mock. Where the production code's effect is observable through state/defaults, we test it directly; where it isn't, we document the gap rather than fake it" — turned an uncovered code path into a queued backlog item instead of a hidden liability. The gap is now visible in the test file itself, not buried in a separate coverage report.

---

## Why the Case Study Was Late

The project rule that every shipped feature gets a case study was introduced two days before Smart Reminders started. The feature transitioned to complete on schedule, the build stayed green, the state file closed cleanly — and then no case study was written, because the work had never produced a dedicated PR to harvest evidence from.

Everything landed through stress-test phase commits. Four features were moving simultaneously, and the commit log grouped them under phase banners (`v5.1 stress test phase 6 — UI views + orchestrator + triggers, BUILD SUCCEEDED`) rather than per-feature PR threads. The velocity was real; the case-study scaffolding wasn't.

Four days later the gap surfaced during an audit sweep. The reconstruction was straightforward — the state file, PRD, UX spec, commit graph, and source files all survived — but the lesson is a framework lesson, not a feature lesson: stress-test commits should keep their per-feature prefixes even when they ship under a phase banner. The two mechanisms compose cleanly. Without the per-feature prefix, reconstruction becomes a grep exercise rather than a PR walk.

---

## Key Takeaways

- **State-aware beats schedule-based.** Competitor analysis pointed at a single design choice and the PRD committed to it. Every shipped guard is state-aware (protein < target, readiness < 40, days since last open, HealthKit authorized). The clock appears only as a quiet-hours boundary and a 4-hour minimum interval, not as a trigger.
- **Flat enums are better than nested conditionals.** Splitting training and rest into two cases turned a frequency-cap table into a loop and made the test a one-liner per contract. The six-case enum is the foundation the scheduler's five guards all read from.
- **Spec a component where it's needed; ship it where it belongs.** The Locked Feature Overlay lived in the Smart Reminders PRD and shipped in the shared views folder. The PRD described the contract; the file placement was a file placement.
- **Document the coverage gap inside the test file.** A honest comment about what isn't testable without mock infrastructure is more valuable than fake green tests. It turns uncovered paths into queued work rather than hidden debt.
- **Stress-test velocity has a PM hygiene cost.** Four features advanced simultaneously in twelve hours. The price was four phases (`testing`, `review`, `merge`, `documentation`) that stayed marked pending in the state file even though the work landed, and a case study that didn't get written until a four-day-later audit noticed the gap.
