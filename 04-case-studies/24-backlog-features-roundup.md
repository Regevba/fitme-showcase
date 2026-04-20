# The Six Features That Didn't Get Their Own Case Study

> Six features that shipped either before the "every feature gets a case study" rule landed, or slipped past it — and the decision to consolidate them honestly rather than pad the library with six thin fabrications.

## The Rule and Its Gap

Mid-April, the project adopted a standing rule: every shipped feature gets a case study. The point was accountability — a growing library of honest narratives about why each thing was built and what shape it took. The rule held cleanly for every feature that followed. It had a back-catalog problem.

Six features sat outside its reach. Two of them — the AI cohort intelligence engine and the Android design system investigation — pre-dated the full PM workflow entirely and didn't have the artifacts a normal case study leans on. One — the development dashboard — was an off-platform web app whose interesting shape was a tail of post-merge stabilization rather than a clean lifecycle. Three — the GDPR compliance pass, the Google Analytics foundation, and the Stats v2 UX alignment — had everything a case study needs, they just shipped before the rule existed.

The temptation was to write six case studies anyway. The honest answer was that doing so would fabricate narrative around three features that genuinely don't warrant it, and dilute the library's signal. One consolidated roundup does the accounting without the padding.

## When a Feature Needs a Case Study

This set clarified the threshold. A feature warrants a dedicated case study when three conditions line up: it was shipped under the current workflow, it has enough artifacts to ground the narrative without invention, and it produces at least one lesson that generalizes beyond the feature itself. Two of three is usually not enough. The Android design system investigation has a solid paper trail and a clean skip-pattern for Phases 3-8 — but its lesson ("research-only deliverables are a legitimate work type") generalizes to exactly one scenario, and the deliverable itself is documentation, so a case study would be a doc about a doc.

The AI cohort intelligence engine fails the first condition entirely. Its state file is a single transition backfilled during a cleanup pass, with every phase below implementation marked as pre-PM-workflow. A dedicated case study would require inventing PM decisions that weren't made, or touring the file diff, neither of which serves the library.

The development dashboard is more interesting. It has the fastest end-to-end lifecycle in project history — research to complete in about six hours on a single evening — and a full 10-transition log. But the feature that *shipped* and the feature that became *useful* were different artifacts. The shipped version had a working reconciliation engine, drag-drop kanban, dark mode, and a Vercel deploy. The useful version required a long tail of post-merge fixes — source wiring, shared-layer conflict resolution, schema normalization, client-side tab navigation, null-guards — none of which the PM workflow currently has a formal phase for. The honest narrative isn't "the dashboard shipped" but "the dashboard stabilized over a week of small commits," and that's a workflow observation rather than a feature story.

## GDPR — A Feature That Cannot Be Killed

GDPR compliance is the cleanest pre-rule feature in the roundup. Full 10-transition lifecycle, dense PRD, research and tasks and UX spec all present, tests and analytics covered. It shipped end-to-end in two hours on a single afternoon. Research consumed thirty minutes, PRD fifteen, tasks and UX another thirty, implementation forty-five, review and merge and docs the last fifteen. Eight files, seven hundred new lines, zero high-risk files touched.

What's interesting about it isn't the wall time — it's the shape of the kill criteria. Every other feature in the project has a growth threshold ("below X% adoption after Y months, simplify or retire"). GDPR's kill criteria read "Legal requirement — cannot be killed." That was the first time the project formally acknowledged that some features have floor-only, never-ceiling success conditions. The feature's success metric is binary: does deletion complete, does export return a valid JSON archive, do the five analytics events fire. There is no growth curve to watch.

The feature's architecture reflects that. Account deletion enters a thirty-day grace period rather than executing immediately, and a background job does the destructive work only after the user has had time to cancel. Nine data stores get cleared in sequence — device, Keychain, UserDefaults, CloudKit, three Supabase tables, the AI cohort (anonymized rather than deleted), Firebase. Review flagged a known gap honestly: the nine-step deletion is not atomic. If the seventh step fails, the first six don't roll back. That's a logged risk rather than a discovered bug, and it's on the list of things to address before a production cutover.

The shape of GDPR — two hours, zero ambiguity, one explicit architectural gap, kill criteria that can only be failed, never outgrown — is the template for legally-bounded features. The pattern is legible enough that it should be the reference for future compliance work.

## Google Analytics — The Measurement Substrate

Google Analytics is arguably the most consequential feature in the project's first two weeks, and easily the richest one in this roundup. It went from "eleven shipped features, forty defined metrics, zero analytics instrumentation" to a working GA4 pipeline in about nineteen hours of active work across two calendar days.

The architectural choice was a protocol-based abstraction. Rather than wiring Firebase calls directly throughout the app, the feature introduced an `AnalyticsProvider` protocol, an `AnalyticsService` singleton façade, a `ConsentManager` that handles both ATT and GDPR consent flows, and two adapters — a production Firebase adapter and a mock for tests. Research had evaluated TelemetryDeck, Mixpanel, Amplitude, and PostHog before picking GA4 + Firebase, with the protocol abstraction framed explicitly as "swap providers without code changes." The abstraction isn't over-engineering; it's what made seventeen unit tests possible and what the code review found clean. Every event flows through one typed surface.

