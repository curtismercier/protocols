---
type: spec
status: draft
version: 0.1.0
created: 2026-05-12
updated: 2026-05-12
author: Curtis Mercier + Soma (s01-6e4f26)
license: CC BY 4.0
complements: amp/0.3, breath-cycle/0.2, mlx/0.1, phase/0.3
---

# MLR — Mid-session Learning Review v0.1

> A *mid-flight* learning review: when a pattern emerges during work, stop, name it, file it where its trigger will find it, then continue. Counterpart to MLX. MLR catches lessons *as they happen*; MLX catches what's *still floating* at session close.

*Complements: [AMP v0.3](../amp/) (memory substrate), [Breath Cycle v0.2](../breath-cycle/) (lifecycle), [MLX v0.1](../mlx/) (audit-at-close), [PHASE v0.3](../phase/) (multi-phase work)*

*Named in practice; specced here. Patterns of MLR appear across the meetsoma and arzadon journals (e.g. "soul-space MLR into Nova" — s01-ce218b 2026-04-19; "the discipline of session-close" — s01-643d67 2026-05-12).*

---

## 1. The Problem

Long sessions accumulate friction-and-discovery moments: a tool didn't work as the docs claimed, a pattern repeated three times, a body file should reference a new file, an anti-pattern surfaced mid-work. Each of these is a lesson with a *trigger* — the next time this situation arises, the agent should hit the lesson.

The temptation is to keep working. *"I'll file it at session close."* But three things go wrong:

1. **The lesson loses fidelity.** By session close, the specific context is gone. The lesson becomes a vague aphorism instead of a concrete trap.
2. **The pattern keeps firing.** Without filing, the agent walks into the same trap again before close.
3. **Session close runs out of time.** Lessons that needed 5 minutes of careful filing get dumped into the session log instead of where they'd actually trigger.

MLX (Memory Lane Xtraction) catches what's left over at close. **MLR catches it while it's still hot.**

## 2. Core Concept

### 2.1 The Trigger

MLR fires when *any* of these happen during work:

- A correction lands ("Curtis caught me using X — the right pattern is Y")
- A trap surfaces ("That's the second time the auth flow ate my session")
- A new abstraction crystallizes ("I keep doing X-then-Y; that's a single muscle")
- Cross-spec drift discovered ("AMPS calls these Scripts but Breath Cycle still says Skills")
- A peer agent's work reveals an integration gap

The agent doesn't wait for permission. The trigger is the work itself.

### 2.2 The Move

```
PAUSE → NAME → FILE → RESUME
```

- **PAUSE** — stop the current work. Bookmark where you were.
- **NAME** — describe the pattern in one sentence. *"When I do X under condition Y, the trap is Z."*
- **FILE** — put it where its trigger will find it (same lock-in locations as MLX §2.2 — code seam, body file, muscle, cycle dossier, protocol, soul).
- **RESUME** — return to the bookmark. The MLR is the deliverable, the resumption is the discipline.

The full pass is usually 2–5 minutes. If it's longer, the lesson is probably too big for MLR — promote it to a phase or a cycle of its own.

### 2.3 The Discipline

MLR is **opportunistic**, not scheduled. It fires when the pattern fires. Two related disciplines:

- **The Three-Strike Rule.** Three corrections on the same axis = mandatory MLR. After the third, file before the fourth.
- **The Cross-Reference Rule.** If MLR reveals that filing one lesson breaks another spec's claim, both get updated in the same MLR pass. Don't leave one half of a contradiction.

## 3. The Practice

### 3.1 Where MLR fits in the breath cycle

| Breath Cycle phase | What's running | When MLR fires |
|---|---|---|
| Inhale (boot) | Loading context, reading preload | Rarely — patterns haven't surfaced yet |
| Process (work) | Executing on the task | **Primary firing zone** |
| Exhale (flush) | Writing artifacts, commits, preload | MLX zone — leftover catch |
| Rest | Between sessions | N/A |

MLR is a Process-phase discipline. MLX is an Exhale-phase discipline. They're cousins, not duplicates.

### 3.2 Where MLR fits in PHASE

