---
type: spec
status: draft
version: 0.1.0
created: 2026-03-10
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
---

# Agent Memory Protocol (AMP) — Specification v0.1

> A filesystem-based protocol for AI agents to accumulate, organize, and retrieve knowledge across sessions without external databases.

## 1. Introduction

Most AI agent frameworks treat memory as a retrieval problem — vector databases, embeddings, similarity search. AMP takes a different approach: the agent reads and writes plain files. Like a human with a notebook.

This works because:
- LLMs are excellent at reading and following written instructions
- Filesystem operations are universal (any language, any OS, any agent framework)
- No infrastructure required — no database, no embeddings model, no cloud service
- The agent can inspect, edit, and curate its own memory
- Humans can read and modify the agent's memory too (it's just Markdown)

## 2. Concepts

### 2.1 Memory
Persistent knowledge stored as files on disk. Organized in a directory hierarchy. Survives between sessions. The agent reads relevant memory at boot and writes to it during work.

### 2.2 Muscle
A crystallized pattern — a piece of knowledge the agent learned through experience and encoded for future reuse. Named after biological muscle memory: the more you use it, the stronger it gets.

### 2.3 Preload
A session continuation document. Written at the end of a session (exhale), consumed at the start of the next session (inhale). Contains: where we left off, what was accomplished, what's next, and what to skip.

### 2.4 Heat
A numeric measure of how frequently a memory (muscle) is accessed. High heat = frequently used = more relevant. Heat decays over time — unused memories cool off. This creates a natural relevance ranking without embeddings.

### 2.5 Promotion
Moving a memory from a narrow scope (project-level) to a broader scope (user-level or global). Triggered when a pattern proves useful across multiple contexts.

### 2.6 Flush
The process of detecting context depletion, extracting essential state, and persisting it to disk before the session ends or continues.

## 3. Directory Structure

```
.soma/                          # AMP-compatible directory (any name works)
├── memory/
│   ├── muscles/                # Learned patterns
│   │   ├── git-workflow.md
│   │   └── api-patterns.md
│   ├── preload-next.md         # Session continuation state
│   └── sessions/               # Optional: session transcripts/logs
├── identity.md                 # Agent identity (see Identity System spec)
├── STATE.md                    # Architecture truth (see ATLAS spec)
└── settings.json               # Configuration (see §8)
```

### 3.1 Memory Hierarchy

Memory exists at multiple scopes. Resolution follows most-specific-first:

```
1. project/.soma/memory/         ← project-level (most specific)
2. workspace/.soma/memory/       ← workspace/parent level
3. ~/.soma/memory/               ← user-global
4. /etc/soma/memory/             ← system-level (optional, rare)
```

A muscle with the same name at a narrower scope shadows the broader one. This lets a project override global patterns when needed.

## 4. Muscle Format

Muscles are Markdown files with YAML frontmatter.

### 4.1 Required Frontmatter

```yaml
---
type: muscle
status: active              # active | archived
topic: <domain>             # what area this covers
keywords: [<tag>, ...]      # searchable terms
heat: <number>              # usage frequency (starts at 1)
loads: <number>             # total times loaded into context (starts at 0)
scope: <local|shared>       # eligible for upward flow? (default: local)
created: <YYYY-MM-DD>
updated: <YYYY-MM-DD>
---
```

### 4.2 Body Structure

```markdown
# <Title>

<!-- digest:start -->
<Compressed essential knowledge. 2-5 sentences.
This is what the agent reads FIRST. If context is tight,
only the digest may be loaded.>
<!-- digest:end -->

<Full content below. Detailed instructions, examples,
edge cases, history. Loaded when context allows.>
```

The `<!-- digest:start/end -->` markers enable two-tier loading:
- **Tight context:** Load only the digest (the compressed essence)
- **Full context:** Load the entire muscle

### 4.3 Heat Rules

- When a muscle is loaded into agent context: `loads += 1`
- Heat recalculation: implementation-defined (e.g., `heat = loads / days_since_created`)
- Decay: muscles not loaded for N days have heat reduced
- Minimum heat for relevance: implementation-defined
- Heat is informational — it guides the agent's memory loader, not a hard rule

### 4.4 Example

```markdown
---
type: muscle
status: active
topic: git-workflow
keywords: [git, branching, commits, pr]
heat: 7
loads: 23
scope: shared
created: 2026-03-01
updated: 2026-03-10
---

# Git Workflow Patterns

<!-- digest:start -->
This team uses trunk-based development. Short-lived feature branches,
squash merges, conventional commits (feat:/fix:/chore:). Always rebase
before merge. Never force-push shared branches.
<!-- digest:end -->

## Branch Naming
- `feat/<ticket>-<short-description>`
- `fix/<ticket>-<short-description>`

## Commit Format
`<type>: <description>` — types: feat, fix, chore, docs, refactor, test

## PR Process
1. Push branch
2. Create PR with description template
3. Request review
4. Squash merge after approval
```

