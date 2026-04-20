# The Refactor Where Two Project-Wide Rules Were Born

> The second screen through a UX Foundations alignment pass — and the feature where an audit open question became a project-wide analytics convention, and a one-off refactor recipe became a codified v2/v1 coexistence rule.

## Context

Home Today Screen was the second screen refactored, after Onboarding. Unlike Onboarding — which patched v1 in place — Home was the first feature to apply what the team later called "the V2 Rule": a `v2/` subdirectory sitting next to the v1 file, same Swift type name on both sides, and a single project file commit to flip the build from v1 to v2 while keeping v1 as a historical reference. The mechanics worked, so the recipe became a rule.

Home also produced a second durable artifact by accident. Audit open question #9 asked how Home's new analytics events should be named. The answer — prefix the screen name — was written into CLAUDE.md as a project-wide rule three days later. Every analytics event on every screen has followed it since.

---

## The Refactor

| Field | Value |
|--------|--------|
| v1 file size | 1,029 lines |
| v2 container | ~703 lines |
| Audit findings | 27 (8 P0 / 14 P1 / 4 P2 / 1 positive) |
| Tasks | 17 across 4 layers |
| Analytics events | 3 shipped + 1 deferred |
| Tests | 21 (11 behavior + 10 analytics) |
| Design tokens added | 4 |
| Components promoted to the design system | 2 |
| Sub-features spawned | 3 |
| Framework version | v3.0 |

The three sub-features spawned — Status+Goal merged card, Metric Tile Deep Linking, Onboarding v2 retroactive — each became their own PM cycle with their own state file, PRD, and tests. Home shipped clean; deferred work did not stall the pilot.

---

## Rule 1: The V2 Rule

Onboarding's refactor had shipped the spirit of a v2 pass but not the mechanics — it patched v1 directly, leaving no historical artifact and requiring a full re-read to understand what had changed. Home was the first feature where the refactor process was deliberately designed to be legible after the fact.

**The V2 Rule has four components:**

1. A `v2/` subdirectory next to the v1 file, with the same filename inside it.
2. The same Swift type name in v1 and v2 — so every parent call-site resolves to the same symbol and requires zero churn.
3. A single project-file commit that adds v2 to the Sources build phase and removes v1 from it, while retaining v1 as a file reference. v1 survives in git history and in the file navigator but is no longer compiled.
4. A header comment on v1 marking it `HISTORICAL —` with a reference to the audit that triggered its replacement.

The rule was validated across five subsequent screens (Training, Nutrition, Stats, Settings, and eventually a retroactive Onboarding pass) and now lives in the project's root configuration. No screen has required a `v3/` directory — because a second alignment pass on an already-v2 screen is a patch, not a rewrite.

---

## Rule 2: Screen-Prefixed Analytics

During the audit, open question #9 was technical and seemingly minor: should a tap on a Home CTA fire `tap_start_workout` or `home_action_tap`? The user's rationale for picking the prefixed form was operational: in GA4, with dozens of events across the app, the originating screen should be obvious without opening source code. Funnel analysis, regression isolation, and per-screen metric drill-downs all become trivially navigable if the event name itself carries the screen context.

Three days later the answer was promoted to a project-wide rule:

- Every event tied to a specific screen MUST begin with the screen name prefix (`home_`, `training_`, `nutrition_`, `stats_`, `settings_`, `onboarding_`, `auth_`).
- Cross-screen lifecycle events (`app_open`, `session_start`, `sign_in`) stay unprefixed — they're global, not screen-scoped.
- GA4 recommended events (`tutorial_begin`, `select_content`, `login`) keep their dictated names for dashboard compatibility.

The analytics skill now refuses to write a spec that contains a non-compliant screen-scoped event. A retroactive migration plan backfills aliases in GA4 so historical dashboards keep working.

---

## How the Audit Became the PRD

Home's audit produced 27 findings and a 22-item Decisions Log. Every audit item that required a product judgment — copy direction, component choice, defer-or-ship — was lifted into an open question and given back to the user. Each answered question became a PRD line. This turned what would otherwise be ambiguous implementation choices into a documented paper trail.

The effect is visible in the PRD output: 22 of 27 findings scoped in, 5 spun out cleanly as sub-features, and zero "we'll figure it out in implementation" placeholders.

---

## Primary Metric and Kill Criteria

Home's primary metric is `sessions_per_day` — the proxy for "users come back" — rather than a Home-specific funnel. Home is the entry point to every other flow, so its success metric is the global return metric.

The kill criterion is deliberately narrow: *"Core screen — always active. v2 alignment ships only if no readiness or biometric flow regresses."* A screen every user sees on every launch cannot be killed on a growth threshold; it can only be rolled back if it breaks something load-bearing.

---

## What v3.0 Did Not Yet Have

The 17-task plan included a foundation layer of 9 independent tasks — tokens, component scaffolds, UX principle walks — that would have been perfect parallel dispatch candidates. They were executed serially. The L1 learning cache and the parallel subagent dispatcher did not land until v4.0, a week later. Training v2 — the next refactor — inherited both, and compressed the same foundation-layer shape from hours to minutes.

Home's wall time was not the framework's headline number. What Home shipped was the *shape* of the per-screen refactor, ready for later framework versions to compress.

---

## Key Takeaways

- **Open questions can become project-wide rules.** The analytics prefix convention was a byproduct of one feature's audit. It is now a standing prompt inside every UX audit: "does any OQ answer generalize beyond this screen?"
- **Spawn sub-features rather than expand scope.** Status+Goal, metric-tile deep-linking, and the Onboarding retroactive refactor each became their own PM cycle. Home shipped with three clean deferrals rather than one bloated scope.
- **Refactor recipes generalize when the mechanics are deliberate.** Onboarding proved the idea of a v2 rewrite. Home proved the mechanics — v2/ subdirectory, same type name, pbxproj swap, historical header — and only then did the recipe become a rule.
- **Audit → Decisions Log → PRD is the right handoff.** Every finding that requires a product call becomes an OQ; every OQ becomes a PRD line. No judgment calls get lost between research and scoping.
- **Kill criteria for always-on screens are regression-only.** A screen that cannot be turned off cannot be killed on a growth threshold — only on a regression on a load-bearing flow downstream.
