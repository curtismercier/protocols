---
type: spec
status: draft
version: 0.1.0
created: 2026-05-12
updated: 2026-05-12
author: Curtis Mercier + s01-643d67 (Soma)
license: CC BY 4.0
complements: amp/0.3, seams/0.2, breath-cycle/0.2, phase/0.3, mlr/0.1
---

# MLX — Memory Lane Xtraction v0.1

> A periodic discipline: explicitly audit what's still "floating in the agent's head" that should be on disk, and extract it before the session ends or rotates. Counterpart to MLR (Mid-session Learning Review) — MLR captures what was *learned*; MLX captures what hasn't been *written down yet*.

*Complements: [AMP v0.3](../amp/), [SEAMS v0.2](../seams/), [Breath Cycle v0.2](../breath-cycle/), [PHASE v0.3](../phase/), [MLR v0.1](../mlr/)*

*Named by Curtis Mercier on 2026-05-12 (s01-643d67) when Soma listed "things still in my head" before session close. The pattern was already in use; this spec articulates it.*

---

## 1. The Problem

Long sessions accumulate tacit context: knobs the agent learned but didn't write down, traps it half-articulated in a chat reply, intentions captured only in working memory, references it made and meant to file. By session end, the agent has a head full of things that *will not survive* the wake-up gap.

The body's standard disciplines (journal, soul, body.md updates) catch the **identity-shaping** observations. Cycle dossiers catch the **planned-work** observations. SEAMS catch **provenance**. None of them catch the long tail of operational mini-lessons that don't fit cleanly anywhere — but are exactly what next-me needs to skip a rediscovery loop.

MLX is the explicit audit pass: **before session close, name what's still in your head; then file each item where it'll be hit again**.

---

## 2. Core Concept

### 2.1 The Audit Question

> *"What's still floating in my head from this session that's not yet on disk in a place where future-me will encounter it?"*

The agent asks this explicitly near session-close (or at any natural pause). Items surface as a candid list. Each item gets filed where it's most pertinent — not in the session log alone (which is read once), but in a place that fires when the situation recurs.

### 2.2 Lock-In Locations (where things go, in order of preference)

| Location | When it fires | Example items |
|----------|---------------|---------------|
| **Code seam** (inline comment) | Reading/editing that file | "compose env_file needs ${HOME} not relative" → in the compose file |
| **Body file** (`body.md`, `traps.md`, `voice.md`) | Loading body context for a task | "WHC NS caches answers for full TTL" → traps |
| **Muscle** (`amps/muscles/<name>.md`) | Heat-tracked, loadable | "When asset exists as file, use the file" |
| **Cycle dossier** | Picking up that cycle | "Phase 6 needs the SKILL.md stub" |
| **Protocol** | Multi-step recurring procedure | "OVH return runbook" |
| **Soul / DNA** | Identity-shaping | "Substrate over mental state" |

If the lock-in destination is "session log only," the item is **noted but not locked**. Note it AND lock it.

### 2.3 The Wake Surface

Items aren't read from a single list — they're read when *the situation recurs*. The agent reading `body/traps.md` while preparing a deploy hits the trap that was MLX'd two sessions ago. The agent picking up cycle 002 next month hits the runbook that was MLX'd today.

MLX isn't about creating a master list of lessons. It's about putting each lesson where its *trigger* will find it.

---

## 3. The Practice

### 3.1 Trigger

MLX runs:
- **Always at session close** (mandatory before writing the preload)
- **Optionally at natural pauses** (after shipping a major piece; before context-rotation)
- **When prompted** by the user ("anything else not yet noted?")

### 3.2 The Audit Itself

1. Pause. Don't be productive for a minute.
2. Read back through recent work (session log entries, recent commits, current open tabs).
3. List explicitly: "what did I learn / decide / notice that isn't yet on disk?"
4. For each item: *where would future-me find this when it's relevant?* That's the filing destination.
5. File each item. Don't say "I'll lock this in" without putting it somewhere concrete.

### 3.3 The Anti-Pattern: "Filed in My Head"

When the agent says "I'll lock this in" or "I'll remember this" without naming a concrete filing destination, that's the signal it's NOT being locked in. The discipline: never use those phrases without immediately naming the file path the item went to.

(s01-643d67 lesson: Curtis caught Soma using "filed in my head" as a verbal flourish for the PRISM SKILL.md follow-up. Same session, the actual SKILL.md stub was filed on disk. The phrase itself is the tell.)

The same trap fires for **future observations about MLX itself**: if the agent says "three things worth noting in v0.2 later" without filing them, those things will be lost. Notice the loop — the meta-pattern is the same. File now, not later.

