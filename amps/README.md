---
type: spec
status: draft
version: 1.1.0
created: 2026-03-11
updated: 2026-03-16
author: Curtis Mercier
license: CC BY 4.0
extends: amp/0.3
---

# AMPS — Agent Memory Protocol Stack v1.1

> Four content types that extend the Agent Memory Protocol. Automations, Muscles, Protocols, Scripts — the building blocks of an evolving agent.

*Extends: [AMP v0.3](../amp/) (Agent Memory Protocol)*
*Supersedes: [Agent Capability Model v0.2](../three-layer/) (archived)*

## 1. Overview

AMPS defines what lives inside an AMP-compatible memory system. Four content types, each with a different nature, authorship profile, and loading behavior — plus a runtime layer that executes them.

**AMPS** = **A**utomations · **M**uscles · **P**rotocols · **S**cripts

The name is deliberate: AMP is the memory engine, AMPS is what it amplifies.

## 2. The Four Content Types

| Type | Nature | Author | Loading |
|------|--------|--------|---------|
| **Automations** | Executable workflows | Workflow designer | Registered as commands |
| **Muscles** | Learned patterns from experience | Agent + user | By heat, digest-first |
| **Protocols** | Behavioral rules, enforced | System designer | By heat, full or breadcrumb |
| **Scripts** | Executable tools | Agent + dev | Run by agent or CI |

### 2.1 Automations (Orchestration Layer)

Executable workflows triggered by command, combining knowledge with structured execution. What was once a manual pattern becomes a repeatable action.

- Triggered by command (e.g., `/publish`, `/release`, `/audit`)
- Multi-step with explicit phases — conversational, each step may involve the user
- Can reference scripts, follow protocols, and depend on other automations
- Heat-tracked: frequently-used automations surface in suggestions, but they don't inject into the prompt
- Register as slash commands when installed

**Examples:** `/publish` (blog post workflow), `/release` (semantic version + changelog), `/audit` (frontmatter compliance check)

**Format:**
```yaml
---
type: automation
name: automation-name
status: active
heat-default: warm
command: /publish
breadcrumb: "One-liner..."
tier: community
tags: [workflow, publishing]
steps: 5
depends-on:
  protocols: [git-identity]
  muscles: []
  automations: []
  scripts: []
---
```

**Body structure:**
```markdown
# Automation Name

## TL;DR
What this does in 2-3 sentences.

## Steps
1. **Step name** — description. User input needed: yes/no.
2. **Step name** — description.
...

## When to Use
## Configuration
```

### 2.2 Muscles (Learning Layer)

Patterns the agent discovered through experience and encoded for future reuse. Named after biological muscle memory: the more you use it, the stronger it gets.

- Born from gaps — moments where friction was noticed, a pattern repeated, a workflow failed
- Grow through use: heat rises on application, decays on neglect
- Two-tier loading: digest (compressed) for tight context, full body when space allows
- The richest source of muscles is observing the *user's* patterns

**The key insight:** Muscles aren't instructions the user wrote. They're patterns the agent noticed and encoded.

**Examples:** "Always run tests after file removal", "User prefers concise responses", "This API returns paginated results"

**Format:**
```yaml
---
type: muscle
name: muscle-name
status: active
heat: 5
heat-default: warm
loads: 12
breadcrumb: "One-liner TL;DR..."
topic: [workflow, testing]
keywords: [tests, file-removal]
depends-on:
  protocols: []
  muscles: []
  automations: []
  scripts: []
---
```

**Body structure:**
```markdown
# Title

<!-- digest:start -->
Compressed essential knowledge. 2-5 sentences.
<!-- digest:end -->

Full content: detailed instructions, examples, edge cases.
```

### 2.3 Protocols (Instinct Layer)

Behavioral rules that the agent follows. Skipping them causes mistakes. Where muscles are *learned patterns*, protocols are *enforced rules*.

- Carry a `breadcrumb` field — a 1-2 sentence TL;DR for warm loading
- Heat-based loading: hot → full body in prompt, warm → breadcrumb only, cold → name only
- Domain-scoped via `applies-to` — a TypeScript protocol only loads in TypeScript projects
- Include `## When to Apply` and `## When NOT to Apply` sections

**Examples:** "Every file gets YAML frontmatter", "Never skip exhale", "Commits must be attributed to the correct identity"

**Format:**
```yaml
---
type: protocol
name: protocol-name
status: active
heat-default: warm
applies-to: [always]
breadcrumb: "One-liner TL;DR..."
tier: core
tags: [workflow, memory]
depends-on:
  protocols: []
  muscles: []
  automations: []
  scripts: []
---
```

