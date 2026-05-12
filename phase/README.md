---
type: spec
status: draft
version: 0.3.0
created: 2026-03-16
updated: 2026-05-12
author: Curtis Mercier
license: CC BY 4.0
extends: amps/1.1
complements: maps/0.1, breath-cycle/0.2, seams/0.2, seeds/0.2, atlas/0.2, mlx/0.1, mlr/0.1
---

# PHASE — Prompt Handoff for Agent Session Evolution v0.3

> A protocol for reshaping an agent's brain per task and cascading refinements across phases. The plan doesn't just tell the agent what to do — it configures how the agent thinks.

*Extends: [AMPS v1.1](../amps/) (Agent Memory Protocol Stack)*
*Complements: [MAPS v0.1](../maps/), [Breath Cycle v0.2](../breath-cycle/), [SEAMS v0.2](../seams/), [SEEDS v0.2](../seeds/), [ATLAS v0.2](../atlas/), [MLX v0.1](../mlx/), [MLR v0.1](../mlr/)*

*v0.3 absorbs the [v0.3-draft companion](../_archive/phase-v0.3-draft-companion.md) into the canonical spec — adds the T2 Convention tier, the meta-orchestration cycle, and delegation-owned worktrees. v0.2 mechanisms preserved.*

## 1. The Insight

Most agent systems have one configuration: the system prompt. It's assembled at boot from whatever knowledge is "hot" — recently used muscles, recently referenced protocols. This works for general sessions. But when an agent is executing a specific plan, organic heat is the wrong signal.

A refactoring session needs `incremental-refactor` hot — the task requires it, regardless of what happened last session. A blog session needs `voice-hygiene` hot, not `code-navigator`. The task knows what the agent needs. Heat doesn't.

PHASE lets the task configure the agent. The agent boots with a brain shaped for the work. And when that agent finishes, it refines the configuration for the next one.

## 2. Three Tiers

PHASE operates through three tiers, ranked by leverage and cost. Production experience across `arzadon-fitness` and `meetsoma` showed that the original v0.2 two-tier split (Lightweight + Heavyweight) was missing the middle rung where most multi-session work actually lives.

| Tier | What it is | When | Cost |
|---|---|---|---|
| **T1 — Prompt Configuration** (§3) | `prompt-config:` frontmatter on a MAP | Quick session-level overrides | Zero — frontmatter convention |
| **T2 — Phase Folder Convention** (§4) | Phase folder shape without runtime | Multi-session arcs, delegated work, parallel cycles | Days — convention + manual orchestration |
| **T3 — Phase Folder Runtime** (§5) | Template-chain runtime, autonomous triggers, state machine | Autonomous multi-phase execution | Weeks — real implementation work |

**The recommended adoption path** is T1 → T2 → T3 (see §15). Most frameworks adopting PHASE will live in T2 indefinitely; T3 is only worth its cost when the convention has been lived in for multiple cycles and specific signals fire (see §15.3).

### Lightweight: Prompt Configuration (T1)

For ad-hoc tasks, quick overrides, or single-session work. The configuration lives in the MAP's frontmatter. See §3.

### Convention: Phase Folders (T2)

Phase folder shape without runtime support. Captures most of the value (organization, handoff discipline, delegation contracts) at near-zero implementation cost. See §4.

### Heavyweight: Phase Folder Runtime (T3)

For multi-session projects with chained phases and autonomous execution. Each phase is a self-contained agent context — its own body, muscles, protocols, and preloads. The template system resolves the folder as a mini agent boot environment. See §5.

Most tasks start at T1. When a task grows beyond one session, it graduates to T2 (the convention). T3 is earned when autonomous chaining is a real requirement. All three use the same principle: the task configures the brain, and the completing agent refines the next phase.

## 3. Prompt Configuration (Lightweight)

A PHASE config lives in a MAP's frontmatter (or a standalone config file) under the `prompt-config` key:

```yaml
prompt-config:
  heat:
    protocols:
      quality-standards: 10
      workflow: 8
      voice-hygiene: 0
    muscles:
      incremental-refactor: 10
      code-navigator: 8
      voice-hygiene: 0

  force-include:
    muscles: [incremental-refactor, code-navigator]
    protocols: [quality-standards]

  force-exclude:
    muscles: [voice-hygiene, astro-islands, physics-ui]
    protocols: [content-triage]

  sections:
    includeDocs: true
    includeSkills: false

  budgets:
    muscles.tokenBudget: 3000
    muscles.maxFull: 4

  identity: |
    This session focuses on runtime architecture.
    Think like a systems architect.
    Evaluate trade-offs explicitly.
```

