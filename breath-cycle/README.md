---
type: spec
status: draft
version: 0.2.0
created: 2026-03-10
updated: 2026-05-12
author: Curtis Mercier
license: CC BY 4.0
complements: amp/0.3, phase/0.3, mlx/0.1, mlr/0.1
---

# Breath Cycle — Specification v0.2

> A lifecycle model for AI agent sessions that embraces context depletion as a design constraint rather than a limitation.

## 1. The Insight

Most frameworks treat the context window as a problem to solve — make it bigger (long context), make it searchable (RAG), make it infinite (external memory).

The Breath Cycle treats it differently: **the context window is a breath.** It fills, it empties, it fills again. The agent knows this and prepares for it.

An agent that knows it will forget is more capable than one that pretends it won't.

## 2. The Four Phases

```
    ┌─── INHALE ───┐
    │ Load identity │
    │ Read preload  │
    │ Init memory   │
    └───────┬───────┘
            │
    ┌───────▼───────┐
    │    PROCESS    │
    │  Work, think  │
    │ Context fills  │
    └───────┬───────┘
            │
    ┌───────▼───────┐
    │    EXHALE     │
    │ Detect 85%    │
    │ Extract state │
    │ Write preload │
    └───────┬───────┘
            │
    ┌───────▼───────┐
    │     REST      │
    │Between sessions│
    │Memory on disk  │
    └───────────────┘
```

### 2.1 Inhale (Boot)

The agent wakes up and orients itself.

1. **Discover identity** — find `.soma/identity.md`, learn who it is here
2. **Check for preload** — find `preload-next.md`, learn where the last session left off
3. **Load relevant muscles** — read memory files that match the current context/project
4. **Orient** — "Here's who I am, here's where we left off, here's what I know"

The inhale should be fast. The agent isn't re-learning — it's remembering.

### 2.2 Process (Work)

The agent works with the human. Context accumulates.

- Every user message uses context
- Every assistant response uses context
- Every tool call (file read, command execution) uses context
- Context usage is monitored continuously (extension or built-in)

The agent is aware of its remaining capacity. This isn't a background concern — it influences behavior. An agent at 40% context works differently from one at 75%.

### 2.3 Exhale (Flush)

Context is running out. The agent prepares for the transition.

1. **Threshold detected** — context usage exceeds limit (default: 85%)
2. **Extract state** — what was accomplished, what's in progress, what's next
3. **Write preload** — session continuation document (see AMP spec)
4. **Crystallize** — if a new pattern emerged, write a muscle
5. **Signal** — indicate readiness for continuation or termination

The exhale is deliberate, not panicked. The agent has been aware of its depleting context throughout the process phase.

**Checkpoints:** On exhale, the agent also commits state to its checkpoint tracks (see AMP spec §8). The internal track (`.soma/` local git) captures memory changes. The project track captures code checkpoints. These provide diffs on the next inhale — "what changed since I was last here?" — without re-reading everything.

### 2.4 Rest (Between Sessions)

The agent isn't running, but its memory persists.

- Muscles remain on disk, ready for the next inhale
- Preload waits to be consumed
- The human can modify memory between sessions (edit muscles, adjust preloads)
- Nothing is lost — just sleeping

## 3. Continuation

When exhale leads directly to a new inhale (auto-continuation):

```
Session N:   inhale → process → exhale
                                  ↓
Session N+1: inhale → process → exhale
                                  ↓
Session N+2: inhale → process → exhale → rest
```

Auto-continuation enables long-running tasks that exceed a single context window. The agent works until done, breathing as needed.

Requirements for auto-continuation:
- Preload written successfully
- New session can be started programmatically
- The new session's inhale consumes the preload from the previous exhale

In multi-phase work, each breath can carry a different **[PHASE](../phase/)** configuration — the agent's brain reshapes between sessions. The completing agent refines the next session's configuration during exhale, so the next inhale boots a smarter agent.

## 4. Context Awareness

The agent should know its own context state. Suggested thresholds:

| Context Used | State | Agent Behavior |
|-------------|-------|---------------|
| 0-40% | Fresh | Full exploration, read files freely |
| 40-65% | Working | Normal operation, mindful of reads |
| 65-80% | Warning | Prioritize, avoid unnecessary reads |
| 80-85% | Critical | Prepare for exhale, start extracting |
| 85%+ | Flush | Execute exhale sequence |

These thresholds are guidelines, not hard rules. Implementations may adjust based on the model's context window size.

## 5. Why "Breath"

The metaphor isn't decoration. It shapes design decisions:

- **Breathing is natural.** Flush isn't failure — it's the exhale that enables the next inhale.
- **Sessions are breaths, not lifetimes.** Each one is complete in itself but part of a longer life.
- **You can't hold your breath forever.** Context is finite. Accept it.
- **Breathing is rhythmic.** The cycle repeats. The agent gets better at it over time.
- **Breathing out isn't losing.** The exhale (flush) preserves what matters. The rest is exhaled.

## 6. Implementation Notes

- The Breath Cycle works with any context window size (4K, 128K, 1M tokens)
- Larger context windows mean longer breaths, not the elimination of breathing
- The flush threshold should be tuned to the model and use case
- Auto-continuation requires framework support (session management, programmatic restart)

## 7. Anti-Patterns

- **"Just one more thing" past the exhale.** When the breath signals exhale, the discipline is to actually exhale — write the preload, flush state, close cleanly. Pushing past exhale because the work feels almost-done is how preloads get rushed and continuity breaks.
- **No preload written at exhale.** The preload IS the breath cycle's output. A session that ends without one didn't actually exhale — it just ran out of air.
- **Treating context window as a resource to maximize.** The window is a breath, not a budget. Maximizing fills produces brittle sessions; rhythm produces durable ones.
- **Inhaling everything at boot.** The preload tells you WHAT happened, not the full history. Read it, then reach for specific files as the work demands. Pre-loading the universe defeats the design.
- **Skipping MLX during exhale.** MLX is the audit pass that catches what's still floating before the breath releases (see MLX spec). Exhaling without MLX leaves orphan lessons.
- **Forcing a continuation when the natural break is here.** A breath that should have ended at exhale but kept going because the work felt unfinished produces lower-quality continuations than a clean rest-and-resume.

## 8. Conformance

A Breath-Cycle-conformant implementation MUST:

1. **Distinguish the four phases** — inhale (boot), process (work), exhale (flush), rest (between sessions). Each phase has different obligations.
2. **Trigger exhale at context-window thresholds** — implementations choose the threshold; the protocol requires that exhale fires before context-window exhaustion forces a hard cut.
3. **Produce a preload at exhale** — the preload-out artifact is mandatory; resume points, what shipped, and read-first targets MUST be present.
4. **Consume a preload at inhale** — the next session boot reads the previous exhale's preload before any other context.
5. **Honor cross-session continuity** — identity, body, journal, soul carry forward across breaths. They are not session-scoped.

A conformant tooling layer SHOULD provide:

- `breathe` — trigger an exhale at the agent's request or at threshold detection
- `rest` — close the current session cleanly
- `inhale` — next-session boot that reads the previous preload first
- Context-window monitoring + threshold alerts (50% / 70% / 85% common values)

## 9. Attribution

```
This project implements the Breath Cycle
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Breath Cycle v0.2 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