**Body structure:**
```markdown
# Protocol Name

## TL;DR
3-5 bullets. The essential rules.

## Rule
Full protocol — reasoning, edge cases, examples.

## When to Apply
## When NOT to Apply
```

### 2.4 Scripts (Foundation Layer)

Executable code that the agent builds and uses. The tools that do the work — and often the starting point from which patterns emerge.

- Written in any language (Bash, Python, TypeScript, etc.)
- Run by the agent during work, by CI for enforcement, or by the user directly
- Often the first response to a problem — "I need to check X" → write a script
- Often paired with a protocol: the script enforces *how*, the protocol explains *why*
- Heat-tracked: frequently-used scripts surface in the agent's tool awareness

**Examples:** `soma-ship.sh` (push + sync + test), `soma-verify.sh` (health checks), `soma-code.sh` (codebase navigation), `soma-refactor.sh` (safe refactoring)

**Format:**
```bash
#!/usr/bin/env bash
# script-name — one-line description
#
# USE WHEN: the situation that calls for this script
# Related muscles: muscle-name (what pattern this embodies)
# Related protocols: protocol-name (what rules this enforces)
# Related scripts: other-script (how this connects)
```

Scripts declare their relationships in header comments, not YAML frontmatter. They're executable files, not Markdown. The agent discovers them by name, reads their headers for context, and runs them for capability.

**How scripts connect to other AMPS types:**
```
Script: soma-ship.sh         → does the work (push, sync, test)
Muscle: ship-cycle            → describes the pattern (when, why, what order)
Protocol: workflow             → enforces the rule (test before commit, push after)
Automation: /release           → orchestrates all three into a workflow
```

## 3. The Runtime Layer

One additional capability type exists but is NOT shareable hub content. It's part of the agent's runtime infrastructure.

### 3.1 Extensions

Code hooks into the agent's lifecycle events. Written in the agent framework's language (TypeScript for Soma/Pi). Execute automatically when their event fires.

- Framework-specific — an extension for Pi won't work in Cursor
- Deterministic: same event → same behavior
- Not distributed via the hub (too coupled to runtime)

**Examples:** Boot sequence, context monitoring, statusline, heat tracking

### 3.2 Skills (External Input)

Domain knowledge from outside the system. Skills are NOT an AMPS content type — they're an **input channel**. A skill written for Claude Code, Cursor, or any other agent framework can be consumed by an AMPS-compatible system without modification.

- Pure Markdown — no code execution, no AMPS-specific metadata required
- Framework-agnostic: works anywhere an LLM can read instructions
- Skills enter the system from outside; AMPS layers refine them through use
- A logo design skill teaches the technique. A muscle learns *your* preferences. A protocol enforces *your* standards.

Skills are the lowest-barrier way to contribute knowledge. But they live outside the AMPS stack — they're consumed by it, not part of it.

### 3.3 The Distinction

| | Scripts (AMPS) | Automations (AMPS) | Extensions (Runtime) | Skills (External) |
|---|---|---|---|---|
| **Part of AMPS** | ✅ | ✅ | ❌ | ❌ |
| **Format** | Bash/Python/etc. | Markdown + frontmatter | TypeScript/etc. | Markdown |
| **Trigger** | Agent decision or CI | Slash command | Lifecycle events | Task matching |
| **Shareable** | ✅ Hub content | ✅ Hub content | ❌ Framework-specific | ✅ Any framework |
| **Example** | `soma-ship.sh` | `/publish` workflow | `soma-boot.ts` | "SVG logo design" |

## 4. Evolution

AMPS content evolves through use — but not on a fixed ladder. The four types influence each other in cycles:

```
        ┌──────── refines ────────┐
        ↓                         │
    Scripts ←── enforces ── Protocols
        │                     ↑
        ↓                     │
    Muscles ──── evolves ─────┘
        │
        ↓
    Automations ← composes all three
```

A script is often the first response to a problem — `soma-ship.sh` existed before the `ship-cycle` muscle. The muscle crystallized from observing how the script was used. But equally: a protocol can create a muscle ("always test before commit" → the agent learns the pattern), which evolves the protocol ("add a pre-push verification step"), which produces an automation to reinforce both.

The relationships are circular, not hierarchical:
- **Scripts → Muscles**: patterns noticed in script usage become muscles
- **Protocols → Muscles**: rules create habits that become learned patterns
- **Muscles → Protocols**: repeated patterns crystallize into enforced rules
- **Automations → all three**: automations compose scripts, follow protocols, embody muscles
- **All three → Automations**: enough maturity in any layer can produce an automation

