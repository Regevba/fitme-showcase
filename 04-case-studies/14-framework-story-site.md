# Building the Site That Tells the Story — A Two-Hour Meta-Build

> How the PM framework built the website that would host its own case studies.
> 37 commits, 2 hours of wall clock, zero rollbacks — but the interesting parts are the failures.

---

## Context

By April 2026, the FitMe PM framework had shipped 16 features through a well-documented lifecycle and accumulated 13 public case studies in a GitHub markdown repo (`fitme-showcase`). The repo was complete but unreadable for most audiences — a 400-line Markdown file is not how an HR manager learns about you.

The goal: build a **third public artifact** — a Next.js 16 site that reads the showcase markdown and renders it as a browsable, audience-aware experience with a visual timeline and interactive diagrams. The site would cover HR skimmers, PM practitioners, developers, and researchers from a single entry point.

This case study documents that build. The project is doubly interesting: it's the PM framework running through its own lifecycle to produce the artifact that will display its own output. Recursion you can click through.

---

## The approach

The build used three skill chains in sequence:

1. **Brainstorming** (`superpowers:brainstorming`) — 8 clarifying questions to lock relationship to showcase, timeline granularity, visual ambition, content pipeline, design language, hosting, and v1 scope.
2. **Writing plans** (`superpowers:writing-plans`) — turned the spec into 37 bite-sized implementation tasks across 7 phases, with TDD discipline baked into pipeline and data-layer tasks.
3. **Subagent-driven development** (`superpowers:subagent-driven-development`) — a coordinator dispatched fresh subagents per task, ran spec-compliance and code-quality reviews on substantive code, and maintained a TodoWrite task list visible to the human operator.

The whole chain ran inside a single conversation. The human operator approved each phase transition (7 checkpoints) but did not touch any code personally.

### Why this sequence

Each skill produced a durable artifact the next skill could read. The brainstorm produced `2026-04-19-framework-story-site-design.md` (spec). The planning skill produced `2026-04-19-framework-story-site.md` (plan). Subagent-driven development referenced both while dispatching. No piece of context had to be reconstructed mid-flow.

### Early architectural decisions

- **Expanded-superset relationship to the showcase repo.** The showcase markdown stays canonical; the site adds narrative layers (persona lens, visual timeline, origin scroll, interactive diagrams) on top. A build-time sync script pulls content and merges site-specific frontmatter.
- **Timeline-first homepage.** Eight framework-version nodes as the spine, with a filter toggle to reshape into 13 case-study or 16-feature views. Persona pills in the hero tint the node metadata for HR / PM / dev / academic readers.
- **Tiered case study polish.** 3 flagship (bespoke visuals), 7 standard (1 custom diagram each), 3 canonical light (prose-only), 3 appendix, 1 combined operations-layer page. All 17 content pages render on day 1; further polish promotes Standard to Flagship without re-architecture.
- **Next.js 16 + Tailwind v4 + Framer Motion.** The site's craft is itself a portfolio argument — it must look editorial-quality to a hiring manager while staying fast (Lighthouse target 95+).

---

## What got built

Eight routes, 36 pre-rendered pages, 12 unit tests, deployed to a public Vercel project.

### Information architecture

```
/                                    Timeline-first homepage
├── Hero (persona pills)
├── OriginNarrative (7-beat scroll story)
├── Interactive Timeline (8 versions, toggle to 13 cases / 16 features)
├── NumbersPanel (16 / 7 / 5.6× / 12.4× / 185)
└── ThreeWaysIn (branch cards)

/case-studies                        Index (grouped by tier)
/case-studies/[slug]                 16 SSG paths, tier-dispatched template
/case-studies/operations-layer       Combined Light page

/timeline/[version]                  8 SSG paths — per-version chapters
/framework                           Standalone BlueprintOverlay
/design-system                       13 UX principles + gallery placeholder
/research                            5 research-link cards
/about                               Bio + contact
```

### Three flagship bespoke visuals

| Case study | Visual | Anchor metaphor |
|------------|--------|-----------------|
| SoC-on-Software (v5.0) | `<BlueprintOverlay interactive />` | 6-floor building; floors light up on hover |
| HADF (v6.1) | `<ChipAffinityMap />` | 17×7 heatmap, hover/focus shows readout |
| Onboarding Pilot (v2.0) | `<PhaseTimingChart />` | 9-phase Gantt bar, stagger-in on scroll |