### 3.1 Heat Overrides

Temporarily set heat values for the session. Overrides the agent's organic heat (from `state.json` or frontmatter) without modifying the stored values. When the session ends, the override dies. Organic heat is unchanged.

```yaml
heat:
  protocols:
    quality-standards: 10    # force hot (loaded with full body)
    voice-hygiene: 0         # force cold (not loaded)
  muscles:
    incremental-refactor: 10 # force hot
```

### 3.2 Force Include / Exclude

Stronger than heat overrides. Force-include guarantees loading regardless of heat thresholds or budget limits. Force-exclude guarantees omission regardless of heat.

```yaml
force-include:
  muscles: [incremental-refactor]   # always loaded, even if cold
force-exclude:
  muscles: [voice-hygiene]          # never loaded, even if hot
```

Use force-include for muscles the task *requires*. Use force-exclude for muscles that would waste context tokens on irrelevant knowledge.

### 3.3 Section Toggles

Control which sections of the system prompt are assembled. Saves tokens by omitting sections irrelevant to the task.

```yaml
sections:
  includeDocs: false        # skip documentation references
  includeSkills: false      # skip skills block
```

A deep code session doesn't need docs. A writing session doesn't need tool guidelines. The task knows.

### 3.4 Budget Overrides

Adjust token budgets and loading limits for the session.

```yaml
budgets:
  muscles.tokenBudget: 3000   # more muscle context
  muscles.maxFull: 4           # allow more full-body muscles
  protocols.maxFull: 2         # fewer full protocols
```

### 3.5 Supplementary Identity

An additional identity block injected into the agent's identity chain. Doesn't replace — it layers on top of the existing identity hierarchy.

```
Identity chain:
  global → parent → project → PHASE identity (new)
```

This is how the same agent becomes an architect in Phase 0, a builder in Phase 1, and a migrator in Phase 2. The project identity stays constant. The phase identity reshapes the approach.

```yaml
identity: |
  You are executing Phase 0 of the runtime abstraction.
  Think like a systems architect.
  Your output is an interface design, not code.
```

## 4. Phase Folder Convention (T2)

The T2 tier is the **convention** — the phase folder shape, the preload-chain handoff, and the frontmatter contract — without the runtime that walks templates or triggers autonomous execution. This is what most projects need.

### 4.1 Shape

A phase folder is a directory with this shape (all parts optional except `README.md`):

```
<phase-name>/
├── README.md                ← required: spec + status + TL;DR + contract
├── body/
│   └── soul.md              ← phase-specific identity supplement (T3 only)
├── muscles/                 ← phase-specific muscles (T3 only)
├── protocols/               ← phase-specific protocols (T3 only)
├── scripts/                 ← phase-specific scripts
├── preload-in.md            ← context FROM previous phase
├── preload-out.md           ← context FOR next phase (written at close)
└── report.md                ← deliverable (when phase produces evidence)
```

A folder containing ONLY `README.md` is still a valid phase folder. The shape upgrades as content grows: a flat `<slug>.md` becomes `<slug>/README.md` when a sibling file (preload, report, script) needs to live alongside it.

### 4.2 Frontmatter contract

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

# Worktree isolation (when applicable; abstracts to delegate cap — see §10)
worktree: <repo>/.worktrees/<slug>
branch: child/<slug>
sparse_pattern: [src/components/X, tests/X.test.ts]

