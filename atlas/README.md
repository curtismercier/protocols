---
type: spec
status: draft
version: 0.1.0
created: 2026-03-10
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
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

## 3. Frontmatter Standard

Every ATLAS-participating document MUST have:

```yaml
---
type: <type>
status: <status>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 3.1 Type Values

| Value | Meaning |
|-------|---------|
| `state` | ATLAS architecture truth doc (STATE.md) |
| `plan` | Something intended to be built/done |
| `spec` | A specification or standard |
| `muscle` | A learned pattern (see AMP spec) |
| `preload` | Session continuation state (see AMP spec) |
| `identity` | Agent identity definition |
| `note` | General reference document |
| `index` | Directory listing / navigation |
| `concept` | Early-stage idea exploration |
| `ritual` | Workflow definition |
| `strategy` | Business, IP, or technical strategy |
| `log` | Chronological record |

### 3.2 Status Lifecycle

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
- `state` (ATLAS) docs should always be `active`. If stale, something's wrong.
- `plan` docs move: `seed → draft → active → complete`
- `muscle` docs are always `active` or `archived`
- `updated` changes with every meaningful content edit

### 3.3 Optional Fields

```yaml
tags: [<searchable keywords>]
project: <project name>
author: <who wrote it>
depends: [<what blocks this>]
blocks: [<what this blocks>]
priority: <high|medium|low>
reviewed: <YYYY-MM-DD>
scope: <local|shared>
```

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

This makes the update discipline explicit and self-documenting.

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

### 6.3 The `updated` Discipline

> Update `updated:` in the same edit as any meaningful content change.

"Meaningful" = content changed in a way that affects understanding. Not typo fixes or reformatting.

## 7. Searchability

With standard frontmatter, finding things is grep:

```bash
# All unfinished plans
grep -rl "^status: draft" .

# All architecture docs
grep -rl "^type: state" .

# Everything that needs review (stale)
grep -rl "^status: stale" .

# All docs updated in the last week
grep -rl "^updated: 2026-03-1" .

# High-priority items
grep -rl "^priority: high" .
```

For agents: scan frontmatter at boot to know what's in progress, what's blocked, what's stale. The frontmatter IS the project management layer.

## 8. Implementation Notes

- ATLAS works with any documentation format that supports frontmatter (Markdown + YAML is the default)
- No tooling required — it's a discipline, not a dependency
- Agents that implement ATLAS should read STATE.md on boot to orient — after identity and [PHASE](../phase/) configuration, before starting work
- The frontmatter standard is shared with the AMP protocol (muscles, preloads use the same fields)
- In multi-phase work ([MAPS](../maps/)), each phase may reference different ATLAS docs depending on scope

## 9. Attribution

```
This project uses the ATLAS method
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*ATLAS v0.1 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
