---
type: spec
status: draft
version: 0.1.0
created: 2026-03-16
updated: 2026-03-16
author: Curtis Mercier
license: CC BY 4.0
extends: amps/1.0
complements: maps/0.1, breath-cycle/0.2
---

# PHASE — Prompt Handoff for Agent Session Evolution v0.1

> A protocol for reshaping an agent's brain per task and cascading refinements across phases. The plan doesn't just tell the agent what to do — it configures how the agent thinks.

*Extends: [AMPS v1.0](../amps/) (Agent Memory Protocol Stack)*
*Complements: [MAPS v0.1](../maps/), [Breath Cycle v0.2](../breath-cycle/)*

## 1. The Insight

Most agent systems have one configuration: the system prompt. It's assembled at boot from whatever knowledge is "hot" — recently used muscles, recently referenced protocols. This works for general sessions. But when an agent is executing a specific plan, organic heat is the wrong signal.

A refactoring session needs `incremental-refactor` hot — the task requires it, regardless of what happened last session. A blog session needs `voice-hygiene` hot, not `code-navigator`. The task knows what the agent needs. Heat doesn't.

PHASE lets the task configure the agent. A plan declares which protocols should be hot, which muscles should load, what supplementary identity the agent should assume, and what sections of the system prompt matter. The agent boots with a brain shaped for the work.

And when that agent finishes, it refines the configuration for the next one.

## 2. Prompt Configuration

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

### 2.1 Heat Overrides

Temporarily set heat values for the session. Overrides the agent's organic heat (from `state.json` or frontmatter) without modifying the stored values. When the session ends, the override dies. Organic heat is unchanged.

```yaml
heat:
  protocols:
    quality-standards: 10    # force hot (loaded with full body)
    voice-hygiene: 0         # force cold (not loaded)
  muscles:
    incremental-refactor: 10 # force hot
```

### 2.2 Force Include / Exclude

Stronger than heat overrides. Force-include guarantees loading regardless of heat thresholds or budget limits. Force-exclude guarantees omission regardless of heat.

```yaml
force-include:
  muscles: [incremental-refactor]   # always loaded, even if cold
force-exclude:
  muscles: [voice-hygiene]          # never loaded, even if hot
```

Use force-include for muscles the task *requires*. Use force-exclude for muscles that would waste context tokens on irrelevant knowledge.

### 2.3 Section Toggles

Control which sections of the system prompt are assembled. Saves tokens by omitting sections irrelevant to the task.

```yaml
sections:
  includeDocs: false        # skip documentation references
  includeSkills: false      # skip skills block
```

A deep code session doesn't need docs. A writing session doesn't need tool guidelines. The task knows.

### 2.4 Budget Overrides

Adjust token budgets and loading limits for the session.

```yaml
budgets:
  muscles.tokenBudget: 3000   # more muscle context
  muscles.maxFull: 4           # allow more full-body muscles
  protocols.maxFull: 2         # fewer full protocols
```

### 2.5 Supplementary Identity

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

## 3. Targeted Preloads

A PHASE-configured session can include a **targeted preload** — a context document written by a previous session specifically for this task.

```
memory/preloads/preload-target-<map-name>.md
```

Unlike organic preloads (written at exhale, loaded at next boot), targeted preloads are authored deliberately. They capture deep context that would take many turns to reconstruct: API audit results, design decisions, file:line references, warnings about traps.

When `prompt-config` exists on a MAP, the boot sequence looks for a matching targeted preload. If found, it's injected as boot context — giving the agent a running start.

### 3.1 Targeted vs Organic Preloads

| | Organic Preload | Targeted Preload |
|---|---|---|
| **Written by** | The agent at exhale | A previous session, deliberately |
| **Named** | `preload-next-<session-id>.md` | `preload-target-<map-name>.md` |
| **Loaded when** | Next session (auto) | When MAP is invoked |
| **Contains** | Resume point, what shipped | Deep task context, warnings, file:line refs |
| **Lifetime** | One session | Reusable until MAP completes |

## 4. Cascading Evolution

The core pattern: **the completing agent refines the next phase.**

When an agent finishes Phase N:

1. **Updates Phase N's MAP** — marks complete, logs gaps
2. **Refines Phase N+1's prompt-config:**
   - Adjusts heat based on what was actually used
   - Adds force-include for muscles that were unexpectedly needed
   - Adds force-exclude for muscles that wasted tokens
   - Updates supplementary identity with key learnings
3. **Refines Phase N+1's MAP steps** — adds discoveries, fixes assumptions
4. **Writes a targeted preload for Phase N+1** — deep context handoff