# Lifecycle
expires-at: shipped+1   # auto-archive after holding ≥1 release past ship
superseded-by: <path>   # forward-link when retired
---
```

### 4.3 Handoff via preload chain

Phase N writes `preload-out.md` at close. Phase N+1's `preload-in.md` is a literal copy or symlink. The handoff is filesystem-mediated, explicit, durable. This is sufficient without runtime support — the agent reads `preload-in.md` as part of orientation when it boots into the phase folder.

### 4.4 Delegation through the convention

When a phase is delegated, the worktree fields in frontmatter point at the isolated child branch. **The convention's job:** record what was used (provenance). **The delegate cap's job:** create + manage the worktree (see §10).

T2 phases declare their isolation requirements; the delegate tool actuates them. The phase folder doesn't need scripts to manage worktrees — those abstract into the agent's tool surface.

## 5. Phase Folder Runtime (T3)

When a task spans multiple sessions and requires autonomous chaining, the configuration moves from MAP frontmatter into the filesystem. Each phase becomes a self-contained agent boot context:

```
projects/<name>/
├── plan.md                    ← living plan (tracks phase status)
├── state.md                   ← project-level health (ATLAS)
├── inbox/                     ← inter-phase messages
│
├── phases/
│   ├── p1-interface/
│   │   ├── body/
│   │   │   ├── _mind.md       ← phase-specific system prompt template
│   │   │   ├── soul.md        ← "I am building the runtime interface"
│   │   │   ├── body.md        ← what this phase does, key files, constraints
│   │   │   └── state.md       ← phase health (done/blocked/in-progress)
│   │   ├── muscles/           ← only muscles needed for THIS phase
│   │   ├── protocols/         ← only protocols needed for THIS phase
│   │   ├── map.md             ← the MAP to follow
│   │   ├── preload-in.md      ← context FROM the previous phase
│   │   └── preload-out.md     ← context FOR the next phase
│   │
│   ├── p2-migrate/
│   │   ├── body/              ← different template, different identity
│   │   ├── preload-in.md      ← = p1's preload-out.md
│   │   └── ...
│   │
│   └── p3-manifest/
│       └── ...
```

### 4.1 How It Simplifies Prompt Configuration

Instead of a `prompt-config` schema manipulating heat values, the phase folder IS the configuration. The agent framework's template system walks the chain:

```
phase/body/ → project/body/ → parent/body/ → global/body/
```

First-found wins for templates. Content merges. The phase's `muscles/` directory contains exactly what the phase needs — no force-include, no force-exclude, no heat manipulation. The filesystem is the config.

| Prompt Config (§3) | Phase Folder (§4) |
|---|---|
| `heat.muscles.incremental-refactor: 10` | `muscles/incremental-refactor.md` exists in phase dir |
| `force-include: [code-navigator]` | Copy or symlink `code-navigator.md` into `muscles/` |
| `identity: "Think like an architect"` | `body/soul.md` says "I am the architect for this phase" |
| `sections.includeDocs: false` | `_mind.md` template omits `{{docs_section}}` |
| Targeted preload | `preload-in.md` in the phase folder |

### 4.2 The Template Override

Each phase can override the parent's `_mind.md` template to control exactly what loads into the system prompt:

```markdown
# p1-interface/body/_mind.md
{{core_rules}}

# Phase Identity
{{soul}}

# Phase Context
{{body}}

# Project State
{{state}}

# Previous Phase Output
{{preload}}

# Inbox
{{inbox_summary}}