The feature also implicitly shaped every PRD that followed. Once GA existed, the PM workflow's Phase 1 gained an analytics spec gate — every new feature now has to spec its events in the PRD or the PRD can't close. The screen-prefix naming convention that became a project-wide rule a week later was implicit in GA's taxonomy before it was codified by Home v2's audit. The mid-sprint GA4 2025 naming refactor (renaming events to match Google's 2025 recommended conventions) is the clearest signal that the analytics taxonomy was still being shaped during the feature itself, not just the tooling.

Shipping a measurement substrate before most of the features that emit events against it is a pattern worth naming. Every feature's kill criteria, funnel metrics, and post-launch review depends on GA existing. Fourteen of forty metrics were instrumented at merge — not forty of forty, by design — the rest get instrumented as the features that emit them land. This is the "integrate measurement first" pattern, and it's the reason the project can make data-driven decisions at all.

## Stats v2 — The Lightest Audit in the v2 Series

Stats v2 is the lightest pass in the per-screen UX Foundations alignment series. The Stats screen came into its audit with perfect scores on every token dimension — fonts, spacing, colors, radii all at 100% — which made the gap it did have especially sharp. Accessibility was at twenty-seven percent. Four elements out of fifteen had labels. The interactive drag-gesture chart at the center of the screen was completely invisible to VoiceOver.

The audit produced nine findings. Two P0: a raw animation that should have been a motion token, and a chart with zero VoiceOver support. Three P1: hardcoded frame values that needed to become an `AppLayout` enum, which itself became a new design system primitive. Four P2: a11y labels on the period picker and ChartCard, documentation on a GeometryReader kept for iOS 16 compat, and nested types extracted to their own files.

The lesson Stats produced is small but sharp: token compliance and UX Foundations compliance are orthogonal. A screen can be the most token-compliant surface in the app and simultaneously the least accessible. The design system governs one axis; the foundations principles govern another. Both are necessary, neither is sufficient.

There's one artifact-integrity footnote worth logging. Stats v2's state file has the feature marked complete at the top level while the per-phase statuses show implementation in progress and all ten tasks pending. The PR shipped the work — the v2 file exists in the build target — but the state file wasn't fully reconciled during merge. That's a concrete example of where the PM workflow's merge checklist could add an automated validation step: the top-level phase should be derived from or validated against the per-phase statuses. Stats v2 is now the reference case for that proposal.

## Three Lightweight Entries, Honestly Acknowledged

The remaining three features — AI cohort intelligence, the Android design system investigation, and the development dashboard — each get a paragraph in the main roundup and no more. That's the right amount. The AI cohort engine shipped before the PM workflow existed and its record is a backfilled single-transition state file. What shipped was a Python FastAPI backend with JWT validation and rate limiting, and a Swift client layer with an orchestrator mediating between on-device inference and the server, but none of that is documented as PM decisions — only as code. Writing a dedicated case study would require either inventing the decisions or touring the code.

The Android design system investigation is a ninety-two-token iOS → Material Design 3 mapping exercise that explicitly didn't ship any app code. Its deliverables are a mapping document and a Style Dictionary Android config. Phases 3 through 8 were skipped with a rationale on each — the PM workflow's first clean example of a research-only deliverable. Worth remembering as a template; not worth a standalone case study.

The development dashboard is the off-platform outlier — an Astro/React/Tailwind web app deployed to Vercel. Its full lifecycle ran in six hours on a single evening, and then about ten post-merge commits over the following week made it actually useful. The case-study-worthy pattern isn't the lifecycle, it's the tail: a feature that passed its PM workflow cleanly but genuinely needed a second, uncounted wave of work to be worth visiting. The workflow currently doesn't have a formal "post-launch stabilization" phase; the dashboard is the best argument for why it probably should.

## What the Roundup Format Buys

Writing one roundup instead of six case studies preserves the signal of the case-study library. A library where every entry is a dense, honest narrative about one feature is more useful than a library where a third of the entries are padded to reach a line count. The roundup captures what's true about the pre-rule features — their existence, their artifacts, their shipped diffs, their kill criteria, their known gaps — and accepts that not every feature produces a project-level lesson.

Three of the six features in this roundup could have supported dedicated case studies. The choice not to split them out was deliberate: pair the three thinner entries with the three richer ones in one honest document, rather than juxtapose six case studies of wildly uneven depth. The richer three — GDPR, Google Analytics, Stats v2 — each get roughly the density they'd have had standalone. The thinner three get what they warrant and no more. The roundup is the accounting; it doesn't pretend to be more.

## Key Takeaways

- **A feature needs a case study only when it was shipped under the current workflow, has enough artifacts to ground the narrative, and produces a lesson that generalizes.** Two of three is often not enough.
- **Pre-rule features should be backfilled honestly, not narratively.** A single-transition state file is the correct record for a pre-workflow feature. Writing a full case study around it would require invention.
- **Research-only deliverables are a legitimate work type.** Skip Phases 3-8 explicitly with a rationale on each, target the deliverable rather than user behavior with the metrics, and stop.
- **The "phase-complete" and "usefully-shipped" states can be different.** The development dashboard is the template: a clean lifecycle followed by a tail of stabilization. The PM workflow could formally name the second wave.
- **Kill criteria shape the narrative.** Floor-only features (GDPR's "cannot be killed") produce stories about compliance and risk; growth-threshold features produce stories about adoption. GA sits in between — "cannot be killed, but simplify consent if opt-in rate is too low."
- **Token compliance and UX Foundations compliance are orthogonal.** Stats is the clearest proof: perfect on every token dimension, worst in the app on accessibility. The design system governs one axis; the foundations principles govern another.
- **The measurement substrate integrates first.** Every subsequent feature's metrics, funnels, and post-launch reviews depend on the analytics layer existing. GA4 is the reason the project can make data-driven decisions at all.
