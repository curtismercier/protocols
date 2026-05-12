---
type: spec
status: draft
version: 0.2.0
created: 2026-03-10
updated: 2026-05-12
author: Curtis Mercier
license: CC BY 4.0
complements: amp/0.3, seams/0.2, seeds/0.2, phase/0.3
---

# ATLAS — Architecture Truth Layered Across Stacks

> A documentation protocol for maintaining living system maps that stay accurate because they're the primary reference, not an afterthought.

## 1. The Problem

Architecture docs rot. Everyone knows this. They rot because:
- They're written *about* the system, separate from the system
- Nobody updates them because they're not in the critical path
- When they're wrong, people stop reading them — a death spiral

ATLAS solves this by making the architecture doc the **primary reference** — the first thing an agent reads on boot, the first thing a human checks when orienting.

## 2. Principles

1. **The doc IS the system's self-knowledge.** Not documentation *about* the system.
2. **Updated in the same commit.** Architecture changes → doc changes. Same PR.
3. **One per scope.** One ATLAS doc per: ecosystem, product, component. No duplicates.
4. **Hierarchical.** Parent ATLAS wins on cross-cutting concerns. Child wins on local details.
5. **Machine-readable frontmatter.** Every doc has standard metadata for searchability.

## 3. Frontmatter

Every ATLAS-participating document MUST have:

```yaml
---
type: <type>
status: <status>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 3.1 Required Fields

| Field | Purpose |
|-------|---------|
| `type` | What kind of document (see §3.2) |
| `status` | Lifecycle state (see §3.3) |
| `created` | When first written |
| `updated` | When last meaningfully changed |

### 3.2 Core Type Values

| Value | Meaning |
|-------|---------|
| `state` | ATLAS architecture truth doc (STATE.md) |
| `plan` | Something intended to be built/done |
| `spec` | A specification or standard |
| `note` | General reference document |
| `index` | Directory listing / navigation |
| `log` | Chronological record |

Implementations MAY extend this list. The [AMPS](../amps/) protocol adds `muscle`, `protocol`, `map`, and others. ATLAS does not prescribe the full taxonomy — it prescribes the mechanism.

### 3.3 Status Lifecycle

```
seed → draft → active → complete
                  ↓
               stale → archived
```

| Value | Meaning |
|-------|---------|
| `seed` | Idea planted, minimal content |
| `draft` | Being written/designed, not yet reliable |
| `active` | Current, maintained, reliable — the truth |
| `complete` | Done, no more work needed |
| `stale` | Was active, hasn't been updated, may be wrong |
| `archived` | No longer relevant, kept for history |
| `blocked` | Can't progress without something else |
| `paused` | Intentionally stopped, will resume |

**Rules:**
- `state` docs should always be `active`. If stale, something's wrong.
- `plan` docs move: `seed → draft → active → complete`
- `updated` changes with every meaningful content edit

### 3.4 Optional Fields

```yaml
tags: [<searchable keywords>]
project: <project name>
author: <who wrote it>
depends: [<what blocks this>]
rule: <when to update this file>
scope: <local|shared>
```

See the reference implementation's frontmatter standard for a complete taxonomy of optional fields suited to agent memory systems.

## 4. The STATE.md File

The core ATLAS artifact. One per scope.

### 4.1 Naming Convention

Always `STATE.md`. Not `ARCHITECTURE.md`, not `SYSTEM.md`, not `README.md` (which serves a different purpose). The name is the convention.

### 4.2 Template

```markdown
---
type: state
method: atlas
project: <name>
updated: <YYYY-MM-DD>
status: active
rule: <when to update this file>
---

# <Project> — Architecture State

> ATLAS — single source of truth for this scope.

## What This Is
<2-3 sentences. What it does, who it's for.>

## System Map
<ASCII art showing components and relationships.>

## Components
| Component | What | Status | Location |
|-----------|------|--------|----------|

## Decisions
| Decision | Date | Rationale |
|----------|------|-----------|

