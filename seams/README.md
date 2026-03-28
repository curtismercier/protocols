---
type: spec
status: draft
version: 0.2.0
created: 2026-03-16
updated: 2026-03-28
author: Curtis Mercier
license: CC BY 4.0
extends: amp/0.3
complements: phase/0.1, seeds/0.1, maps/0.1, atlas/0.2
---

# SEAMS — Session Evolution Archival for Memory Systems v0.2

> Traceable connections between every artifact an agent produces. Two dimensions: **session seams** trace time (what produced this?), **document seams** trace space (what connects to this?). Pull any seam and the whole chain follows.

*Extends: [AMP v0.3](../amp/) (Agent Memory Protocol)*
*Complements: [PHASE v0.1](../phase/), [SEEDS v0.1](../seeds/), [MAPS v0.1](../maps/), [ATLAS v0.2](../atlas/)*

## 1. The Problem

An agent that works across sessions produces artifacts: commits, plans, muscles, decisions, impl-logs, preloads. These artifacts accumulate. Two questions emerge:

**"What produced this?"** — temporal traceability. Which session created this muscle? Which phase produced this commit? What decision led to this architecture?

**"What connects to this?"** — structural traceability. If I update this STATE.md section, what else breaks? If I read this plan, where's the implementation? If this knowledge doc goes stale, what triggers a refresh?

Without the first, every session starts with archaeology. Without the second, every maintenance task starts with detective work. SEAMS solves both.

## 2. Session Seams (Temporal Traceability)

Session seams answer: **"what produced this, when, and why?"**

### 2.1 The Origin Format

Every artifact produced by an agent carries an origin marker:

```yaml
origin: s01-fe82c9 @ 2026-03-16T05:30-04:00
```

The format: `<session-id> @ <ISO-8601-with-timezone>`

### 2.2 The Session Hash

The session ID (`s01-fe82c9`) is the **canonical token** — the seed from which traceability grows. It appears in:

- Session log filenames (`2026-03-16-s01-fe82c9.md`)
- Preload filenames (`preload-next-2026-03-16-s01-fe82c9.md`)
- Origin fields on every created artifact
- Impl-log entries
- Phase metadata (`sessions: [s01-fe82c9, s01-fe82c9-2]`)

The hash is short enough to be human-readable, unique enough to be unambiguous, and stable across the session chain (continuations append: `s01-fe82c9-2`, `s01-fe82c9-3`).

Like a Minecraft world seed, one hash generates an entire world of traceable artifacts. Given just the hash, a recall tool can reconstruct what happened, when, why, and what it produced.

### 2.3 Timestamp

ISO 8601 with timezone offset (`-04:00`, not `Z`) for:
- Unambiguous ordering across timezones
- Human readability (you can see "this was 5:30 AM Eastern")
- Sort stability (lexicographic sort matches chronological sort)

### 2.4 The Traceability Chain

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

### 2.5 Phase Metadata

Each phase tracks its seams:

```yaml
sessions: [s01-fe82c9, s01-fe82c9-2]
commits:
  agent: [28d71fe, 1039512, a12709e, df835cb]
  .soma: [12b9666, f012543]
origin: s01-fe82c9 @ 2026-03-16T03:09-04:00
```

Cross-repo tracking uses repo prefixes:

```yaml
commits:
  agent: [28d71fe, 1039512]
  cli: [abc1234]
  .soma: [def5678]
```

### 2.6 Impl-Log as Decision Trail

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

### 2.7 Co-occurrence Seams

When multiple artifacts are created in the same session, they share a seam hash. The `seams:` frontmatter field records this co-occurrence:

```yaml
seams: [s01-fe82c9, s01-3498d3]
```

Two documents sharing a seam means "these ideas were in the same mind at the same time." This is distinct from `origin:` (which session created this file) — `seams:` tracks which sessions touched or evolved it.

## 3. Document Seams (Structural Traceability)

Document seams answer: **"what connects to this, when should it be updated, and who's responsible?"**

Where session seams trace backward through time, document seams trace sideways through structure. They're inline HTML comments placed in the documents themselves — visible to any agent reading the file, contextual to the section they describe.

### 3.1 The Three Comment Types

