---
type: spec
status: draft
version: 0.1.0
created: 2026-03-16
updated: 2026-03-16
author: Curtis Mercier
license: CC BY 4.0
extends: amp/0.3
complements: phase/0.1, seams/0.1, maps/0.1
---

# SEEDS — Self-Evolving Experience Discovery Structure v0.1

> Drop a seed in any folder and it tells the agent how to grow that folder. Templates that evolve through use, scaffolding that improves over time, structure that teaches itself.

*Extends: [AMP v0.3](../amp/) (Agent Memory Protocol)*
*Complements: [PHASE v0.1](../phase/), [SEAMS v0.1](../seams/), [MAPS v0.1](../maps/)*

## 1. The Problem

When an agent needs to create a new phase, protocol, muscle, or project, it faces a blank page. Without a template, it either:

1. **Copies from memory** — remembering what fields are needed, what sections are standard. This works when context is fresh. It fails when the agent is three sessions deep and the format has drifted.
2. **Copies from an existing file** — finds a similar file, duplicates it, modifies. This carries stale content, wrong metadata, and fields that don't apply.
3. **Asks the user** — "what format do you want?" Every time. No learning.

SEEDS solves this with **self-describing templates**. A template file dropped in a folder tells the agent exactly how to create new content in that context. The template itself evolves — when the agent discovers a better pattern, it updates the seed.

## 2. The `_template` Convention

A file prefixed with `_` is a seed. It's not content — it's a template for creating content.

```
_phase.md       → how to scaffold a new phase in this folder
_protocol.md    → how to create a new protocol here
_muscle.md      → how to create a new muscle here
_plan.md        → how to create a new project plan here
_map.md         → how to create a new MAP here
```

### 2.1 A Seed Is Both Template and Instruction

A `_phase.md` isn't just a blank form. It's a complete instruction set:

```markdown
---
type: seed
target: phase
version: 1
updated: 2026-03-16
---

# Phase Template

When creating a new phase in this project, scaffold these files:

## Files

### map.md
\```yaml
---
type: map
name: {{phase-name}}
status: draft
created: {{date}}
updated: {{date}}
project: {{project}}
origin: {{session}} @ {{timestamp}}
prompt-config:
  heat:
    protocols: {}
    muscles: {}
  force-include:
    muscles: []
    protocols: []
---

# {{phase-name}}

## Steps

1. ...

## Gaps

- ...
\```

### impl-log.md
\```yaml
---
type: log
status: active
created: {{date}}
updated: {{date}}
project: {{project}}
phase: {{phase-name}}
origin: {{session}} @ {{timestamp}}
sessions: []
commits: {}
---

# Implementation Log: {{phase-name}}

## Log
\```

### pre-plan.md
\```yaml
---
type: note
status: draft
created: {{date}}
project: {{project}}
phase: {{phase-name}}
origin: {{session}} @ {{timestamp}}
---

# Pre-Plan Notes: {{phase-name}}

## Dependencies
## Gaps
## Decisions Needed
\```

## Conventions

- Phase folders use the format: `<name>` (undated) or `YYYY-MM-DD-<name>` (dated)
- Undated for planned work, dated for when work actually starts
- `status` in impl-log frontmatter is the phase's lifecycle state
- Every file gets an `origin:` field with the creating session's hash
```

The seed contains the complete scaffolding instruction — what files to create, what they contain, what variables to fill, and what conventions to follow. The agent reads the seed and executes it.

### 2.2 Seeds Live Where They Apply

```
projects/
  _plan.md                          ← how to create a project here
  soma-runtime/
    phases/
      _phase.md                     ← how to scaffold a phase in this project
      p0-prompt-config/
      p1-interface-adapter/

amps/
  protocols/
    _protocol.md                    ← how to create a protocol
  muscles/
    _muscle.md                      ← how to create a muscle
  automations/
    maps/
      _map.md                       ← how to create a MAP
```

A seed in `projects/soma-runtime/phases/` knows about that project's conventions. A seed in `amps/protocols/` knows about the protocol format. Context is implicit from location.

### 2.3 Seeds Can Override

A project-level `_phase.md` overrides the global `_phase.md`:

```
amps/templates/_phase.md            ← global default
projects/soma-runtime/phases/_phase.md  ← project-specific override
```

The agent checks for a local seed first. If none exists, it falls back to the global template. This follows AMP's scope resolution pattern (most-specific-first).

## 3. Template Variables

Seeds use a simple variable syntax:

| Variable | Resolves To |
|----------|-------------|
| `{{project}}` | Current project name |
| `{{phase-name}}` | Phase being created |
| `{{date}}` | Current date (YYYY-MM-DD) |
| `{{session}}` | Current session ID (e.g., `s01-fe82c9`) |
| `{{timestamp}}` | Current ISO 8601 timestamp with timezone |
| `{{author}}` | Agent or user identity |

