---
type: spec
status: draft
version: 0.1.0
created: 2026-03-16
updated: 2026-03-16
author: Curtis Mercier
license: CC BY 4.0
extends: amp/0.3
complements: phase/0.1, seeds/0.1, maps/0.1
---

# SEAMS — Session Evolution Archival for Memory Systems v0.1

> Traceable connections between every artifact an agent produces. Pull any seam and the whole chain follows — from commit to phase, from phase to session, from session to decision.

*Extends: [AMP v0.3](../amp/) (Agent Memory Protocol)*
*Complements: [PHASE v0.1](../phase/), [SEEDS v0.1](../seeds/), [MAPS v0.1](../maps/)*

## 1. The Problem

An agent that works across sessions produces artifacts: commits, plans, muscles, decisions, impl-logs, preloads. These artifacts accumulate. After ten sessions, the question becomes: **what produced this?**

- Which session created this muscle?
- Which phase produced this commit?
- What decision led to this architecture?
- When did this plan change, and why?

Without traceability, every new session starts with archaeology. The agent digs through files trying to reconstruct context that was obvious to the agent that did the work. This wastes turns, burns context, and risks wrong assumptions.

SEAMS makes every artifact traceable. Every file knows its origin. Every commit knows its phase. Every decision knows its reasoning. The chain is walkable — backward from any point to the seed that started it.

## 2. The Origin Format

Every artifact produced by an agent carries an origin marker:

```yaml
origin: s01-fe82c9 @ 2026-03-16T05:30-04:00
```

The format: `<session-id> @ <ISO-8601-with-timezone>`

### 2.1 The Session Hash

The session ID (`s01-fe82c9`) is the **canonical token** — the seed from which traceability grows. It appears in:

- Session log filenames (`2026-03-16-s01-fe82c9.md`)
- Preload filenames (`preload-next-2026-03-16-s01-fe82c9.md`)
- Origin fields on every created artifact
- Impl-log entries
- Phase metadata (`sessions: [s01-fe82c9, s01-fe82c9-2]`)

The hash is short enough to be human-readable, unique enough to be unambiguous, and stable across the session chain (continuations append: `s01-fe82c9-2`, `s01-fe82c9-3`).

Like a Minecraft world seed, one hash generates an entire world of traceable artifacts. Given just the hash, a recall tool can reconstruct what happened, when, why, and what it produced.

### 2.2 Timestamp

ISO 8601 with timezone offset (`-04:00`, not `Z`) for:
- Unambiguous ordering across timezones
- Human readability (you can see "this was 5:30 AM Eastern")
- Sort stability (lexicographic sort matches chronological sort)

## 3. The Traceability Chain

```
seed (session hash)
  → session log (what happened)
    → phase (which unit of work)
      → impl-log (what was decided and why)
        → commits (what was produced)
          → artifacts (files created/modified)
            → origin field (traces back to seed)
```

Any artifact can answer: **"what produced you?"** — follow the origin to the session, the session to the phase, the phase to the plan. And: **"what did you produce?"** — follow the phase to its commits, the commits to their files.

### 3.1 Phase Metadata

Each phase tracks its seams:

```yaml
sessions: [s01-fe82c9, s01-fe82c9-2]
commits:
  agent: [28d71fe, 1039512, a12709e, df835cb]
  .soma: [12b9666, f012543]
origin: s01-fe82c9 @ 2026-03-16T03:09-04:00
```

### 3.2 Cross-Repo Tracking

When work spans multiple repositories, commits are tracked with repo prefixes:

```yaml
commits:
  agent: [28d71fe, 1039512]
  cli: [abc1234]
  .soma: [def5678]
```

The recall tool knows which repo to query for each prefix.

### 3.3 Impl-Log as Decision Trail

The impl-log is the richest seam. It captures not just WHAT happened but WHY:

```markdown
### Entry 3 — 2026-03-16 03:45 — Part A Complete

**What shipped:** `28d71fe` — PlanPromptConfig interface + 4 override points

**Design choices made:**
1. Optional 3rd param, not a new function — backward compatible
2. Force-include inserted after filtering — preserves sort order
3. Budget overrides — shallow merge, not deep

**What I watched for:** Every existing call site uses 2-arg calls.
Zero breaking changes confirmed.
```

Each entry is a seam connecting a commit to its reasoning. Months later, when someone asks "why is `getProtocolHeat()` taking 3 params?" — the impl-log has the answer.

## 4. Recall

SEAMS enables a recall tool that walks the chain:

```bash
soma-seam trace session s01-fe82c9          # everything this session produced
soma-seam trace phase soma-runtime/p0-plan   # full phase story
soma-seam trace commit agent:28d71fe         # which phase/session produced this
soma-seam trace file core/maps.ts            # when was this created, by whom, why
soma-seam trace time 2026-03-16T03:00/05:00  # everything that happened in this window
```

Each query walks the traceability chain and returns a structured answer:
- What MAP was active
- What prompt-config was used (which brain configuration)
- What sessions contributed
- What commits were produced
- What the impl-log says about decisions made
- What changed in downstream phases as a result

### 4.1 Trace by Hash

The most powerful query. Given just a session hash:

```bash
soma-seam trace hash fe82c9
```

Returns: every artifact with `fe82c9` in its origin, session references, or filename. This is how you reconstruct an entire chain of work from a single seed.

### 4.2 Trace by Time

Files track their `updated:` timestamp in frontmatter. Impl-log entries have timestamps. Git commits have timestamps. The recall tool can reconstruct "what happened between 3 AM and 5 AM" by querying all three sources.

## 5. Relationship to Other Protocols

```
  A M P S
      H
  M A P S
      S E A M S  ← you are here
    S E E D S
```

- **AMP** stores the files. SEAMS adds origin metadata to those files.
- **AMPS** defines content types. SEAMS traces when and why each piece of content was created or modified.
- **MAPS** provides navigation. SEAMS traces which MAP was active when work happened.
- **PHASE** configures the brain. SEAMS records which phase produced which artifacts.
- **SEEDS** grows structure forward. SEAMS traces structure backward.

SEAMS and [SEEDS](../seeds/) are complementary:
- **SEEDS** plants forward — templates, scaffolding, initialization
- **SEAMS** traces backward — connections, recall, history
- Both anchor on the same session hash

## 6. Anti-Patterns

- **Origin on every file** — don't add origin to third-party files, generated output, or ephemeral scratch. Origin is for artifacts the agent creates as part of tracked work.
- **Tracing without impl-logs** — commits are traceable via git, but the reasoning is lost. SEAMS without impl-logs gives you WHAT but not WHY.
- **Over-querying** — the recall tool is for reconstruction, not monitoring. Don't run traces every turn. Run them when you need to understand history.
- **Stale origins** — if you copy a file and modify it, update the origin. A copied origin traces to the wrong source.

## 7. Future Directions

### 7.1 Automated Seam Detection

The agent automatically adds `origin:` to files it creates. No manual step. The session hash is available in the runtime context; the agent stamps it on every artifact.

### 7.2 Visual Timeline

A timeline view of a project's evolution — phases on the x-axis, artifacts on the y-axis, seams as connections. Like a git graph but for the agent's entire memory, not just code.

### 7.3 Cross-Agent Tracing

When multiple agents contribute to the same project (parallel phases, child agents), their seams interweave. The recall tool can trace which agent produced which artifact and how they coordinated.

---

*SEAMS v0.1 — Curtis Mercier — CC BY 4.0*
*Extends: Agent Memory Protocol (AMP) v0.3*
*Complements: PHASE v0.1, SEEDS v0.1, MAPS v0.1*
*Reference implementation: Soma (soma.gravicity.ai)*