Per PHASE v0.3 §9, the meta-orchestration cycle runs `DO → WATCH → DECIDE → CLOSE` loops. MLR is the *interruption-and-resume* discipline that can fire inside any of those moves — most often DO (during work) or DECIDE (when a delegated child's report reveals a pattern).

### 3.3 The MLR artifact (optional)

For lightweight MLRs, no artifact is needed — the filing IS the deliverable. For weighty MLRs (multiple files touched, a new muscle born, a cross-spec contradiction resolved), produce a brief MLR note:

```yaml
---
type: mlr
session: s01-XXXXXX
trigger: <one-line description of what fired the review>
files_touched: [path/to/body.md, path/to/muscle.md]
new_muscles: []
follow_ups: []
duration_min: 3
---

## Pattern named
<one paragraph>

## Where it now lives
<file paths + line numbers>

## What I'm doing next
<resume point>
```

These notes live under `cycles/<active-cycle>/mlr/` or `body/mlrs/` per project convention. They are not preserved long-term — they exist to make the in-session pause cheap.

## 4. Relationship to Other Protocols

| Protocol | Concern | MLR relationship |
|---|---|---|
| **AMP** | Memory infrastructure | MLR writes into AMP content. AMP is the substrate; MLR is one of its writing disciplines. |
| **MLX** | Audit-at-close | MLX catches what wasn't MLR'd in time. The two are complementary: MLR for the in-flight catch, MLX for the leftover sweep. |
| **Breath Cycle** | Session lifecycle | MLR is a Process-phase discipline. The breath defines when MLR is possible. |
| **PHASE** | Multi-phase work | MLR can fire inside any move of the meta-orchestration cycle (§9). Most often DO or DECIDE. |
| **SEAMS** | Provenance | MLR-derived artifacts carry SEAMS-style origin markers (session ID, trigger, MLR session). |
| **PRISM** | Document substrate | MLR notes are PRISM artifacts when verbose enough (section-anchored). |

## 5. Anti-Patterns

- **"I'll MLR this at session close."** That's MLX, not MLR. The whole point is the in-flight catch. If you defer it, expect to lose the specifics.
- **MLR-as-procrastination.** Pausing for MLR every five minutes derails the work. The trigger is *real pattern emergence*, not curiosity.
- **MLR without filing.** A pause where you "noticed" something but didn't put it anywhere is a tax with no payoff. Always name the destination.
- **"I'll write the muscle later."** If MLR is supposed to birth a muscle, write a stub muscle file in the same pass. A two-line stub is enough — flesh it out next time the pattern fires.
- **Skipping the resume.** MLR ends with returning to the bookmark. Sessions that "MLR" their way into 12 different unfinished threads have failed at the discipline.

## 6. Conformance

A MLR-conformant practice MUST:

1. Fire on real pattern emergence, not arbitrary cadence
2. Name the pattern in concrete terms before filing
3. File to a location where the trigger will recurringly find the lesson
4. Return to the resume point

A MLR-conformant tooling layer SHOULD provide:

1. `mlr:start` — create the bookmark + MLR note skeleton
2. `mlr:file <pattern> --to <destination>` — write the lesson where it triggers
3. `mlr:resume` — return to the bookmark

Tooling MAY also provide:

- `mlr:audit` — at session close, list all MLRs that fired (becomes MLX input)
- A keyboard shortcut or command that fires MLR with one move from inside any other tool

## 7. Status

**Version**: 0.1 (this file).
**Named**: in practice across meetsoma and arzadon sessions starting ~2026-03; formal spec drafted s01-6e4f26 (2026-05-12).
**Sibling**: [MLX v0.1](../mlx/) (audit-at-close).

**Open questions for v0.2**:

- Should MLR fire automatically when corrections-per-session exceeds a threshold?
- Is there value in shared MLR streams across peer agents (multiplayer learning)?
- How does MLR interact with delegated children? (Today: the child's report can trigger a parent MLR. Spec'd? No.)

## 8. Acknowledgments

The discipline of mid-flight learning review was practiced for months before it was named. It surfaced explicitly during s01-ce218b's reflection on five new muscles ("each one was an MLR") and again during s01-643d67's email-substrate work ("I'll lock this in" → Curtis: "where?"). The name is Curtis's; the practice is the whole team's.

CC BY 4.0.