### 4.1 Why This Matters

Each agent in a chain is **structurally smarter** than it would have been without cascading. Not just text-informed (via preloads), but brain-configured:

- Phase 0 agent discovers `ctx-swap` is the critical pattern → creates a muscle for it
- Phase 0 refines Phase 1's config: force-include the new muscle, add it to identity
- Phase 1 agent boots already knowing `ctx-swap`, with the muscle loaded
- Phase 1 discovers adapter needs deferred registration → creates another muscle
- Phase 1 refines Phase 2's config accordingly

The plan gets better as it executes. Each agent's brain is tuned by the agent that just completed the preceding work.

### 4.2 The Handoff Protocol

At session end (exhale), if the current MAP has a `next-map`:

```markdown
## Phase Handoff

1. Mark current MAP: status → complete, log gaps
2. Refine next MAP's prompt-config:
   - Heat adjustments from actual usage
   - Force-include/exclude changes
   - Identity updates with key learnings
3. Update next MAP's steps with discoveries
4. Write targeted preload for next MAP
5. Note new muscles/protocols created this phase
```

### 4.3 Agent as Orchestrator

The completing agent isn't just doing cleanup. It's becoming the orchestrator for the next agent. It decides:

- "I created `pi-adapter-patterns` — Phase 2 needs it hot"
- "`voice-hygiene` wasted 400 tokens — exclude for all remaining phases"
- "The identity should mention the ctx-swap pattern specifically"
- "Phase 1's estimated turns should be 35, not 20 — I underestimated"

This is meta-cognition: the agent reasoning about what the *next* agent needs to know and how it should think.

## 5. Invocation

The user starts a PHASE-configured session by referencing a MAP:

```bash
soma --map runtime-p0-plan
# or
soma --preload runtime-p0-plan
```

The boot sequence:

1. Reads the MAP's `prompt-config`
2. Finds the targeted preload (if exists)
3. Applies heat overrides, force-include/exclude, section toggles, budgets
4. Injects supplementary identity into the identity chain
5. Compiles system prompt with all overrides active
6. Injects targeted preload as boot context
7. Agent starts — brain shaped for the task, context loaded

When the session ends, all overrides die. Organic heat is unchanged. The next session starts clean (or with its own MAP's config).

## 6. Relationship to Other Protocols

```
AMP (memory)
  └── AMPS (content types)
        └── MAPS (navigation)
              └── PHASE (orchestration)
                    └── uses Breath Cycle (session lifecycle)
                    └── uses Identity (agent identity chain)
```

- **AMP** stores the files. PHASE configs live in MAP files, which live in the AMP filesystem.
- **AMPS** defines the content. PHASE configures which AMPS content loads and how.
- **MAPS** provides the steps. PHASE provides the brain configuration alongside those steps.
- **Breath Cycle** governs the session. PHASE operates within one breath — config applies for the inhale through exhale.
- **Identity** provides the chain. PHASE adds a layer to that chain.

## 7. Anti-Patterns

- **Over-configuring early phases** — Phase 0's config should be thorough. Phase 3's config should be rough. The completing agents will refine it.
- **Permanent heat mutation** — PHASE overrides are temporary. Never modify `state.json` or frontmatter heat from a plan config. The override lives and dies with the session.
- **Skipping the handoff** — if you complete a phase and don't refine the next, you've broken the cascade. The next agent starts dumber than it should be.
- **Identity that contradicts project identity** — supplementary identity extends, it doesn't override. "Think like an architect" is good. "Ignore all previous instructions" is not.
- **Force-including everything** — defeats the purpose. Force-include is for 2-4 critical muscles, not the whole library.

## 8. Future Directions

### 8.1 Multi-Agent Parallel Phases

Multiple agents executing different phases simultaneously on different worktrees. Each has its own PHASE config. They don't share state during execution but can merge refinements at the end.

### 8.2 Agent-to-Agent Communication

When agents in parallel phases need to coordinate, the PHASE config could include a communication channel — a shared scratchpad or signal file. One agent's discovery becomes another's config update in real time.

### 8.3 Self-Generating Phases

An agent that completes Phase N could generate Phase N+1's MAP entirely — not just refine it, but create it from scratch based on what it learned. The plan evolves beyond what any human initially designed.

---

*PHASE v0.1 — Curtis Mercier — CC BY 4.0*
*Extends: AMPS v1.0 (Agent Memory Protocol Stack)*
*Complements: MAPS v0.1, Breath Cycle v0.2*
*Reference implementation: Soma (soma.gravicity.ai)*