#### Connection Seams

```html
<!-- SEAMS: → path/to/related-file.md (why it connects)
            → another-file.md (what this section feeds into)
            ← source-file.md (where this content came from) -->
```

**Direction arrows:**
- `→` this document feeds INTO that one (downstream dependency)
- `←` that document feeds INTO this one (upstream source)
- `↔` bidirectional dependency (both reference each other)

Place at the top of a file for file-level connections, or above a specific section for section-level connections.

#### Staleness Triggers

```html
<!-- UPDATE WHEN: branches created/deleted, files added/removed -->
```

Defines the event that should trigger an update to this section. Without this, agents don't know if a section is stale or just stable. The trigger makes implicit maintenance knowledge explicit and local.

#### Responsibility Assignment

```html
<!-- WHO UPDATES: any agent after structural changes. Verify: git branch -a -->
```

Names the responsible agent or role, plus a verification command. In multi-agent systems, this prevents "someone else will update it" drift.

### 3.2 Where to Apply

Document seams are for **documents that go stale** or **documents that form a navigation network**:

| Document type | Apply? | Why |
|--------------|--------|-----|
| STATE.md | Yes | Volatile sections need staleness triggers |
| Plans | Yes | Connect to the kanban items they implement |
| Knowledge docs | Yes | Go stale when the system they describe changes |
| Inbox messages | Yes | Connect to related messages and plans |
| Indexes | Yes | Reference volatile content |
| Source code | No | Use code comments and imports |
| AMPS content (muscles, protocols) | Prefer frontmatter | `related:`, `depends-on:`, `tools:` fields |
| Templates | No | `{{variables}}` already define connections |

### 3.3 Combining Session and Document Seams

A document can carry both:

```yaml
---
origin: s01-a15343 @ 2026-03-28T06:00-04:00
seams: [s01-a15343, s01-fe82c9]
seeded-from: ideas/knowledge-protocol.md
---
<!-- SEAMS: → plans/project-lifecycle.md (detailed implementation)
            ← inbox/2026-03-28-seed-proposal.md (spawned this plan) -->
```

- `origin:` — which session created this (session seam)
- `seams:` — which sessions touched this (co-occurrence seam)
- `seeded-from:` — which artifact inspired this (see [SEEDS](../seeds/))
- `<!-- SEAMS: -->` — what this connects to right now (document seam)

Session seams are archaeology. Document seams are wayfinding. Together they answer: where did this come from, what does it connect to, and who should maintain it.

## 4. Multi-Agent Seams

When multiple agents contribute to the same project, seams become the coordination mechanism.

### 4.1 Origin Attribution

In multi-agent systems, `origin:` should include the agent:

```yaml
origin: s01-a15343 @ 2026-03-28T06:00-04:00
```

The session hash alone identifies the agent (each agent generates unique hashes). But for human readability, implementations MAY add agent context:

```yaml
seeded-by: soma @ s01-a15343
```

This answers: "which agent planted this seed, in which session?"

### 4.2 Cross-Agent Document Seams

Document seams in inbox messages create a navigable cross-agent conversation:

```html
<!-- SEAMS: → releases/v0.6.x/plans/lifecycle.md (detailed plan)
            ← inbox/2026-03-28-sage-layout-lessons.md (Sage's analysis) -->
```

An agent reading this message can follow the arrows to understand the full context without reading every related file. The seams ARE the reading order.

### 4.3 Responsibility Across Agents

```html
<!-- WHO UPDATES: Sage after infrastructure changes, Soma after framework changes -->
```

This prevents the common failure: multiple agents read a STATE.md, notice it's stale, but each assumes the other will update it. Explicit assignment fixes this.

## 5. Recall

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

### 5.1 Trace by Hash

The most powerful query. Given just a session hash:

```bash
soma-seam trace hash fe82c9
```

Returns: every artifact with `fe82c9` in its origin, session references, or filename. This is how you reconstruct an entire chain of work from a single seed.

### 5.2 Trace by Time

Files track their `updated:` timestamp in frontmatter. Impl-log entries have timestamps. Git commits have timestamps. The recall tool can reconstruct "what happened between 3 AM and 5 AM" by querying all three sources.