## Open Questions
-
```

### 4.3 The `rule` Field

Each STATE.md declares its own update trigger:

```yaml
rule: Update this file whenever the architecture changes.
rule: Update when repos, infrastructure, or cross-repo relationships change.
rule: Update when components are added, removed, or their relationships change.
```

This makes the update discipline explicit and self-documenting. Every agent that reads the file knows what events should trigger an update — without needing external documentation.

## 5. Hierarchy

```
ecosystem/STATE.md          ← cross-cutting decisions (wins on conflicts)
  └── product/STATE.md      ← product-level architecture
       └── component/STATE.md  ← component internals
```

### 5.1 Conflict Resolution

- Parent ATLAS wins on **cross-cutting concerns** (tech choices, conventions, standards)
- Child ATLAS wins on **local implementation details** (internal structure, component-specific patterns)
- When in doubt, check the parent

### 5.2 Example

```
ecosystem/STATE.md:    "All projects use pnpm"     ← authoritative
product/STATE.md:      "This product uses Astro"    ← authoritative for this product
component/STATE.md:    "This page uses React islands" ← authoritative for this component
```

If `component/STATE.md` said "we use npm" — that conflicts with the ecosystem-level decision. The parent wins: use pnpm.

## 6. Update Discipline

### 6.1 MUST Update When

- A component is added or removed
- A technology choice changes
- A relationship between components changes
- A decision is made that affects architecture
- Infrastructure changes (hosting, DNS, CI)

### 6.2 MUST NOT Contain

- Tutorials or how-to guides (those go in docs/)
- Changelog entries (CHANGELOG.md)
- Task tracking (kanban, issues)
- Secrets or credentials (vault, .env)
- Aspirational features not yet built (those are plans, not state)

### 6.3 Staleness Signals

The `updated` field is a staleness indicator. Implementations SHOULD define thresholds:

| Age | Signal | Recommended action |
|-----|--------|-------------------|
| ≤3 days | Fresh | No action needed |
| 3–7 days | Warning | Verify before starting cross-scope work |
| >7 days | Stale | Full ATLAS audit before trusting content |

These thresholds are guidelines, not rules — a stable system may go weeks without architectural changes. The signal is the gap between `updated` and the last structural change, not `updated` and today.

Automation helps: build tools that warn when `updated` exceeds the threshold. The discipline decays without reminders.

### 6.4 Verification

After a structural change, verify the STATE.md matches reality:

**After a component change:**
- Component table matches the actual filesystem
- Relationships in the system map are current
- Status column reflects actual state

**After a release or deployment:**
- Version numbers match `package.json` / build output
- Deployment paths match actual infrastructure

**Quick check (any scope):**
```bash
grep "^updated:" STATE.md      # Is the date recent?
grep "^status:" STATE.md       # Is it still active?
```

Implementations MAY add scope-specific checklists (e.g., "verify branch list matches `git branch -a`"). The principle is: **the doc claims to be truth — verify the claim**.

## 7. Inline Maintenance Markers

STATE.md sections that track volatile state (branch lists, version numbers, component tables) benefit from inline markers that tell the reader when to update that specific section:

```html
## Components
<!-- UPDATE WHEN: components added, removed, or status changes -->
<!-- VERIFY: compare table to actual filesystem -->
```

These markers serve a different purpose than frontmatter — they're contextual (per-section) rather than per-document, and actionable (tell you HOW to verify, not just THAT you should).

The `UPDATE WHEN` marker defines the trigger. The `VERIFY` marker defines the check. Together they make maintenance instructions local to the content they describe.

In multi-agent systems, a `WHO UPDATES` marker can assign responsibility:

```html
<!-- WHO UPDATES: any agent after structural changes. Verify with: ls src/ -->
```

This is especially valuable when multiple agents maintain the same STATE.md — without it, every agent assumes someone else will update.

## 8. Companion Documents

A STATE.md that tries to be comprehensive becomes too large for quick orientation. A STATE.md that's too brief doesn't orient effectively. The two-document pattern solves this:

| Document | Role | Size | Loading |
|----------|------|------|---------|
| **STATE.md** | Health snapshot — branches, versions, known issues, quick verify | Small (<50 lines) | Always loaded (eager) |
| **Companion index** | Full inventory — file map, architecture diagram, data flow, known issues detail | Large (100–200 lines) | Loaded on demand (lazy) |

STATE.md is the dashboard. The companion index is the manual. Both follow ATLAS principles (frontmatter, update discipline, hierarchy). The companion references the STATE.md and vice versa.

This pattern is optional — small scopes may only need STATE.md. Introduce the companion when the STATE.md grows past ~50 lines or when a codebase index would save repeated exploration.

## 9. Searchability

With standard frontmatter, finding things is grep:

```bash
# All architecture docs
grep -rl "^type: state" .