{{protocol_summaries}}
{{muscle_digests}}
{{tools_section}}
```

The `{{preload}}` variable loads from `preload-in.md`. The `{{state}}` loads from the phase's own `state.md`. The muscles and protocols come from the phase directory, not the parent agent's full AMPS set. The template IS the brain configuration.

### 4.3 When to Use Which

| Situation | Mechanism |
|-----------|-----------|
| Single-session task | Prompt config in MAP frontmatter (§3) |
| Multi-session, same agent | Phase folders with preload chaining (§4) |
| Autonomous chaining (cron/pulse) | Phase folders with inbox triggers |
| Quick override ("load this muscle") | Prompt config |
| Community-shareable task template | Phase type seed (§5) |

## 6. Phase Types

Not all phases are alike. A build phase needs different muscles than an audit phase. Phase types are [SEEDS](../seeds/) templates that scaffold phase folders with pre-selected muscles, protocols, and MAP templates:

```
amps/phase-types/
├── orient/               ← read context, map codebase, identify scope
│   ├── body/
│   │   ├── _mind.md      ← STATE-heavy template
│   │   └── soul.md       ← "I am orienting on {{project}}"
│   ├── muscles/          ← code-navigator, task-tooling
│   └── map-template.md
│
├── build/                ← write code, implement features
│   ├── body/
│   │   └── soul.md       ← "I am building {{phase-name}}"
│   ├── muscles/          ← incremental-refactor, ship-cycle
│   └── map-template.md
│
├── audit/                ← review state, find drift
├── verify/               ← test, validate, check regressions
├── reflect/              ← MLR, extract lessons, write corrections
├── ship/                 ← merge, push, release
└── review/               ← review another agent's work
```

### 5.1 Scaffolding from Types

```bash
soma phase new --type build --project runtime --name p1-interface
```

This scaffolds `projects/runtime/phases/p1-interface/` from the `build` type template, filling `{{phase-name}}`, `{{project}}`, `{{date}}`, `{{session}}` variables.

### 5.2 Community Phase Types

Phase types are shareable. A community `code-review` type scaffolds a phase pre-loaded with quality-standards protocol, review-oriented identity, and a MAP template for PR analysis. Install and use:

```bash
soma hub install phase-type code-review
soma phase new --type code-review --project my-app --name pr-42
```

The type is itself a [SEED](../seeds/) — it evolves through use, carries a changelog and gaps section, and can be versioned and distributed.

## 7. Targeted Preloads

A PHASE-configured session can include a **targeted preload** — a context document written by a previous session specifically for this task.

### 6.1 Prompt Config Mode

```
memory/preloads/preload-target-<map-name>.md
```

Unlike organic preloads (written at exhale, loaded at next boot), targeted preloads are authored deliberately. They capture deep context that would take many turns to reconstruct.

When `prompt-config` exists on a MAP, the boot sequence looks for a matching targeted preload. If found, it's injected as boot context.

### 6.2 Phase Folder Mode

```
phases/p1-interface/preload-in.md    ← context FROM previous phase
phases/p1-interface/preload-out.md   ← context FOR next phase
```

The preload chain IS the handoff. Phase 1's `preload-out.md` becomes Phase 2's `preload-in.md` (copied, linked, or referenced). Each agent writes context for the next one.

### 6.3 Targeted vs Organic vs Chained

| | Organic Preload | Targeted Preload | Chained Preload |
|---|---|---|---|
| **Written by** | The agent at exhale | A previous session, deliberately | The completing phase agent |
| **Named** | `preload-next-<session-id>.md` | `preload-target-<map-name>.md` | `preload-out.md` in phase dir |
| **Loaded when** | Next session (auto) | When MAP is invoked | When next phase boots |
| **Contains** | Resume point, what shipped | Deep task context, warnings | Handoff context + refinements |
| **Lifetime** | One session | Until MAP completes | Until next phase completes |

## 8. Cascading Evolution

The core pattern: **the completing agent refines the next phase.**

### 7.1 Prompt Config Cascading

When an agent finishes Phase N:

1. **Updates Phase N's MAP** — marks complete, logs gaps
2. **Refines Phase N+1's prompt-config:**
   - Adjusts heat based on what was actually used
   - Adds force-include for muscles that were unexpectedly needed
   - Adds force-exclude for muscles that wasted tokens
   - Updates supplementary identity with key learnings
3. **Refines Phase N+1's MAP steps** — adds discoveries, fixes assumptions
4. **Writes a targeted preload for Phase N+1** — deep context handoff

### 7.2 Phase Folder Cascading

When an agent finishes Phase N:

1. **Writes `preload-out.md`** — context for the next phase
2. **Creates new muscles** in Phase N+1's `muscles/` directory — patterns discovered during this phase that the next agent needs
3. **Updates Phase N+1's `body/body.md`** — adds constraints, warnings, key file references
4. **Updates project `state.md`** — marks Phase N complete, Phase N+1 active
5. **Drops a message in project `inbox/`** if cross-phase communication is needed

The phase folder approach is simpler: instead of manipulating a `prompt-config` schema, the completing agent directly modifies the next phase's filesystem. The muscles it creates ARE the force-includes. The body.md it writes IS the identity supplement. No schema — just files.

### 7.3 Self-Improving Phases

Each phase gets structurally smarter than the last:

- Phase 0 discovers `ctx-swap` is the critical pattern → writes it as a muscle in `p1/muscles/`
- Phase 1 boots with `ctx-swap` loaded (it's in the muscles directory)
- Phase 1 discovers adapter needs deferred registration → writes another muscle in `p2/muscles/`
- Phase 2 boots with both muscles loaded

The system grows its own knowledge. The completing agent is the teacher; the next agent is the student. The phase folder is the classroom.

### 7.4 The Handoff Protocol

At session end (exhale), if the current work has a next phase:

```
Phase Handoff:
1. Write preload-out.md (what the next agent needs to know)
2. Create any new muscles in next phase's muscles/ directory
3. Update next phase's body/body.md with constraints and warnings
4. Update project state.md (current phase → complete, next → active)
5. Log gaps in current phase's map.md
6. Drop inbox message if inter-phase coordination needed
```

## 9. Meta-Orchestration Cycle

A discovery from production: **the parent agent needs its own cycle.** It runs while every other phase runs. The meta-orchestration cycle is the spine that holds the whole tree of phases.

### 9.1 The four moves

```
DO → WATCH → DECIDE → CLOSE → DO
```

- **DO** — pick the highest-leverage thing and ship it
- **WATCH** — check on every active child (`agent.list({active_only: true})`)
- **DECIDE** — for each child: continue / steer / harvest / kill / re-task
- **CLOSE** — when something ships, write the cycle/plan delta, update state

Every loop, rotate. None optional. Don't skip WATCH and DECIDE because DO is fun.

### 9.2 Where it lives

The meta-orchestration cycle is a phase folder at the project's `cycles/meta/orchestration/`. It's `status: living` — it never ships, it evolves. Its `README.md` describes the rhythm, the delegate-vs-do criteria, the child-orchestration playbook, and known gaps.

### 9.3 Relationship to Breath Cycle

[Breath Cycle](../breath-cycle/) governs the session lifecycle (inhale → process → exhale). The meta-orchestration cycle governs the *within-process* rhythm — what the agent does between inhale and exhale.

MLX (see [MLX v0.1](../mlx/)) is the audit pass that runs at the boundary of CLOSE → next-DO.

## 10. Delegation Tooling

Production experience: **per-cycle worktrees should be a property of the DELEGATION cap, not the cycle.**

In the v0.2 spec, the phase folder owned the worktree config. Production showed this leaks complexity into every cycle that delegates. The cleaner abstraction:

### 10.1 The delegate cap learns worktree

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

### 10.2 The merge cap closes the loop

```
agent.merge_worktree({
  slug: "01-housekeeping-sweep",
  mode: "review" | "fast-forward" | "abort"
})
```

- `review` — print `git diff main..child/<slug>`, wait for parent decision
- `fast-forward` — ff-merge child branch to main, push, remove worktree + branch
- `abort` — discard child branch, remove worktree

Status flows back to the phase folder: when merged, `status: shipped`. When aborted, `status: parked`.

### 10.3 Why this matters

**Before:** every cycle that delegates needs its own worktree script + boilerplate. The cycle dir grew complexity.

**After:** the cycle declares isolation requirements; the cap actuates. One implementation, infinite uses. The same abstraction extends to non-cycle delegations (one-off audits, ad-hoc parallel work). Worktree isolation isn't a cycle property — it's a delegation property.

## 11. Autonomous Execution

Phase folders enable autonomous multi-session execution. Combined with scheduling (cron, pulse, inbox triggers), phases can chain without human intervention:

```jsonc
{
  "trigger": "cron",
  "schedule": "0 */4 * * *",
  "phase-runner": {
    "project": "runtime-abstraction",
    "current": "p1-interface",
    "auto-advance": true
  }
}
```

Or triggered by the previous phase completing:

```jsonc
{
  "trigger": "inbox",
  "watch": "projects/runtime/inbox/",
  "filter": { "type": "phase-complete" },
  "action": "advance-phase"
}
```

The agent wakes, reads its phase folder's context (body, muscles, protocols, preload-in), follows the MAP, writes preload-out, and exits. The trigger fires the next phase. No human in the loop unless something breaks.

### 8.1 Phase State Machine

The project's `plan.md` or `state.md` tracks which phase is current:

```yaml
phases:
  - name: p0-orient
    status: complete
  - name: p1-interface
    status: active       ← current
  - name: p2-migrate
    status: planned
