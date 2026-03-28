---
type: spec
status: draft
version: 0.2.0
created: 2026-03-16
updated: 2026-03-28
author: Curtis Mercier
license: CC BY 4.0
extends: amps/1.0
complements: maps/0.1, breath-cycle/0.2, seams/0.2, seeds/0.2, atlas/0.2
---

# PHASE — Prompt Handoff for Agent Session Evolution v0.2

> A protocol for reshaping an agent's brain per task and cascading refinements across phases. The plan doesn't just tell the agent what to do — it configures how the agent thinks.

*Extends: [AMPS v1.0](../amps/) (Agent Memory Protocol Stack)*
*Complements: [MAPS v0.1](../maps/), [Breath Cycle v0.2](../breath-cycle/), [SEAMS v0.2](../seams/), [SEEDS v0.2](../seeds/), [ATLAS v0.2](../atlas/)*

## 1. The Insight

Most agent systems have one configuration: the system prompt. It's assembled at boot from whatever knowledge is "hot" — recently used muscles, recently referenced protocols. This works for general sessions. But when an agent is executing a specific plan, organic heat is the wrong signal.

A refactoring session needs `incremental-refactor` hot — the task requires it, regardless of what happened last session. A blog session needs `voice-hygiene` hot, not `code-navigator`. The task knows what the agent needs. Heat doesn't.

PHASE lets the task configure the agent. The agent boots with a brain shaped for the work. And when that agent finishes, it refines the configuration for the next one.

## 2. Two Mechanisms

PHASE operates through two mechanisms. They solve the same problem — task-driven brain configuration — at different scales.

### Lightweight: Prompt Configuration

For ad-hoc tasks, quick overrides, or single-session work. The configuration lives in the MAP's frontmatter.

### Heavyweight: Phase Folders

For multi-session projects with chained phases. Each phase is a self-contained agent context — its own body, muscles, protocols, and preloads. The template system resolves the folder as a mini agent boot environment.

Most tasks start with prompt configuration. When a task grows beyond one session or needs autonomous chaining, it graduates to phase folders. Both use the same principles: the task configures the brain, and the completing agent refines the next phase.

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

## 4. Phase Folders (Heavyweight)

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

## 5. Phase Types

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

## 6. Targeted Preloads

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

## 7. Cascading Evolution

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

## 8. Autonomous Execution

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

## 9. Invocation

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

## 10. Relationship to Other Protocols

```
  A M P S           ← content types
      H
    M A P S         ← navigation
      S E A M S     ← traceability
  S E E D S         ← templates
      ↑
    PHASE            ← the spine
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

## 11. Anti-Patterns

- **Over-configuring early phases** — Phase 0's config should be thorough. Phase 3's config should be rough. The completing agents will refine it.
- **Permanent heat mutation** — PHASE overrides are temporary. Never modify `state.json` or frontmatter heat from a plan config. The override lives and dies with the session.
- **Skipping the handoff** — if you complete a phase and don't refine the next, you've broken the cascade. The next agent starts dumber than it should be.
- **Identity that contradicts project identity** — supplementary identity extends, it doesn't override. "Think like an architect" is good. "Ignore all previous instructions" is not.
- **Force-including everything** — defeats the purpose. Force-include is for 2-4 critical muscles, not the whole library.
- **Phase folders for single-session tasks** — overhead without benefit. Use prompt config for quick overrides. Graduate to folders when the task spans sessions.
- **Isolated phase folders** — a phase folder that doesn't write `preload-out.md` or update project state breaks the chain. The handoff is not optional.

## 12. Future Directions

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
