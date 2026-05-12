---
type: spec-draft
status: draft
version: 0.3.0-draft
extends: phase/0.2 (gravicity/personal/protocols/phase/README.md)
created: 2026-04-27
updated: 2026-04-27
session: s01-b97ce5
author: Curtis Mercier (drafted by Soma during s01-b97ce5 reflection)
license: CC BY 4.0 (when published)
purpose: "An 'as-implemented' companion to PHASE v0.2 — documents the production-validated lightweight adoption path informed by arzadon-fitness's per-cycle pattern + meetsoma's cycles consolidation. v0.2 designed the destination; v0.3 names the road and the staging."
---

# PHASE v0.3 (Draft) — Lightweight First, Heavyweight Earned

> **Status: DRAFT.** This sits in our session artifacts (not the public protocols repo) until Curtis reviews. v0.2 designed the abstraction; v0.3 is what we learned implementing the lightweight subset and what we'd recommend any other framework do BEFORE attempting the full §4 phase-folder runtime.

## 1. What changed since v0.2

v0.2 specified two mechanisms — lightweight (prompt config) and heavyweight (phase folders). Production experience from two implementations (meetsoma's cycles + arzadon-fitness's numbered work cycles) suggests a **third tier between them**: phase-folder *convention* without phase-folder *runtime*.

The convention captures most of the value (organization, handoff discipline, delegation contracts) at near-zero implementation cost. The runtime (template-chain walking, per-phase brain config) earns its weight only when the convention has been lived in for multiple cycles.

**v0.3 names this middle tier.** Frameworks adopting PHASE should land the convention first, run with it for several cycles, *then* decide if/when to invest in the runtime. Many will discover the runtime is unnecessary for their use case — the convention alone is sufficient.

## 2. The three tiers, ranked by leverage

| Tier | What it is | When | Cost |
|---|---|---|---|
| **T1 Lightweight** (v0.2 §3) | `prompt-config:` frontmatter on a MAP | Quick session-level overrides | Zero — frontmatter convention |
| **T2 Convention** (NEW in v0.3) | Phase folder shape without runtime | Multi-session arcs, delegated work, parallel cycles | Days — pure convention + manual orchestration |
| **T3 Runtime** (v0.2 §4 + §8) | Template-chain runtime, autonomous triggers, state machine | Autonomous multi-phase execution, fully recursive `.soma/`s | Weeks — real implementation work |

T1 and T3 were in v0.2. T2 is the missing rung. It's what most projects actually need.

## 3. The Convention (T2 in detail)

### 3.1 Shape

A phase folder is a directory with this shape (all parts optional except `README.md`):

```
<phase-name>/
├── README.md                ← required: spec + status + TL;DR + contract
├── body/
│   └── soul.md              ← phase-specific identity supplement
├── muscles/                 ← phase-specific muscles
├── protocols/               ← phase-specific protocols
├── scripts/                 ← phase-specific scripts
├── preload-in.md            ← context FROM previous phase
├── preload-out.md           ← context FOR next phase (written at close)
└── report.md                ← deliverable (when phase produces evidence)
```

A folder containing ONLY `README.md` is still a valid phase folder. The shape upgrades as content grows: a flat `<slug>.md` becomes `<slug>/README.md` when a sibling file (preload, report, script) needs to live alongside it.

### 3.2 Frontmatter

The README's frontmatter declares the phase's contract:

```yaml
---
type: phase
slug: <kebab-case>
status: queued | active | shipped | parked | superseded
created: YYYY-MM-DD
updated: YYYY-MM-DD
session_origin: s01-XXXXXX

# Delegation (when applicable)
delegated: true
delegated_model: claude-sonnet-4-6
time_box_minutes: 30

# Worktree isolation (when applicable; abstracts to delegate cap)
worktree: <repo>/.worktrees/<slug>
branch: child/<slug>
sparse_pattern: [src/components/X, tests/X.test.ts]

# Lifecycle
expires-at: shipped+1   # auto-archive after holding ≥1 release past ship
superseded-by: <path>   # forward-link when retired

# PHASE inheritance (T3 only; ignored by T2)
inherits-body-from: ../parent/body/
override-template: false
---
```

### 3.3 Handoff via preload chain

Phase N writes `preload-out.md` at close. Phase N+1's `preload-in.md` is a literal copy or symlink. The handoff is filesystem-mediated, explicit, durable.

This is sufficient even without runtime support: the agent reads `preload-in.md` as part of orientation when it boots into the phase folder.

### 3.4 Delegation through the convention

When a phase is delegated, the worktree fields in frontmatter point at the isolated child branch. **The convention's job:** record what was used (provenance). **The delegate cap's job:** create + manage the worktree (see §5 below).

T2 phases declare their isolation requirements; the delegate tool actuates them. The phase folder doesn't need scripts to manage worktrees — those abstract into the agent's tool surface.

## 4. The Meta-Orchestration Cycle (NEW in v0.3)

A discovery from production: **the parent agent needs its own cycle.** It runs while every other phase runs. v0.3 names this the **meta-orchestration cycle** — the spine that holds the whole tree of phases.

### 4.1 The four moves

```
DO → WATCH → DECIDE → CLOSE → DO
```

- **DO** — pick the highest-leverage thing and ship it
- **WATCH** — check on every active child (`agent.list({active_only:true})`)
- **DECIDE** — for each child: continue / steer / harvest / kill / re-task
- **CLOSE** — when something ships, write the cycle/plan delta, update state

Every loop, rotate. None optional. Don't skip WATCH and DECIDE because DO is fun.

### 4.2 Where it lives

The meta-orchestration cycle is a phase folder at the project's `cycles/meta/orchestration/`. It's `status: living` — it never ships, it evolves. Its `README.md` describes the rhythm, the delegate-vs-do criteria, the child-orchestration playbook, and known gaps.