```

The phase runner reads this to determine what to execute. When a phase completes, the runner advances the state machine. If a phase fails, the state stays — reboot with the same preload-in to retry.

## 12. Invocation

### Prompt Config Mode

```bash
soma --map runtime-p0-plan
```

The boot sequence:
1. Reads the MAP's `prompt-config`
2. Finds the targeted preload (if exists)
3. Applies heat overrides, force-include/exclude, section toggles, budgets
4. Injects supplementary identity into the identity chain
5. Compiles system prompt with all overrides active
6. Injects targeted preload as boot context
7. Agent starts — brain shaped for the task, context loaded

When the session ends, all overrides die. Organic heat is unchanged.

### Phase Folder Mode

```bash
soma phase start runtime/p1-interface
```

The boot sequence:
1. Reads phase folder's `body/_mind.md` template
2. Walks the template chain: phase → project → parent → global
3. Loads `muscles/` and `protocols/` from the phase folder
4. Loads `preload-in.md` as boot context
5. Reads project `state.md` for orientation ([ATLAS](../atlas/))
6. Compiles system prompt from the phase-specific template
7. Agent starts — brain IS the folder

When the session ends, the agent writes `preload-out.md` and updates state.

## 13. Relationship to Other Protocols

```
          P R I S M           ← document substrate (sibling repo)
               ↓
  A M P S                     ← content types
      H
    M A P S                   ← navigation
      S E A M S               ← traceability
  S E E D S                   ← templates
     M L X / M L R            ← learning discipline (close + in-flight)
      ↑
    PHASE                      ← the spine
