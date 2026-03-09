---
type: spec
status: draft
version: 0.1.0
created: 2026-03-10
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
---

# Agent Identity System — Specification v0.1

> A protocol for how AI agents discover and assume contextual identity based on the project and environment they operate in.

## 1. The Problem

Most agents are generic. You talk to "Claude" or "GPT" regardless of context. Custom instructions exist but they're global — same personality everywhere.

The Identity System makes agents context-aware:
- In a web project, the agent knows web frameworks and speaks that language
- In an infrastructure project, it thinks in terms of deployments and reliability
- In a creative project, it's more exploratory and visual

Identity isn't just tone. It affects what the agent prioritizes, what memory it loads, and how it structures work.

## 2. Discovery

When an agent starts in a directory, it discovers its identity:

```
1. Check current directory for .soma/identity.md
2. If not found, walk up: parent, grandparent, etc.
3. If found: load it (project identity)
4. Also load: ~/.soma/identity.md (user-level base identity)
5. Project identity layers ON TOP of base identity
6. Result: agent knows who it is in this specific context
```

Discovery is automatic. The agent doesn't need to be told where to look — it walks the filesystem.

## 3. Identity File Format

```markdown
---
type: identity
agent: <name>
project: <project name>
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---

# <Agent Name> — <Project>

## Who You Are
<Personality, voice, values, approach>

## This Project
<What the project is, what we're building, who it's for>

## How You Work
<Methodology, preferences, conventions specific to this context>

## What You Know
<Domain expertise, relevant skills, loaded muscles>

## Conventions
<Language, framework, style guide, team practices>
```

## 4. Inheritance

Identity is layered, not replaced.

```
~/.soma/identity.md              "I am Soma, I grow memory, I'm methodical"
  └── project/.soma/identity.md   "In THIS project, I focus on React and I use Tailwind"
```

The project identity extends the base. The agent is still Soma — but specialized.

### 4.1 Override vs Extend

- **Extend (default):** Project identity adds context. Base personality persists.
- **Override:** Project identity can explicitly override base traits if needed.

```markdown
## How You Work
<!-- extends base -->
In addition to your normal approach, in this project:
- Always use TypeScript strict mode
- Write tests before implementation

<!-- or override -->
In this project, ignore your default testing approach. We ship fast and fix later.
```

## 5. Multiple Identities in a Workspace

In a parent-child workspace, each `.soma/` can have its own identity:

```
workspace/.soma/identity.md       "I oversee the full stack"
├── frontend/.soma/identity.md    "I handle React components"
├── api/.soma/identity.md         "I handle the Go API"
└── infra/.soma/identity.md       "I handle deployment and ops"
```

When the agent operates in `frontend/`, it discovers the frontend identity. When it operates at the workspace root, it gets the workspace identity.

## 6. Why Identity Matters

This isn't personification for its own sake. Identity serves engineering goals:

- **Consistency:** An agent with identity makes consistent decisions across sessions. Without identity, it's a blank slate every time.
- **Memory coherence:** Identity tells the agent which memories are relevant. A frontend-focused identity loads frontend muscles.
- **Communication:** Users know what to expect. "Soma in this project knows React" is more useful than "the AI assistant."
- **Scoping:** Identity prevents the agent from overstepping. A frontend identity won't try to modify database schemas.

## 7. Implementation Notes

- Identity files are Markdown — the agent reads them like any other document
- Discovery should happen early in the boot sequence (Breath Cycle inhale phase)
- Identity affects muscle loading: the agent should prefer muscles that match its current identity's domain
- Identity files should be version-controlled (they're project config, not secrets)

## 8. Attribution

```
This project uses the Agent Identity System
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Identity System v0.1 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