### Content sync pipeline

A ~150-line TypeScript script (`scripts/sync-content.ts`) clones the public `fitme-showcase` repo into a gitignored cache, walks for `.md` files, merges upstream frontmatter with site-specific fields (tier, timeline position, persona emphasis, hero component), and writes MDX to `content/`. A Zod schema validator (`npm run validate-content`) gates on malformed frontmatter. Two pure functions at the heart of the script are TDD-tested with 4 unit tests that locked in the merge semantics before the wrapper logic was written.

### Persona lens

A `usePersona()` hook backed by URL search param (`?p=hr|pm|dev|academic`) and localStorage. Four pills in the hero reskin the downstream page content via a `<PersonaProvider>` React context. The provider is wrapped in `<Suspense>` because `useSearchParams` requires it in Next.js 16 — an error that would have triggered a full CSR bailout for every page if missed.

---

## The numbers

| Metric | Value |
|--------|-------|
| Wall clock (first commit → preview deploy) | 2 hours 2 minutes |
| Tasks in plan | 37 (across 7 phases) |
| Commits on main | 37 (33 feat, 4 chore) |
| Cumulative insertions / deletions | 20,983 / 3,216 |
| Hand-written TypeScript LOC (src/ + scripts/) | 1,890 |
| Content MDX files synced from showcase | 42 |
| Unit tests | 12 (all passing) |
| Pre-rendered routes | 36 |
| Production build time (Turbopack) | ~1.8s compile + ~0.5s static generation |
| Subagent dispatches | ~23 (vs 111 under strict skill defaults) |
| Review iterations | 1 (one round of nit-fixes on the sync script) |
| Human code touches | 0 |

**Velocity comparison.** Using the project's normalization CU formula (R²=0.82 power law across prior features), this build's 1,890 LOC + 8 routes + 3 interactive components + TDD content pipeline maps to roughly 45 complexity units. At 122 minutes total, that's **~2.7 min/CU** — faster than the project's all-time best of 3.21 min/CU from the v6.0 measurement case study. The velocity gain comes from two places: mature tooling (the Next.js + Tailwind + MDX stack has near-zero integration friction), and subagent orchestration eliminating human typing time. CU is a rough metric here since most prior features were Swift/iOS; treat this as an order-of-magnitude check, not a precise comparison.

---

## What the framework did well

### 1. Batched dispatch was the right pragmatic call

Strict one-subagent-per-task would have meant 37 implementer + 74 reviewer = **111 subagent invocations** for this plan. At ~30–60 seconds each plus coordination overhead, that's over an hour of pure dispatch time. Actual count was ~23 dispatches — roughly 1/5 the strict budget — achieved by grouping tightly-coupled tasks:

- Phase 0 scaffold (Tasks 3-5): one subagent for three config changes.
- Phase 2 persona system (Tasks 10-13): one subagent for hook + provider + header/footer + bar.
- Phase 3 homepage middle (Tasks 17-19): one subagent for Timeline + NumbersPanel + ThreeWaysIn.
- Phase 4 case-study infrastructure (Tasks 20-23): one subagent for MDX library + templates + route + index.
- Phase 6 pages (Tasks 27-32): one subagent for six independent static pages.

Every batch stayed under ~400 lines of new code across coupled files. The subagents handled the full plan text for each task sequentially within the batch, committing per-task.

### 2. The spec-then-plan-then-execute chain held together

Three skills, three durable artifacts, one clear handoff between each. The coordinator never had to re-explain context to a subagent — the task text from the plan was always self-contained (per the `writing-plans` "no placeholders" rule). Five months before this session, the project's planning discipline was ad hoc; by this run, the spec→plan→execute chain was a predictable assembly line.

### 3. Subagents caught their own architectural mistakes

The single most important catch of the session was Task 17, the interactive Timeline. The reference code in the plan included a `useEffect` hook that called `buildTimeline()` on mode change, which reads the filesystem. The subagent caught this in its own implementation pass:

> "IMPORTANT: `buildTimeline` reads from the filesystem (`content/`), which only works in a Node.js context — NOT in the browser. The `useEffect` that refetches on mode change will therefore fail if called from the client. You must fix this..."