## 5. Preload Format

Preloads bridge sessions. Written at exhale (end of session), consumed at inhale (start of next session).

### 5.1 Required Structure

```markdown
---
type: preload
agent: <agent-name>
created: <YYYY-MM-DD>
session: <session-identifier>
---

# Preload — <Session Description>

## Resume Point
<Where we left off. Dense, 1-3 sentences. The agent reads this
to immediately orient: "this is what I was doing.">

## What Shipped
<Completed work this session. Prevents re-doing finished tasks.>

## What's Next
<Priority-ordered task list for the next session.>

## What NOT to Re-read
<Negative context. Files, topics, or docs the agent should skip
because they were already fully processed.>

## Key Files
<Quick reference table of important paths touched this session.>
```

### 5.2 Negative Context

The "What NOT to Re-read" section is critical. Without it, an agent that has processed a large codebase will re-read the same files every session, burning context on already-understood material.

Negative context says: "I already know this. Don't spend tokens on it."

### 5.3 Preload Lifecycle

```
Session N exhale:  write preload-next.md
Session N+1 boot:  read preload-next.md → orient → work
Session N+1 exhale: overwrite preload-next.md with new state
```

Only one preload exists at a time. Each session overwrites the previous one. Historical preloads can be moved to `sessions/` for archival if desired.

## 6. Flush Pipeline

Flush is the process of persisting state when context is depleted.

### 6.1 Trigger

The agent (or an extension) monitors context usage. When usage exceeds a threshold (default: 85% of the context window), flush begins.

### 6.2 Steps

```
1. DETECT    — context usage ≥ threshold
2. EXTRACT   — summarize: what was done, what's in progress, what's next
3. PERSIST   — write preload-next.md
4. PROMOTE   — if new patterns emerged, write/update muscles
5. SIGNAL    — indicate continuation is ready
6. TERMINATE — end current session (or auto-continue)
```

### 6.3 What Gets Extracted

The agent determines what state is essential for continuation:
- Current task and progress
- Decisions made this session
- Files modified
- Unresolved questions
- Priority order for next session

What does NOT get extracted:
- Full conversation history (too large)
- File contents (the files exist on disk)
- Already-known context (in muscles or preloads)

## 7. Promotion

Promotion moves a memory from a narrow scope to a broader one.

### 7.1 Conditions

A muscle is eligible for promotion when:
- It exists at project scope
- It has been loaded across N different projects (configurable, default: 2)
- Its heat exceeds a threshold (configurable)
- It has `scope: shared` (or no scope restriction)

### 7.2 Mechanism

```
project-A/.soma/memory/muscles/pattern.md    (heat: 5, seen in 3 projects)
    ↓ promote
~/.soma/memory/muscles/pattern.md            (user-global, available everywhere)
```

The project-level copy can be kept (for project-specific overrides) or removed (if the global version is sufficient).

### 7.3 Automatic vs Manual

- **Manual promotion:** The agent or user explicitly copies a muscle to a broader scope. Recommended for v1 implementations.
- **Automatic promotion:** The agent detects cross-project patterns and promotes autonomously. Requires tracking muscle usage across projects.

## 8. Configuration

AMP configuration lives in `settings.json` alongside the memory directory.

### 8.1 Memory Settings

```json
{
  "memory": {
    "flowUp": false,
    "flowFilter": null,
    "flowMode": "copy",
    "autoPromote": false,
    "promotionThreshold": {
      "heat": 5,
      "loads": 10,
      "crossProject": 2
    }
  },
  "flush": {
    "threshold": 0.85,
    "autoFlush": true,
    "autoContinue": true
  }
}
```

### 8.2 Flow Settings (Parent-Child)

When multiple AMP instances exist in a workspace hierarchy, memories can flow upward from child to parent.

- **`flowUp`**: Whether this instance shares muscles with its parent (default: `false`)
- **`flowFilter`**: Rules for which muscles flow up (by scope, heat, topic)
- **`flowMode`**: `"copy"` (duplicate to parent) or `"reference"` (metadata pointer)
- **Flow is child-controlled.** The child decides what to share. The parent does not pull.

## 9. Implementation Notes

### 9.1 Requirements
- Filesystem read/write capability
- Markdown parsing (for frontmatter extraction)
- No external services required

### 9.2 Compatibility
Any AI agent framework that can read and write files can implement AMP. The protocol is:
- **Language-agnostic** — works with Python, TypeScript, Rust, anything
- **LLM-agnostic** — works with any model that can read Markdown
- **Platform-agnostic** — works on any OS with a filesystem

### 9.3 Reference Implementation
[Soma](https://soma.gravicity.ai) — MIT-licensed, open source.
GitHub: [github.com/meetsoma](https://github.com/meetsoma)

## 10. Attribution

When implementing AMP, include attribution in your project:

```
This project implements the Agent Memory Protocol (AMP)
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Agent Memory Protocol (AMP) v0.1 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