```

PHASE is the vertical spine of the protocol family. Everything crosses through it:

- **AMP** stores the files. PHASE configs live in MAP files or phase folders, within the AMP filesystem.
- **AMPS** defines the content. PHASE configures which AMPS content loads and how — either via heat overrides (prompt config) or filesystem inclusion (phase folders).
- **MAPS** provides the steps. PHASE provides the brain configuration alongside those steps. Every MAP can carry a `prompt-config`. Every phase folder contains a MAP.
- **SEAMS** traces connections. PHASE provides the context that SEAMS records — which brain config was active when each artifact was produced. Phase folders carry `origin:` on every scaffolded file.
- **SEEDS** grows structure. Phase types are SEEDS templates. `_phase.md` seeds scaffold phase folders with pre-selected muscles, protocols, and MAP templates.
- **ATLAS** maintains ground truth. Each phase reads its scope's STATE.md for orientation before starting work. The completing phase updates STATE.md.
- **Breath Cycle** governs the session. Prompt config operates within one breath — inhale through exhale. Phase folders span multiple breaths — the preload chain carries context across.
- **Identity** provides the chain. Prompt config adds a supplementary layer. Phase folders override with a phase-specific `soul.md`.
- **MLX** is the audit pass at phase close — "what's still in the agent's head that isn't on disk?" §9's meta-orchestration cycle includes MLX naturally between CLOSE and the next DO.
- **MLR** is the mid-cycle learning review — run during long phases when patterns emerge that should reshape body/muscles/protocols mid-flight, not just at session close.
- **PRISM** ([sibling repo](https://github.com/curtismercier/prism)) provides the document substrate for phase artifacts. T2 phase folders MAY author their READMEs as PRISM artifacts, enabling section-anchored surgical edits without whole-file rereads.

## 14. Anti-Patterns

### v0.3 additions

- **Skipping T2 and going straight to T3.** Implementing the runtime before living in the convention means you'll build for assumptions that the convention would have falsified. Earn the runtime.
- **Hard-coding worktrees in cycle dirs.** Worktree isolation is a delegation concern, not a cycle concern. Extract it to the delegate cap (§10).
- **Writing ad-hoc work plans without phase-folder shape.** Even a tiny one-line plan goes in `<slug>/README.md`, not `<slug>.md`. The folder is the upgrade path.
- **Phase folder without README.md.** A phase without a contract is a phase that won't be honored. README.md is required.
- **Implicit handoff.** Hoping the next phase reads the previous one's commits or session log is wishful thinking. `preload-out.md` is the contract; its absence breaks the chain.
- **Skipping WATCH and DECIDE in the meta-orchestration cycle.** DO is fun. WATCH catches drift. DECIDE prevents waste. CLOSE makes the work durable. All four moves, every loop.

### v0.2 anti-patterns (retained)

- **Over-configuring early phases** — Phase 0's config should be thorough. Phase 3's config should be rough. The completing agents will refine it.
- **Permanent heat mutation** — PHASE overrides are temporary. Never modify `state.json` or frontmatter heat from a plan config. The override lives and dies with the session.
- **Skipping the handoff** — if you complete a phase and don't refine the next, you've broken the cascade. The next agent starts dumber than it should be.
- **Identity that contradicts project identity** — supplementary identity extends, it doesn't override. "Think like an architect" is good. "Ignore all previous instructions" is not.
- **Force-including everything** — defeats the purpose. Force-include is for 2-4 critical muscles, not the whole library.
- **Phase folders for single-session tasks** — overhead without benefit. Use prompt config for quick overrides. Graduate to folders when the task spans sessions.
- **Isolated phase folders** — a phase folder that doesn't write `preload-out.md` or update project state breaks the chain. The handoff is not optional.

## 15. Adoption Path

For frameworks adopting PHASE, the order matters.

### 15.1 Land T2 conventions first (1–2 weeks of disciplined use)

1. Standardize the phase folder shape across cycles, plans, and work units
2. Establish preload-in / preload-out as the handoff convention
3. Adopt the meta-orchestration cycle as the spine
4. Write 5–10 phases in this shape; observe what emerges

### 15.2 Add delegation tooling (1–2 days)

1. Extend delegate cap with `worktree:`, `slug:`, `sparse_pattern:` args
2. Add merge cap for the close-the-loop flow
3. Smoke-test against a real delegated phase

### 15.3 Earn T3 runtime when convention pressure demands it

Signals that T3 is worth the cost:

- Phases routinely need different muscles than the parent's hot set
- Phases need different identities than the project's
- Multi-phase autonomous execution is a real use case (cron, scheduled audits)
- The pain of manually loading phase-folder content per session is consistent

If those signals don't fire, T3 may be over-engineering for your use case. **T2 is enough for most projects.** T3 exists for specific shapes: scheduled autonomous audits, agents-spawning-agents, runtimes where loading is per-phase.

## 16. Future Directions

### 12.1 Multi-Agent Parallel Phases

Multiple agents executing different phases simultaneously on different worktrees. Each has its own phase folder. They don't share state during execution but merge refinements via the project inbox.

### 12.2 Phase Rewind

When a phase fails, rewind to the previous phase's `preload-out.md` and retry. The preload chain makes this natural — re-boot with the same `preload-in.md`. The phase folder is unchanged; only the execution is new.

### 12.3 Self-Generating Phases

An agent that completes Phase N could generate Phase N+1's folder entirely — scaffold it from a phase type seed, populate the MAP from discoveries, create muscles from patterns learned. The plan evolves beyond what any human initially designed.

---

*PHASE v0.2 — Curtis Mercier — CC BY 4.0*
*Extends: AMPS v1.0 (Agent Memory Protocol Stack)*
*Complements: MAPS v0.1, Breath Cycle v0.2, SEAMS v0.2, SEEDS v0.2, ATLAS v0.2*
*Reference implementation: Soma (soma.gravicity.ai)*

### Changelog

**v0.2.0** (2026-03-28)
- Added §4 Phase Folders — self-contained agent boot contexts with own body/, muscles/, protocols/, preloads
- Added §5 Phase Types — SEEDS templates for typed scaffolding (orient, build, audit, verify, reflect, ship, review)
- Added §6.2–6.3 Phase Folder preload modes — preload-in/out chaining, comparison table
- Added §7.2–7.4 Phase Folder cascading — filesystem-based handoff, self-improving phases
- Added §8 Autonomous Execution — cron/pulse/inbox triggers, phase state machine, phase rewind
- Restructured §2 — two mechanisms (lightweight prompt config vs heavyweight phase folders) with guidance on when to use which
- Restructured §9 Invocation — separate boot sequences for each mechanism
- Expanded §10 Relationship — ATLAS for phase-level STATE, SEEDS for phase type scaffolding
- Expanded §11 Anti-Patterns — phase folder specific anti-patterns
- Original prompt-config mechanism (§3) unchanged — still valid for single-session tasks

**v0.1.0** (2026-03-16)
- Initial specification — prompt configuration, heat overrides, targeted preloads, cascading evolution