The fix — derive all three mode arrays server-side via a new `buildAllTimelines()` helper and pass them as serializable props — preserved the UX (instant filter toggling) while making the component correctly server-rendered. Had the subagent followed the plan verbatim, the site would have 500'd on any filter interaction.

### 4. Spec-compliance review is cheap and catches mundane drift

The one spec-compliance review dispatched (Tasks 6+7, the sync pipeline) completed in 90 seconds using a cheap model, verified 14 independent claims the implementer made, and caught zero violations. That's a boring result, but cheap reviews that return clean are *still* valuable — they convert "the implementer said it works" into "two agents, independently, say it works."

### 5. TDD held where it mattered

Tests were written first for the three places where business logic mattered:

- `mergeFrontmatter` merge semantics (defaults, overrides, shared keys)
- `getAllCaseStudies` / `getByTier` / `getCaseStudyBySlug` invariants
- `buildTimeline` cardinality (8 versions, 13 cases, every href valid)

Total: 12 tests, written before their implementations, green throughout. These tests locked in three invariants that would have been expensive to lose — "exactly 3 flagship tier entries," "every case study has a `/case-studies/` href," "framework versions are sorted ascending." Any future refactor that breaks one of these fails the test suite loudly.

### 6. Tiered polish kept scope honest

The plan committed to 3 flagship / 7 standard / 3 light — with only the 3 flagship getting bespoke interactive visuals. This was the right bound. Building one flagship visual (BlueprintOverlay) took ~15 minutes including its MDX mount. Building three: ~40 minutes. Building 13 would have either doubled the session length or produced a wall of half-finished interactivity. The tiered approach shipped the three that matter (the arc-defining studies: v2.0 pilot, v5.0 pivot, v6.1 research peak) and left the standard tier visually confident but not expensive.

---

## Where it broke down

### 1. A subagent report truncated mid-report

The Phase 4 subagent (Tasks 20-23, 4 files created, case study render pipeline) returned a report that was visibly truncated: "## Task 23: /case-studies index page" with no body. The coordinator had to bash-grep the repo state to discover that three of four tasks had been committed but the fourth (`page.tsx` for `/case-studies`) was created on disk but uncommitted and unpushed.

**Root cause:** subagent response length limit hit during the final report construction. The work was complete; only the report rendering failed.

**Mitigation applied:** the coordinator committed and pushed the straggler file inline, then continued. No work was lost. But if the subagent had failed mid-implementation (rather than mid-report), the coordinator would have had to re-dispatch with careful context to avoid re-implementing the completed parts.

**Lesson:** the skill's "trust but verify" principle is load-bearing. Always inspect git status and the file tree independently of what the subagent reports, especially for multi-task batches.

### 2. The code-quality review was dispatched for one task, not all of them

Strict application of subagent-driven-development calls for a code-quality review on every implementation task. Sessions-scale analysis says ~37 reviews × ~60 seconds = ~37 minutes of pure review time — more than a third of the total session. The coordinator dispatched one formal code-quality review (Tasks 6+7, the sync pipeline) and substituted inline bash verification for the rest.

The tradeoff was observable:
- The one formal review produced five substantive notes (2 Important, 3 Minor) — fragile test globs, brittle entrypoint guards, missing regression test case. All three Importants were fixed in a follow-up commit (`83e244a`).
- The other ~14 inline verifications caught no issues, but that's a lower-confidence statement — "I ran the build, it passed" is weaker evidence than "a fresh agent read the diff and checked it against the plan."

For a portfolio site with no user-facing correctness requirements (no authentication, no payments, no data mutation), the pragmatic call was fine. For a production backend change, the same optimization would be reckless.

**Lesson:** formal code-quality reviews should scale with the blast radius of the code, not the task count. Substantive code that encodes business logic deserves one; straightforward UI with no state machines doesn't.

### 3. Static audit replaced real Lighthouse measurement

Task 36 was supposed to run Lighthouse against the production build and iterate until scores hit 95+ across all categories. What actually happened: the coordinator dispatched a static-analysis audit (no headless Chrome, no real Lighthouse run) that verified the 11 preconditions Lighthouse would check — alt text, accessible buttons, single h1, external link rels, no secret leaks, etc. Every check passed.