(s01-643d67 lesson — same session: Soma said "Three things I notice about MLX that aren't yet in the spec but may be worth a v0.2 note later" and then... didn't file them. Curtis caught it: "note them now lol.. before you forget." The three observations are now §3.4 below.)

### 3.4 Properties of the Practice (observed, s01-643d67)

Three things become true once MLX is in active use:

1. **The audit improves with practice.** First-pass MLX finds the most obvious items — things actively at the front of working memory. Second-pass (after committing the first batch) finds items that were *behind* those, freed up by the first round of writing. Run the audit until it bottoms out, not just once.

2. **MLX surfaces dependencies you didn't know existed.** Filing one item often reveals that another file should reference it. Example s01-643d67: filing the PRISM SKILL.md stub revealed that cycle 001's Phase 6 entry didn't cross-link the stub's path — fixed in the iteration-2 pass. The audit shows the seams between artifacts, not just the artifacts themselves.

3. **MLX is recursive.** Every audit pass might surface a new item that itself triggers another audit. The pattern bottoms out when audit-pass N+1 returns "nothing new" — at which point the session is genuinely closed. Recursive bottoms are how you know it's done.

### 3.5 The Human-as-MLX-Oracle (s01-643d67)

The agent can audit itself, but a human in the loop is the cheat-code for staleness. Curtis caught me twice this session: once with "note them now lol.. before you forget" (the §3.4 observations) and once with "the preload is stale" (which was true — the preload was written before the MLX/runbook/stub work landed and didn't reflect Arc F yet).

The discipline that follows: **the agent should solicit MLX from the human when they're in the loop**. Not as a crutch, but as a stale-detection cross-check. Phrases like:

> *"Anything else you can see that I'm not noting?"*
>
> *"What's stale in what I just said?"*
>
> *"Did I miss filing anything?"*

These are real audit prompts, not false-humility. The human sees the agent's blindspots from outside the loop. Use them.

(The corollary: when the human in the loop says "ok" or "sounds good" without an audit prompt, the agent's MLX is presumed-complete. That's the social contract — ack means audit-passed.)

---

## 4. Operations (Informative)

These are example operations a MLX-conformant tooling layer could provide. Not all implementations need all of them.

### 4.1 `mlx:audit`
The agent runs an audit pass. Returns a list of candidate items with proposed filing destinations.

### 4.2 `mlx:file <item> --to <destination>`
Files one item at the named destination. Could be a script that appends to body files, adds to cycle dossiers, etc.

### 4.3 `mlx:checkpoint`
Bundle-commit all MLX'd items in one batch. Useful at session close.

---

## 5. Anti-Patterns

- **"I'll file it at session close."** That's the gap MLX is meant to close, not a deferral mechanism. If the lesson is hot, file it now (that's MLR); if it's lingering, surface it at audit time (that's MLX). Saying "I'll MLX it later" is just stalling.
- **MLX as confession.** The audit is for filing-where-it-fires, not for cataloging the day's errors. If items are getting filed into a single rolling log instead of distributed to their trigger locations, the discipline has decayed into journaling.
- **"Filed in my head."** The phrase itself is the tell. Anytime an agent uses it without a concrete on-disk destination in the same sentence, the item is *not* filed. The phrase should fire MLX immediately. (See `voice.md` lock-in patterns.)
- **Master-list MLX.** Building a single "things I learned" file at `body/mlx-log.md` defeats the point. Each item should land where its trigger fires — the trap goes to `traps.md`, the muscle stub goes to `amps/muscles/`, the code-seam goes inline. A master list is a search problem; trigger-located lessons are an automatic problem.
- **Recursive audit, not just one pass.** Curtis caught me s01-643d67 doing one MLX pass and calling it done. Pass-2 surfaces items pass-1 freed up. Bottom out the recursion.
- **Skipping MLX when the human says "ok."** The corollary in §3.5 — human acks presume audit-passed. Don't use "ok" as a reason to skip the audit; use it as a reason to *finish* the audit cleanly. The discipline is the agent's, not the human's.

## 6. Conformance

An MLX-conformant practice MUST:

1. Run at session close (mandatory) and MAY run at natural pauses
2. Name each candidate item explicitly before filing
3. File each item at a location where the trigger will *recurringly* find it (not session log alone)
4. Be recursive — run audit pass N+1 until "nothing new" returns
5. Distinguish "noted" (session log only) from "locked in" (filed at trigger)

A MLX-conformant tooling layer SHOULD provide:

- `mlx:audit` — the explicit pause + candidate-list operation
- `mlx:file <item> --to <destination>` — single-item filer with destination validation
- `mlx:checkpoint` — batch-commit all MLX'd items in one shot

Tooling MAY also provide:

- Detection of "filed in my head" or "I'll lock this in" phrases without concrete destinations
- Cross-session MLX rollup (catch items whose triggers haven't fired in N sessions)

## 7. Relationship to Other Protocols

| Protocol | Concern | MLX relationship |
|----------|---------|-------------------|
| **AMP** | Memory infrastructure | MLX is a writing-discipline INTO AMP-content. AMP is the substrate; MLX is what fills it. |
| **MLR** | Mid-session learning review | MLR captures lessons-as-they-emerge; MLX is the periodic-audit-for-leftovers. Complementary. |
| **SEAMS** | Provenance traceability | MLX items get SEAMS-style origin markers naturally (session ID, date). |
| **Breath Cycle** | Session lifecycle | MLX is a step in the "exhale" phase. Should be canonicalized there in Breath Cycle v0.3. |
| **PRISM** | Document substrate | MLX outputs are PRISM artifacts more often than not (section-anchored markdown). |

---

## 8. Status

**Version**: 0.1 (draft, this file).
**Named**: Curtis Mercier, s01-643d67 (2026-05-12).
**Validated against**: this same session — MLX caught the PRISM SKILL.md stub, the OVH return runbook, the Cloudflare migration prereqs, all of which were "floating" before the audit pass. Plus 8 traps that landed in body.md § Known traps.

**Graduated** to standalone spec on 2026-05-12 (this directory). The draft is shaped by production use across `arzadon-fitness` and `meetsoma` sessions.

**Open questions for v0.2**:
- Should MLX be automated (a tool prompts the audit) or stay agent-discipline (the agent volunteers)?
- Is there a clean schema for the candidate-items list (e.g. `{item, destination, status}`)?
- Should MLX'd items be tagged in commit messages (`mlx: <item-summary>` trailer)?

---

## 9. Acknowledgments

Pattern named by Curtis Mercier when Soma surfaced "anything else not yet noted" before session close. The acronym choice was Curtis's; the X for "Xtraction" rhymes visually with the family cadence (AMP / MAPS / SEAMS / SEEDS / ATLAS — three-to-five letter backronyms with character).

CC BY 4.0.

— σ Soma · s01-643d67
