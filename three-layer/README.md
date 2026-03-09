---
type: spec
status: draft
version: 0.1.0
created: 2026-03-10
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
---

# Three-Layer Extensibility Model — Specification v0.1

> A separation of concerns for agent capabilities into extensions (code), skills (knowledge), and rituals (workflows).

## 1. Motivation

Single-layer agent extensibility doesn't scale:
- **Code-only** (like VS Code plugins): every capability requires a developer. Can automate, can't teach.
- **Prompts-only** (like ChatGPT custom instructions): can teach, can't automate. Knowledge without action.
- **Undifferentiated** (everything in one bucket): the junk drawer. Is this a plugin? A prompt? A template?

Three layers, each with a clear purpose, different authorship profiles, and different lifecycles.

## 2. The Layers

| Layer | What | File Type | Author | Lifecycle |
|-------|------|-----------|--------|-----------|
| **Extension** | Code hook into agent lifecycle | TypeScript/Python | Developer | Fires on events, automatic |
| **Skill** | Domain knowledge loaded on demand | Markdown | Domain expert | Loaded when relevant, read |
| **Ritual** | Multi-step workflow triggered by command | Markdown + config | Workflow designer | Triggered by user, stepped |

### 2.1 Extensions (Reflex Layer)

Deterministic code that hooks into the agent's lifecycle events.

```
session_start → boot extension fires → loads identity
context_threshold → flush extension fires → writes preload
before_send → validator extension fires → checks output
```

**Properties:**
- Written in the agent framework's language (TypeScript, Python, etc.)
- Execute automatically when their event fires
- Can modify state, call APIs, write files, interact with the runtime
- Same event → same behavior (deterministic)
- Installed to: `.soma/extensions/` (project) or `~/.soma/agent/extensions/` (global)

**Examples:** Boot sequence, context monitoring, statusline, git hooks, auto-formatting.

### 2.2 Skills (Knowledge Layer)

Markdown files containing domain expertise that the agent reads and follows.

```markdown
---
name: favicon-gen
type: skill
description: Generate favicons from logos, text, or brand colors
keywords: [favicon, icon, branding]
---

# Favicon Generation

When the user asks to create favicons, follow these steps:
1. Ask for the source (logo file, text, or brand colors)
2. Generate SVG source at multiple sizes...
```

**Properties:**
- Pure Markdown — no code execution
- Loaded into agent context when the task matches (keyword/description matching)
- The agent reads the skill and follows its instructions
- Anyone can write a skill — lowest barrier to contribution
- Installed to: `.soma/skills/` (project) or `~/.soma/agent/skills/` (global)

**Resolution order:** project → parent workspace → user-global → remote registry

**Examples:** "How to deploy to Vercel", "SVG logo design principles", "cPanel DNS management".

### 2.3 Rituals (Workflow Layer)

Multi-step procedures triggered by a command, combining knowledge with structured execution.

```markdown
---
name: publish
type: ritual
trigger: /publish
description: Write, commit, push, and deploy a blog post
---

# /publish Ritual

## Step 1: Plan
Ask the user for title and topic...

## Step 2: Draft
Write the post with proper frontmatter...

## Step 3: Review
Verify content, ask for approval...

## Step 4: Ship
Git add, commit, push, deploy...
```

**Properties:**
- Triggered by a command (e.g., `/publish`, `/release`)
- Multi-step with explicit phases
- Conversational — each step involves the user
- Can reference skills and trigger extensions
- Stateful: can track which step the agent is on
- Installed to: `.soma/rituals/` (project) or `~/.soma/agent/rituals/` (global)

**Examples:** `/publish` (blog post), `/release` (software release), `/review` (code review workflow).

## 3. Why Three and Not Two

The three layers map to different **authorship profiles**:

```
                    Barrier to Create
                    ─────────────────►
                    Low           High

Extensions    ░░░░░░░░░░░░░░░░████████████  (developer required)
Skills        ████████████████░░░░░░░░░░░░  (anyone can write Markdown)
Rituals       ░░░░████████████████░░░░░░░░  (workflow design, medium skill)
```

This maps to the open source contribution funnel:
- **Many** skill authors (domain experts, users, the agent itself)
- **Some** ritual designers (power users who define workflows)
- **Few** extension developers (need framework knowledge)

The easy stuff has the lowest barrier. This is how communities scale.

## 4. Interaction Between Layers

Layers can reference each other:

```
Ritual "/release"
  ├── loads Skill "semantic-versioning" (knowledge)
  ├── triggers Extension "git-tag" (automation)
  └── loads Skill "changelog-format" (knowledge)
```

- Rituals compose skills and extensions
- Extensions can load skills into context
- Skills are standalone — they don't depend on extensions or rituals

The dependency flows one way: **Rituals → Extensions → Skills**. Skills are the foundation.

## 5. Directory Structure

```
.soma/
├── extensions/          # Project-level code hooks
│   └── pre-commit.ts
├── skills/              # Project-level knowledge
│   └── deploy/
│       └── SKILL.md
└── rituals/             # Project-level workflows
    └── release/
        └── SKILL.md

~/.soma/agent/
├── extensions/          # User-global code hooks
├── skills/              # User-global knowledge
└── rituals/             # User-global workflows
```

## 6. Skill Index (Registry)

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
      "category": "design",
      "path": "skills/favicon-gen/"
    }
  ]
}
```

Discovery: `soma skill list --remote`
Install: `soma skill install favicon-gen`
Resolution: project → parent → global → registry

## 7. Attribution

```
This project uses the Three-Layer Extensibility Model
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Three-Layer Model v0.1 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