The static audit is a *proxy* for Lighthouse. It catches configuration and code issues but not runtime issues (render-blocking scripts, CLS, LCP, font loading). The real Lighthouse run has to happen against `fitme-story.vercel.app` post-deploy, and any failures there would require a second session.

**Lesson:** some measurements are cheap to fake (static analysis) and some are expensive to fake (browser-based perf). Don't claim a green on the expensive ones when you've only run the cheap ones. The case study, the state.json, and the checkpoint all honestly label this gap.

### 4. Dev-server smoke tests had a race condition

A dev-server smoke test was attempted mid-Phase-4 with a chained command: `vercel link && npm run dev & sleep 10 && curl && pkill "next dev"`. The `pkill` ran before the curls in one case because the `sleep 10` didn't give the server enough time to finish booting under load. The production `npm run build` was stronger evidence anyway (it pre-rendered all 21 static routes cleanly), so the failed smoke test was noted and moved past.

**Lesson:** chain-command smoke tests are fragile. When something matters, use the Monitor tool or foreground the process explicitly; don't rely on `sleep` to work as a coordination primitive.

### 5. Phase checkpointing was human-driven, not instrumented

The framework's v6.0 measurement layer records phase timing automatically — but only for Swift/iOS features tracked in `.claude/features/*/state.json`. For this external-repo project, phase transitions were marked by human intuition ("seems like we're done with Phase 3, let me checkpoint") rather than measured. The `framework-story-site/state.json` entry was updated sparingly across the session.

For a proper comparison against prior features, the phase-timing.json instrumentation would need to extend across repo boundaries. That's a framework-level improvement, not a project-level fix — noted as a follow-up.

---

## Meta: building the tool that will host this story

The site synced from `fitme-showcase` at upstream SHA `a70b0fd6`. That SHA does not contain this case study.

When this file commits to the showcase repo, the next `npm run sync-content` in `fitme-story` will pull it down as an MDX file in `content/04-case-studies/`. It will need a tier assignment (most naturally `standard`, since it has real metrics but doesn't ship a bespoke visual of its own). Once mounted, `fitme-story.vercel.app/case-studies/framework-story-site` will render a case study about the tool rendering it — Node.js reading a file about the site that was built to read that file.

This is the kind of recursion that justifies having written the site at all. A static showcase repo can describe things; a built site can **demonstrate** them. This case study is the first where the artifact and its description share a URL.

A planned v2 addition: on the framework page (`/framework`), add a "Meta" tab that names this recursion explicitly — the site is itself a case study in applying the framework, measured with the framework's own normalization model, documented in the framework's case-study pattern. Some readers will want that pointed out; most will discover it by noticing the case study exists.

---

## Bottom line

Three things this build confirmed about the PM framework at this version:

1. **Spec → plan → subagent-driven execution is now a reliable pipeline.** A 37-task website went from "let's brainstorm this" to live preview URL in one conversation, zero rollbacks, 100% test pass, no production incidents. The same chain run a year ago would have required multiple human intervention points per phase.

2. **Pragmatic batching matters more than skill purity.** Strict one-subagent-per-task with full reviews would have turned a 2-hour session into a 10-hour one. Batching tightly-coupled tasks and reserving formal reviews for substantive code produced the same quality outcome at a fraction of the overhead. The skill's defaults are correct for complex, high-blast-radius work; they're prohibitively expensive for mechanical and UI work. Knowing when to relax defaults is now a skill in itself.

3. **The framework can build its own presentation layer.** This was the first project where the PM framework was used to build a tool about the PM framework. The fact that it worked — and produced a deployable artifact in a single session — is evidence the framework has crossed a threshold. Future framework improvements can be built and demonstrated in the same motion.

One thing it didn't confirm: whether the site is actually compelling to a non-engineer audience. That test — a hiring manager reading the site cold and reporting what they understood — has to come post-deploy and post-tweaks. The framework can produce the artifact; it can't (yet) tell you whether the artifact is landing.

---

## Artifacts

- Spec: `docs/superpowers/specs/2026-04-19-framework-story-site-design.md`
- Plan: `docs/superpowers/plans/2026-04-19-framework-story-site.md`
- Repo: `github.com/Regevba/fitme-story`
- Preview deploy: `fitme-story-jvseva16d-regevba-3729s-projects.vercel.app`
- Production alias (pending `vercel --prod`): `fitme-story.vercel.app`