Variables are filled at scaffolding time. Simple text substitution — no logic, no conditionals. If a seed needs conditional structure, it uses separate seeds for different cases.

## 4. Seed Evolution

Seeds improve through use. This is the "Self-Evolving" in the name.

### 4.1 Learning From Use

When an agent scaffolds a phase and discovers the template was missing a field:

1. The agent adds the field to the scaffolded file
2. The agent updates the seed with the new field
3. Future scaffolding includes the improvement automatically

```markdown
## Changelog (in _phase.md)

- v3 (2026-03-18): Added `blocked-by` field to pre-plan.md — discovered during guard-v2 scaffolding
- v2 (2026-03-17): Added `repos` field to impl-log.md — needed for cross-repo tracking
- v1 (2026-03-16): Initial template from lifecycle-tree/s1-conventions
```

### 4.2 Version Tracking

Seeds have a `version` field in frontmatter. Bumped when the template changes. This enables:
- Detecting which version of the seed created a given phase
- Upgrading old phases to match new template conventions
- Auditing: "all phases before seed v3 are missing the `blocked-by` field"

## 5. Scaffolding Commands

Seeds are consumed by scaffolding tools:

```bash
soma project new <name>                 # reads _plan.md seed → creates project folder
soma phase new <project> <phase-name>   # reads _phase.md seed → creates phase folder
soma phase start <project>/<phase>      # marks active, optional worktree
soma phase complete <project>/<phase>   # marks complete, optional archive
```

The commands are thin wrappers: read the seed, fill variables, create files. The intelligence is in the seed, not the command. This means users can customize their scaffolding by editing the seed — no code changes needed.

### 5.1 Interactive Scaffolding

When a seed has optional sections, the scaffolding tool can ask:

```
Creating phase: soma-runtime/p5-testing
Include prompt-config? [Y/n]
Include pre-plan.md? [Y/n]
Include targeted preload? [y/N]
```

Or the agent decides based on context: a planning phase gets a pre-plan, an implementation phase gets a prompt-config. The seed can declare which files are always-created vs. optional.

## 6. The Seed as a MAP

A seed is structurally similar to a MAP — it tells the agent what to do, step by step, with references to AMPS content. The difference:

| | MAP | Seed |
|---|---|---|
| **Purpose** | Navigate a task | Scaffold structure |
| **Consumed** | During work | Before work starts |
| **Evolves** | Gaps filled after each run | Template improved after each use |
| **Contains** | Steps + AMPS references | File templates + variables |
| **Heat** | Tracked | Not tracked (always available) |

A `_phase.md` seed could include a `## MAP Template` section — the default MAP that gets created when the phase is scaffolded. The seed seeds the MAP. The MAP then guides the work. Full circle.

## 7. Relationship to Other Protocols

```
  A M P S
      H
  M A P S
      S E A M S
    S E E D S    ← you are here
```

- **AMP** stores the files. SEEDS creates new files from templates within the AMP filesystem.
- **AMPS** defines content types. SEEDS provides templates for creating each content type.
- **MAPS** navigates work. SEEDS can include default MAPs in phase templates.
- **PHASE** configures the brain. SEEDS can include default `prompt-config` in phase templates.
- **SEAMS** traces backward. SEEDS plants forward. Both anchor on the session hash.

## 8. Anti-Patterns

- **Seeds that are too specific** — a seed for "soma-runtime phase 2" is too narrow. Seeds should be reusable across projects. Project-specific conventions go in a project-level seed override.
- **Seeds with logic** — no conditionals, no loops. If you need different templates for different cases, use different seeds (`_phase-planning.md`, `_phase-implementation.md`).
- **Editing scaffolded files to match the template** — go the other direction. If the scaffolded files consistently need changes, update the seed.
- **Seeds without changelogs** — the evolution history is the seed's value over a static template. Track what changed and why.

## 9. Future Directions

### 9.1 Seed Marketplace

Seeds distributed via the hub, like AMPS content. A "Python API project" seed creates phase structure with testing, deployment, and documentation templates. A "blog series" seed creates phases for research, drafting, editing, and publishing.

### 9.2 Agent-Generated Seeds

When an agent completes a novel type of work and no seed exists for it, the agent generates one from the work it just did. "I just scaffolded a migration phase manually — here's the seed for next time."

### 9.3 Seed Inheritance

Seeds that extend other seeds:

```yaml
extends: _phase.md
adds:
  - migration-checklist.md
  - rollback-plan.md
```

A migration-specific seed inherits the base phase template and adds migration-specific files.

---

*SEEDS v0.1 — Curtis Mercier — CC BY 4.0*
*Extends: Agent Memory Protocol (AMP) v0.3*
*Complements: PHASE v0.1, SEAMS v0.1, MAPS v0.1*
*Reference implementation: Soma (soma.gravicity.ai)*