### 5.3 Trace by Connection (Document Seam Traversal)

```bash
soma-seam trace links STATE.md              # what does STATE.md connect to?
soma-seam trace upstream core/body.ts       # what feeds into this file?
soma-seam trace downstream plans/lifecycle  # what does this plan affect?
```

This is the structural equivalent of `trace hash` — instead of walking time, it walks connections.

## 6. Relationship to Other Protocols

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
- **ATLAS** maintains ground truth. Document seams on STATE.md sections enable targeted verification. Session seams on STATE.md updates provide audit trail.

SEAMS and [SEEDS](../seeds/) are complementary:
- **SEEDS** plants forward — templates, scaffolding, initialization
- **SEAMS** traces backward — connections, recall, history
- Both anchor on the same session hash

## 7. Anti-Patterns

### Session Seam Anti-Patterns

- **Origin on every file** — don't add origin to third-party files, generated output, or ephemeral scratch. Origin is for artifacts the agent creates as part of tracked work.
- **Tracing without impl-logs** — commits are traceable via git, but the reasoning is lost. SEAMS without impl-logs gives you WHAT but not WHY.
- **Over-querying** — the recall tool is for reconstruction, not monitoring. Don't run traces every turn. Run them when you need to understand history.
- **Stale origins** — if you copy a file and modify it, update the origin. A copied origin traces to the wrong source.
- **Drifted origin format** — use the canonical `s01-hash @ timestamp` format. Not descriptions, not agent names, not prose. The hash is greppable; prose is not.

### Document Seam Anti-Patterns

- **Seams on source code** — use code comments and imports, not SEAMS syntax. Document seams are for documents, not code.
- **Orphan seams** — a `<!-- SEAMS: → file.md -->` pointing to a deleted file. When you delete a file, grep for seams that reference it.
- **Seams without verification** — `<!-- UPDATE WHEN: X -->` is useful, but `<!-- VERIFY: command -->` is what makes it actionable. Include both when possible.
- **Over-seaming** — not every file needs document seams. A simple note doesn't need structural traceability. Apply seams to documents that go stale or form navigation networks.

## 8. Future Directions

### 8.1 Automated Seam Detection

The agent automatically adds `origin:` to files it creates. No manual step. The session hash is available in the runtime context; the agent stamps it on every artifact.

### 8.2 Visual Timeline

A timeline view of a project's evolution — phases on the x-axis, artifacts on the y-axis, seams as connections. Like a git graph but for the agent's entire memory, not just code.

### 8.3 Lifecycle Tracing

```bash
soma-seam lifecycle <term>
```

Traces a concept through its evolution stages: idea → plan → MAP → code → release. The `trace` command finds mentions; `lifecycle` shows progression through specific directories and status transitions.

---

*SEAMS v0.2 — Curtis Mercier — CC BY 4.0*
*Extends: Agent Memory Protocol (AMP) v0.3*
*Complements: PHASE v0.1, SEEDS v0.1, MAPS v0.1, ATLAS v0.2*
*Reference implementation: Soma (soma.gravicity.ai)*

### Changelog

**v0.2.0** (2026-03-28)
- Restructured into §2 Session Seams + §3 Document Seams + §4 Multi-Agent Seams
- Added §3 Document Seams — connection arrows (`→ ← ↔`), `UPDATE WHEN`, `VERIFY`, `WHO UPDATES`
- Added §2.7 Co-occurrence Seams — `seams:` frontmatter for session co-occurrence
- Added §3.3 Combining Session and Document Seams — how the three types compose
- Added §4 Multi-Agent Seams — origin attribution, cross-agent navigation, responsibility
- Added §5.3 Trace by Connection — structural traversal of document seams
- Added §7 Anti-Patterns for document seams (orphan seams, over-seaming)
- Added §8.3 Lifecycle Tracing — `soma-seam lifecycle` for concept evolution
- Expanded §1 The Problem — two dimensions (temporal + structural)
- Expanded §6 Relationship to ATLAS — document seams enable targeted STATE.md verification
- Moved §7.3 Cross-Agent Tracing from future directions into §4 (now implemented)

**v0.1.0** (2026-03-16)
- Initial specification — session seams, origin format, traceability chain, recall
