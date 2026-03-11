---
type: spec
status: draft
version: 0.2.0
created: 2026-03-10
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
---

# Agent Capability Model — Specification v0.2

> A separation of concerns for agent capabilities into six types, each with a different nature, authorship profile, and lifecycle.

*Previously: Three-Layer Extensibility Model (v0.1). Expanded to reflect the full hierarchy that emerged from practice.*

## 1. Motivation

Single-layer agent extensibility doesn't scale:
- **Code-only** (like VS Code plugins): every capability requires a developer. Can automate, can't teach.
- **Prompts-only** (like ChatGPT custom instructions): can teach, can't automate. Knowledge without action.
- **Undifferentiated** (everything in one bucket): the junk drawer. Is this a plugin? A prompt? A template?

Six capability types, each with a clear purpose, different authorship profiles, and different lifecycles.

## 2. The Capabilities

| Capability | Nature | File Type | Author | Loads |
|-----------|--------|-----------|--------|-------|
| **Extension** | Code hook into agent lifecycle | TypeScript/Python | Developer | Fires on events, automatic |
| **Skill** | Domain knowledge, on demand | Markdown | Anyone | When task matches |
| **Muscle** | Learned pattern from experience | Markdown | Agent + user | By heat, digest-first |
| **Protocol** | Behavioral rule, enforced | Markdown | System designer | By heat, full or breadcrumb |
| **Ritual** | Multi-step workflow | Markdown + config | Workflow designer | Triggered by command |
| **Script** | Automated enforcement | Bash/Python | Developer | Executed, not loaded into prompt |

### 2.1 Extensions (Reflex Layer)

Deterministic code that hooks into the agent's lifecycle events.

```
session_start → boot extension fires → loads identity
context_threshold → flush extension fires → writes preload
before_send → validator extension fires → checks output
```

**Properties:**
- Written in the agent framework's language
- Execute automatically when their event fires
- Can modify state, call APIs, write files, interact with the runtime
- Same event → same behavior (deterministic)

**Examples:** Boot sequence, context monitoring, statusline, git hooks, auto-formatting.

### 2.2 Skills (Knowledge Layer)

Markdown files containing domain expertise that the agent reads and follows.

**Properties:**
- Pure Markdown — no code execution
- Loaded into agent context when the task matches (keyword/description matching)
- The agent reads the skill and follows its instructions
- Anyone can write a skill — lowest barrier to contribution
- Framework-agnostic: a skill from Claude Code, Cursor, or any framework works without modification

**Examples:** "How to deploy to Vercel", "SVG logo design principles", "cPanel DNS management".

**What makes skills special:** They're the universal building block. Other systems have skills too. What AMP adds is the *refinement layer* — muscles and protocols that personalize skills through use. A logo design skill teaches the technique. A muscle learns *your* preferences. A protocol enforces *your* standards. The skill provides raw expertise; the layers above improve it every time it's used, without the user ever asking.

### 2.3 Muscles (Learning Layer)

Patterns the agent discovered through experience and encoded for future reuse.

**Properties:**
- Born from gaps — moments where friction was noticed, a pattern repeated, a workflow failed
- Grow through use: heat rises on application, decays on neglect
- Two-tier loading: digest (compressed) for tight context, full body when space allows
- The richest source of muscles is observing the *user's* patterns — how they like PRs, what they always verify, their communication preferences

**Examples:** "Always run tests after file removal", "User prefers concise responses", "This API returns paginated results — always check for next page".

**The key insight:** Muscles aren't instructions the user wrote. They're patterns the agent noticed and encoded. The user's repeated behaviors are the richest source of new muscles.

### 2.4 Protocols (Instinct Layer)

Behavioral rules that the agent follows. Skipping them causes mistakes.

**Properties:**
- Carry a `breadcrumb` field — a 1-2 sentence TL;DR for warm loading
- Heat-based loading: hot protocols inject fully, warm protocols inject as breadcrumbs
- Domain-scoped via `applies-to` tags — a TypeScript protocol only loads in TypeScript projects
- Include `## When to Apply` and `## When NOT to Apply` sections

**Examples:** "Every file gets YAML frontmatter", "Never skip exhale", "Commits must be attributed to the correct identity".

### 2.5 Rituals (Workflow Layer)