### 4.3 Relationship to existing protocols

- **Breath Cycle** (v0.2) governs the session lifecycle (inhale → process → exhale). The meta-orchestration cycle governs the *within-process* rhythm — what the agent does between inhale and exhale.
- **AMPS** (v1.0) defines content. The meta cycle is one specific application of AMPS content (a MAP that's also a living phase folder).
- **PHASE** (this spec) makes the meta cycle's shape consistent with every other cycle.

## 5. Delegation Tooling (NEW in v0.3)

Production experience: **per-cycle worktrees should be a property of the DELEGATION cap, not the cycle.**

In the v0.2 spec, the phase folder owned the worktree config. Production showed this leaks complexity into every cycle that delegates. The cleaner abstraction:

### 5.1 The delegate cap learns worktree

```
agent.delegate({
  task: "...",
  role: "auditor",
  model: "claude-sonnet-4-6",
  background: true,
  worktree: true,                    // create isolated worktree
  slug: "01-housekeeping-sweep",     // becomes child/<slug> branch
  sparse_pattern: [...],             // optional: sparse-checkout
  base: "main"                       // base branch (default: HEAD)
})
```

The cap creates `<repo>/.worktrees/<slug>/` on `child/<slug>` branch, applies sparse patterns if specified, spawns the child process inside the worktree. The phase folder's frontmatter records what was used (provenance). The cap orchestrates the worktree.

### 5.2 The merge cap closes the loop

```
agent.merge_worktree({
  slug: "01-housekeeping-sweep",
  mode: "review" | "fast-forward" | "abort"
})
```

- `review` — print `git diff main..child/<slug>`, wait for parent decision
- `fast-forward` — ff-merge child branch to main, push, remove worktree + branch
- `abort` — discard child branch, remove worktree

Status flows back to the cycle: when merged, the phase folder's `status: shipped`. When aborted, `status: parked`.

### 5.3 Why this matters

**Before:** every cycle that delegates needs its own worktree script + boilerplate. The cycle dir grew complexity.

**After:** the cycle declares isolation requirements; the cap actuates. One implementation, infinite uses.

The same abstraction extends to non-cycle delegations (one-off audits, ad-hoc parallel work). Worktree isolation isn't a cycle property — it's a delegation property.

## 6. Adoption path (RECOMMENDED in v0.3)

For frameworks adopting PHASE, the order matters:

### 6.1 Land T2 conventions first (1-2 weeks of disciplined use)

1. Standardize the phase folder shape across cycles, plans, and work units
2. Establish preload-in/preload-out as the handoff convention
3. Adopt the meta-orchestration cycle as the spine
4. Write 5-10 phases in this shape; observe what emerges

### 6.2 Add delegation tooling (1-2 days)

1. Extend delegate cap with `worktree:`, `slug:`, `sparse_pattern:` args
2. Add merge cap for the close-the-loop flow
3. Smoke-test against a real delegated audit phase

### 6.3 Earn T3 runtime when convention pressure demands it

Signals that T3 is worth the cost:
- Phases routinely need different muscles than the parent's hot set
- Phases need different identities than the project's
- Multi-phase autonomous execution is a real use case (cron, scheduled audits)
- The pain of manually loading phase-folder content per session is consistent

If those signals don't fire, T3 may be over-engineering for your use case. T2 is enough.

## 7. What stays unchanged from v0.2

Everything else. v0.3 doesn't break v0.2 — it stages the adoption.

- T1 prompt-config (v0.2 §3) is unchanged.
- T3 phase-folder runtime (v0.2 §4) is unchanged.
- Phase types as SEEDS (v0.2 §5) is unchanged.
- Cascading evolution (v0.2 §7) is unchanged.
- Anti-patterns (v0.2 §11) is unchanged + extended.

v0.3 adds T2 (the convention tier) and §5 (delegation tooling abstraction). Everything else is refinement to point at the new tier.

## 8. Anti-patterns added in v0.3

- **Skipping T2 and going straight to T3.** Implementing the runtime before living in the convention means you'll build for assumptions that the convention would have falsified. Earn the runtime.
- **Hard-coding worktrees in cycle dirs.** Worktree isolation is a delegation concern, not a cycle concern. Extract it to the delegate cap.
- **Writing ad-hoc work plans without phase-folder shape.** Even a tiny one-line plan goes in `<slug>/README.md`, not `<slug>.md`. The folder is the upgrade path.
- **Phase folder without README.md.** A phase without a contract is a phase that won't be honored. README.md is required.
- **Implicit handoff.** Hoping the next phase reads the previous one's commits or session log is wishful thinking. preload-out.md is the contract; its absence breaks the chain.

## 9. Migration from v0.2

If you've implemented v0.2 in production, v0.3 doesn't require changes — it formalizes what you may already be doing.

If you're considering v0.2, v0.3 says: start with T2 (the convention). T1 and T3 are still valid; T2 just becomes the recommended default for multi-session arcs.

---

## Changelog

**v0.3.0-draft** (2026-04-27)
- Added T2 (Convention tier) between T1 and T3
- Added §4 Meta-Orchestration Cycle (the parent's spine)
- Added §5 Delegation Tooling abstraction (worktree as a property of the cap, not the cycle)
- Added §6 Adoption path (recommend T2 → T3 staging)
- Extended §11 (now §8) with v0.3-specific anti-patterns
- Drafted at meetsoma s01-b97ce5 after the cycles consolidation + housekeeping pass + arzadon-fitness study; not yet committed to `gravicity/personal/protocols/phase/`

**v0.2.0** (2026-03-28) — see `gravicity/personal/protocols/phase/README.md`

**v0.1.0** (2026-03-16) — original spec
