---
type: spec
status: draft
version: 1.0.0
created: 2026-03-11
updated: 2026-03-11
author: Curtis Mercier
license: CC BY 4.0
extends: amp/0.3
---

# AMPS — Agent Memory Protocol Stack v1.0

> Four shareable content types that extend the Agent Memory Protocol. Automations, Muscles, Protocols, Skills — the building blocks of an evolving agent.

*Extends: [AMP v0.3](../amp/) (Agent Memory Protocol)*
*Supersedes: [Agent Capability Model v0.2](../three-layer/) (archived)*

## 1. Overview

AMPS defines what lives inside an AMP-compatible memory system. Four content types, each with a different nature, authorship profile, and loading behavior — plus a runtime layer that executes them.

**AMPS** = **A**utomations · **M**uscles · **P**rotocols · **S**kills

The name is deliberate: AMP is the memory engine, AMPS is what it amplifies.

## 2. The Four Content Types

| Layer | Type | Nature | Author | Loading |
|-------|------|--------|--------|---------|
| 1 | **Skills** | Domain knowledge, on demand | Anyone | When task matches |
| 2 | **Muscles** | Learned patterns from experience | Agent + user | By heat, digest-first |
| 3 | **Protocols** | Behavioral rules, enforced | System designer | By heat, full or breadcrumb |
| 4 | **Automations** | Executable workflows | Workflow designer | Registered as commands |

### 2.1 Skills (Knowledge Layer)

Markdown files containing domain expertise that the agent reads and follows.

- Pure Markdown — no code execution
- Loaded into agent context when the task matches (keyword/description matching)
- The agent reads the skill and follows its instructions
- Anyone can write a skill — lowest barrier to contribution
- **Framework-agnostic**: a skill from Claude Code, Cursor, or any agent framework works without modification

**What makes skills special in AMPS:** Muscles and protocols *refine* skills. A logo design skill teaches the technique. A muscle learns *your* preferences. A protocol enforces *your* standards. The skill provides raw expertise; the layers above improve it through use — without the user ever asking.

**Examples:** "How to deploy to Vercel", "SVG logo design principles", "cPanel DNS management"

**Format:**
```yaml
---
type: skill
name: skill-name
status: active
breadcrumb: "One-liner..."
tier: community
topic: [design, icons]
keywords: [svg, logo]
depends-on:
  protocols: []
  muscles: []
  automations: []
  skills: []
---
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
  skills: []
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
  skills: []
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

### 2.4 Automations (Reflex Layer)

Executable workflows triggered by command, combining knowledge with structured execution. The final crystallization — what was once a manual pattern becomes a repeatable action.

Automations absorb what were previously called "rituals" (multi-step workflows) and shareable scripts (community automation patterns). They are distinct from internal scripts (enforcement code) and extensions (framework-specific hooks).

- Triggered by command (e.g., `/publish`, `/release`, `/audit`)
- Multi-step with explicit phases — conversational, each step may involve the user
- Can reference skills, follow protocols, and depend on other automations
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
  skills: [semantic-versioning]
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

## 3. The Runtime Layer

Two additional capability types exist but are NOT shareable hub content. They're part of the agent's runtime infrastructure.

### 3.1 Extensions

Code hooks into the agent's lifecycle events. Written in the agent framework's language (TypeScript for Soma/Pi). Execute automatically when their event fires.

- Framework-specific — an extension for Pi won't work in Cursor
- Deterministic: same event → same behavior
- Not distributed via the hub (too coupled to runtime)

**Examples:** Boot sequence, context monitoring, statusline, heat tracking

### 3.2 Scripts (Internal)

Executable code that enforces protocols automatically. Run by the agent or by CI — not loaded into the prompt.

- Often paired with a protocol: the protocol explains *why*, the script enforces *how*
- The protocol doesn't disappear when a script exists — it remains the reasoning layer
- Internal to the implementation, not shareable as content

**Examples:** PII audit, frontmatter validator, drift checker, date hook

### 3.3 The Distinction

| | Automations (AMPS) | Scripts (Runtime) | Extensions (Runtime) |
|---|---|---|---|
| **Shareable** | ✅ Hub content | ❌ Internal | ❌ Framework-specific |
| **Format** | Markdown + frontmatter | Bash/Python/etc. | TypeScript/etc. |
| **Trigger** | Slash command | Agent decision or CI | Lifecycle events |
| **User-facing** | Yes — conversational | No — runs silently | No — runs automatically |
| **Example** | `/publish` workflow | `validate-frontmatter.sh` | `soma-boot.ts` |

## 4. Evolution

Knowledge matures through use. This is the common evolution path, not a required progression:

```
observation (noticed gap, repeated action)
  ↓ seen 2+ times → write it down
skill (domain knowledge — how to know)
  ↓ refined through use
muscle (learned pattern — personal refinement)
  ↓ crystallizes based on nature
  ├──→ protocol    (behavioral rule — how to be)
  └──→ automation  (executable workflow — how to do)
```

Not every piece of content climbs the full ladder. Some stay skills forever. A pattern becomes whatever it naturally is:
- A testing discipline you must always follow → **protocol**
- A design technique you sometimes need → stays a **skill**
- A release workflow you repeat every time → **automation**
- A frequently-applied pattern → **muscle** (may never crystallize further)

**Skills can also be entry points from outside the system.** A skill written for another agent framework works immediately. AMPS muscles and protocols then refine it through use.

### 4.1 Authorship Funnel

```
                    Barrier to Create
                    ─────────────────►
                    Low           High

Skills        ████████████████░░░░░░░░░░░░  (anyone — Markdown)
Muscles       ████████████░░░░░░░░░░░░░░░░  (agent + user — observed)
Protocols     ░░░░████████████░░░░░░░░░░░░  (system designer — rules)
Automations   ░░░░████████████████░░░░░░░░  (workflow designer — steps)
```

The easy stuff has the lowest barrier. This is how communities scale.

## 5. Dependencies

All AMPS content can declare cross-type dependencies:

```yaml
depends-on:
  protocols: [breath-cycle, git-identity]
  muscles: [micro-exhale]
  automations: []
  skills: [semantic-versioning]
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
  ├── depends-on Skill "semantic-versioning" (knowledge)
  ├── depends-on Automation "changelog" (sub-workflow)
  └── uses Muscle "release-patterns" (learned preferences)
```

The dependency flows naturally: **Automations compose everything. Protocols inform automations. Skills are standalone. Muscles refine all layers.**

## 7. Directory Structure

Within an AMP-compatible memory directory:

```
.soma/
├── protocols/              # Behavioral rules
│   ├── breath-cycle.md
│   └── git-identity.md
├── memory/
│   └── muscles/            # Learned patterns
│       ├── git-workflow.md
│       └── api-patterns.md
├── skills/                 # Domain knowledge
│   └── logo-design/
│       └── SKILL.md
├── automations/            # Executable workflows
│   └── release/
│       └── AUTOMATION.md
├── identity.md
├── STATE.md
└── settings.json
```

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
      "depends-on": { "protocols": [], "muscles": [], "automations": [], "skills": [] }
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

*AMPS v1.0 — Curtis Mercier — CC BY 4.0*
*Extends: Agent Memory Protocol (AMP) v0.3*
*Supersedes: Agent Capability Model v0.2*
*Reference implementation: Soma (soma.gravicity.ai)*