Multi-step procedures triggered by a command, combining knowledge with structured execution.

**Properties:**
- Triggered by a command (e.g., `/publish`, `/release`)
- Multi-step with explicit phases
- Conversational — each step involves the user
- Can reference skills and trigger extensions
- Stateful: tracks which step the agent is on

**Examples:** `/publish` (blog post), `/release` (software version), `/review` (code review workflow).

### 2.6 Scripts (Enforcement Layer)

The final crystallization — what was once a behavioral rule the agent followed manually becomes code that runs automatically.

**Properties:**
- Executable code (bash, python, etc.)
- Run by the agent or by CI/hooks — not loaded into the prompt
- Often paired with a protocol: the protocol explains *why*, the script enforces *how*
- The protocol doesn't disappear when a script exists — it remains the reasoning layer

**Examples:** PII audit script (enforces community-safe protocol), frontmatter validator (enforces frontmatter-standard), drift checker (enforces code sync between repos).

## 3. How Capabilities Relate

### 3.1 The Evolution Path

Capabilities aren't created equal — they evolve from observation:

```
observation (noticed gap, repeated action)
  ↓ seen 2+ times
muscle (learned pattern)
  ↓ crystallizes based on nature
  ├──→ protocol  (behavioral rule — how to be)
  ├──→ skill     (domain knowledge — how to know)
  ├──→ ritual    (workflow — how to do)
  └──→ script    (automation — how to enforce)
```

A muscle doesn't choose its destination consciously. It becomes whatever it naturally is:
- A testing pattern you must always follow → **protocol**
- A logo design technique you sometimes need → **skill**
- A publish workflow you repeat every release → **ritual**
- A protocol whose checks can be automated → **script**

### 3.2 Composition

Layers can reference each other:

```
Ritual "/release"
  ├── loads Skill "semantic-versioning" (knowledge)
  ├── triggers Extension "git-tag" (automation)
  ├── follows Protocol "git-identity" (rule)
  └── runs Script "audit.sh" (enforcement)
```

The dependency flows naturally: **Rituals compose everything. Extensions can load skills. Protocols inform scripts. Skills are standalone.**

### 3.3 Authorship Funnel

```
                    Barrier to Create
                    ─────────────────►
                    Low           High

Skills        ████████████████░░░░░░░░░░░░  (anyone — Markdown)
Muscles       ████████████░░░░░░░░░░░░░░░░  (agent + user — observed patterns)
Protocols     ░░░░████████████░░░░░░░░░░░░  (system designer — rules)
Rituals       ░░░░████████████████░░░░░░░░  (workflow designer — multi-step)
Scripts       ░░░░░░░░░░░░████████████████  (developer — code)
Extensions    ░░░░░░░░░░░░░░░░████████████  (developer — framework knowledge)
```

The easy stuff has the lowest barrier. This is how communities scale — many skill authors, some ritual designers, few extension developers.

## 4. Directory Structure

```
.soma/
├── extensions/          # Code hooks
│   └── boot.ts
├── skills/              # Domain knowledge
│   └── deploy/
│       └── SKILL.md
├── memory/
│   └── muscles/         # Learned patterns
│       └── git-workflow.md
├── protocols/           # Behavioral rules
│   └── breath-cycle.md
├── rituals/             # Multi-step workflows
│   └── release/
│       └── SKILL.md
├── scripts/             # Automated enforcement
│   └── audit.sh
├── identity.md
├── STATE.md
└── settings.json
```

Resolution order for all types: **project → parent workspace → user-global → remote registry**.

## 5. Skill Registry

Skills (and rituals) can be distributed via a JSON registry:

```json
{
  "skills": [
    {
      "name": "favicon-gen",
      "description": "Generate favicons from logos or text",
      "version": "1.0.0",
      "author": "Curtis Mercier",
      "license": "MIT",
      "keywords": ["favicon", "icon"],
      "path": "skills/favicon-gen/"
    }
  ]
}
```

Discovery, installation, and resolution are implementation-defined.

## 6. Attribution

```
This project uses the Agent Capability Model
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Agent Capability Model v0.2 — Curtis Mercier — CC BY 4.0*
*Previously: Three-Layer Extensibility Model v0.1*
*Reference implementation: Soma (soma.gravicity.ai)*