Not every piece of content evolves. Some stay where they are. A pattern becomes whatever it naturally is:
- A testing discipline you must always follow → **protocol**
- A codebase navigation tool you refine over time → stays a **script**
- A release workflow you repeat every time → **automation**
- A frequently-applied pattern → **muscle** (may never crystallize further)

**Skills can enter the system from outside.** A skill written for another agent framework works immediately. AMPS layers then refine it through use. But skills are input, not a layer — they're consumed by the stack, not part of it.

For task-specific navigation through AMPS content, see **[MAPS](../maps/)** — tested paths through the stack for recurring tasks. For plan-driven configuration that controls what loads and how the agent thinks, see **[PHASE](../phase/)**.

### 4.1 Authorship Funnel

```
                    Barrier to Create
                    ─────────────────►
                    Low           High

Automations   ░░░░████████████████░░░░░░░░  (workflow designer — orchestration)
Muscles       ████████████░░░░░░░░░░░░░░░░  (agent + user — observed)
Protocols     ░░░░████████████░░░░░░░░░░░░  (system designer — rules)
Scripts       ████████████████░░░░░░░░░░░░  (agent + dev — executable)
```

AMPS order. Scripts and muscles have the lowest barrier — the agent writes them naturally. Skills (external input) have even lower barrier but live outside the stack.

## 5. Dependencies

All AMPS content can declare cross-type dependencies:

```yaml
depends-on:
  protocols: [breath-cycle, git-identity]
  muscles: [micro-exhale]
  automations: []
  scripts: [soma-ship.sh]
```

### 5.1 Resolution
- On install: resolve dependencies automatically. Prompt user for missing deps.
- `--with-deps` flag: install all dependencies recursively
- Circular dependencies: error at install time
- Missing dependencies: warn but don't block (content may still be partially useful)

### 5.2 Rules
- Dependencies can cross types freely (a muscle can depend on a protocol)
- Dependencies are on names, not versions (AMP content uses names as identifiers)
- Optional dependencies use a separate field: `suggests` (same structure as `depends-on`)

## 6. Composition

Content types reference each other naturally:

```
Automation "/release"
  ├── depends-on Protocol "git-identity" (attribution rule)
  ├── depends-on Script "soma-changelog.sh" (changelog generation)
  ├── depends-on Automation "changelog" (sub-workflow)
  └── uses Muscle "ship-cycle" (learned release patterns)
```

The dependency flows naturally: **Automations compose everything. Protocols inform automations. Scripts do the work. Muscles remember the patterns.**

## 7. Directory Structure

Within an AMP-compatible memory directory:

```
.soma/
├── amps/
│   ├── automations/        # Executable workflows + MAPs
│   │   ├── maps/
│   │   │   └── refactor.md
│   │   └── release/
│   │       └── AUTOMATION.md
│   ├── muscles/            # Learned patterns
│   │   ├── ship-cycle.md
│   │   └── code-navigator.md
│   ├── protocols/          # Behavioral rules
│   │   ├── workflow.md
│   │   └── quality-standards.md
│   └── scripts/            # Executable tools
│       ├── soma-ship.sh
│       └── soma-verify.sh
├── identity.md
├── STATE.md
└── settings.json
```

The `amps/` directory follows the acronym order: **A**utomations, **M**uscles, **P**rotocols, **S**cripts. Other directories (memory, projects, skills) may live alongside or outside `.soma/` depending on the user's setup — they're not part of the AMPS structure.

## 8. Hub Distribution

AMPS content is distributed via the **Soma Hub** (or any compatible registry). The hub serves a `hub-index.json`:

```json
{
  "version": 1,
  "generated": "2026-03-11T00:00:00Z",
  "count": 17,
  "items": [
    {
      "slug": "breath-cycle",
      "type": "protocol",
      "name": "breath-cycle",
      "description": "...",
      "tier": "core",
      "depends-on": { "protocols": [], "muscles": [], "automations": [], "scripts": [] }
    }
  ]
}
```

### 8.1 Tiers

| Tier | Meaning |
|------|---------|
| **core** | Ships with the reference implementation. Gravicity-authored. |
| **official** | Gravicity-authored. Curated but not essential. |
| **community** | Anyone. Default tier for contributions. |
| **pro** | Premium/private. Future. |

`core ⊂ official` — all core content is also official.

## 9. Attribution

```
This project uses AMPS (Agent Memory Protocol Stack)
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*AMPS v1.1 — Curtis Mercier — CC BY 4.0*
*Extends: Agent Memory Protocol (AMP) v0.3*
*Supersedes: Agent Capability Model v0.2*
*Reference implementation: Soma (soma.gravicity.ai)*