# Everything that needs review (stale)
grep -rl "^status: stale" .

# All docs updated in the last week
grep -rl "^updated: 2026-03-2" .

# High-priority items
grep -rl "^priority: high" .
```

For agents: scan frontmatter at boot to know what's in progress, what's blocked, what's stale. The frontmatter IS the project management layer.

## 10. For Implementors

### Introducing ATLAS to an Existing Project

1. Create `STATE.md` in the project root
2. Write the system map (ASCII art — doesn't need to be perfect)
3. List components with status
4. Set the `rule:` field (what triggers updates)
5. Set `updated:` to today
6. Read it on every session start — this is how the habit forms

### First ATLAS Audit

When adopting ATLAS on a project with existing docs:
1. Create STATE.md as described above
2. Read existing docs and note what's stale
3. Move architectural claims from README/docs into STATE.md
4. Mark the old docs with notes pointing to STATE.md
5. Delete architectural claims that are duplicated (don't maintain two truths)

### Multi-Agent Systems

When multiple agents work on the same system:
- Each agent reads STATE.md on boot
- Each agent updates STATE.md when it makes structural changes
- Inline maintenance markers (`UPDATE WHEN`, `WHO UPDATES`) prevent "someone else will do it" drift
- The hierarchy resolves conflicts — parent scope wins cross-cutting

### Automation

ATLAS is a discipline, not a dependency — no tooling required. But discipline decays:
- Build scripts can warn when `updated` exceeds staleness thresholds
- CI can check that structural changes include STATE.md edits
- Agents can verify STATE.md on session start (pre-flight check)

## 11. Relationship to Other Protocols

- **[AMP](../amp/)** — ATLAS uses AMP's frontmatter convention. STATE.md is a first-class AMP document type.
- **[AMPS](../amps/)** — ATLAS documents may describe the AMPS content loaded per scope.
- **[MAPS](../maps/)** — Each MAP phase may reference different ATLAS docs depending on scope.
- **[PHASE](../phase/)** — When a PHASE configures an agent, it should load the scope-appropriate STATE.md.
- **[SEAMS](../seams/)** — STATE.md updates should carry session seams for traceability. Inline maintenance markers complement session seams with structural navigation.
- **[SEEDS](../seeds/)** — A `_STATE.md` seed template scaffolds new scopes with the ATLAS structure.

## 12. Attribution

```
This project uses the ATLAS method
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*ATLAS v0.2 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*

### Changelog

**v0.2.0** (2026-03-28)
- Added §6.3 Staleness Signals — thresholds for when to verify vs when to audit
- Added §6.4 Verification — post-change checklists and quick checks
- Added §7 Inline Maintenance Markers — per-section `UPDATE WHEN`, `VERIFY`, `WHO UPDATES`
- Added §8 Companion Documents — two-file pattern (STATE.md + companion index)
- Added §10 For Implementors — concrete guidance on adoption, first audit, multi-agent, automation
- Revised §3 Frontmatter — trimmed to core fields, implementations extend the taxonomy
- Revised §11 Relationship to Other Protocols — expanded with specific connection points
- Principles, hierarchy, and core template unchanged

**v0.1.0** (2026-03-10)
- Initial specification
