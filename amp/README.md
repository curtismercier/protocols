---
type: spec
status: draft
version: 0.3.0
created: 2026-03-10
updated: 2026-05-12
author: Curtis Mercier
license: CC BY 4.0
---

# Agent Memory Protocol (AMP) — Specification v0.3

> A filesystem-based protocol for AI agents to accumulate, organize, and retrieve knowledge across sessions without external databases.

## 1. Introduction

Most AI agent frameworks treat memory as a retrieval problem — vector databases, embeddings, similarity search. AMP takes a different approach: the agent reads and writes plain files. Like a human with a notebook.

This works because:
- LLMs are excellent at reading and following written instructions
- Filesystem operations are universal (any language, any OS, any agent framework)
- No infrastructure required — no database, no embeddings model, no cloud service
- The agent can inspect, edit, and curate its own memory
- Humans can read and modify the agent's memory too (it's just Markdown)

## 2. Core Concepts

### 2.1 Memory
Persistent knowledge stored as files on disk. Organized in a directory hierarchy. Survives between sessions. The agent reads relevant memory at boot and writes to it during work.

### 2.2 Content Types
Memory consists of typed content — each type has a different nature, authorship profile, and loading behavior. AMP defines the mechanics (heat, flush, preload) that act on content. The content types themselves are defined by extensions to this protocol.

The reference content type system is **[AMPS](../amps/)** (Agent Memory Protocol Stack), which defines four shareable content types: Automations, Muscles, Protocols, and Skills. For task-specific navigation through content, see **[MAPS](../maps/)**. For plan-driven configuration that shapes what loads and how, see **[PHASE](../phase/)**.

### 2.3 Heat
A numeric measure of how frequently and recently a piece of content is accessed. High heat = frequently used = loads first. Heat decays over time — unused content cools off. This creates a natural relevance ranking without embeddings.

Heat operates on a three-temperature model:

| Temperature | Range | Behavior |
|------------|-------|----------|
| **Hot** | 8+ | Full content loaded into agent context |
| **Warm** | 3–7 | Summary/breadcrumb loaded |
| **Cold** | 0–2 | Not loaded, but discoverable via search |

Heat events:
- **+2** when explicitly referenced by user
- **+1** when applied in agent action
- **-1** per session if unused (decay)
- **Pin** (user override → hot) and **Kill** (user override → cold)

Thresholds between temperatures are configurable per implementation.

### 2.4 Preload
A session continuation document. Written at the end of a session (exhale), consumed at the start of the next (inhale). Contains: where we left off, what was accomplished, what's next, and what to skip.

### 2.5 Flush
The process of detecting context depletion, extracting essential state, and persisting it to disk before the session ends or continues.

### 2.6 Checkpoint
A lightweight version control snapshot of agent state, taken during flush. Two tracks: the agent's internal state (memory directory, local git) and the project code (local commits, squashed before push).

### 2.7 Promotion
Moving content from a narrow scope (project-level) to a broader scope (user-level or global). Triggered when a pattern proves useful across multiple contexts.

## 3. The Memory Hierarchy

### 3.1 Scope Resolution

Memory exists at multiple scopes. Resolution follows most-specific-first:

```
1. project/.soma/            ← project-level (most specific)
2. workspace/.soma/          ← workspace/parent level
3. ~/.soma/                  ← user-global
```

Settings, content, identity — all merge up the chain. A piece of content with the same name at a narrower scope shadows the broader one.

The rule for where to put things: **"Where does this pattern die?"**
- Dies with this repo → project memory
- Dies with this workspace → workspace memory
- Never dies → global memory

### 3.2 Content Format

All AMP content uses Markdown files with YAML frontmatter. Required fields:

```yaml
---
type: <content-type>
status: draft | active | stable | dormant | archived | deprecated
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

Additional fields are defined by the content type extension (e.g., AMPS defines `heat`, `heat-default`, `breadcrumb`, `depends-on`, etc.).

### 3.3 Two-Tier Loading

Content supports digest-first loading via markers:

```markdown
<!-- digest:start -->
Compressed essential knowledge. 2-5 sentences.
<!-- digest:end -->

Full content below.
```

- **Tight context:** Load only the digest
- **Full context:** Load the entire body

## 4. Directory Structure

```
.soma/                          # AMP-compatible directory
├── memory/
│   ├── preload-next.md         # Session continuation state
│   └── sessions/               # Optional: session logs
├── identity.md                 # Agent identity
├── STATE.md                    # Architecture truth
└── settings.json               # Configuration
```

Content type directories (e.g., `protocols/`, `muscles/`, `skills/`, `automations/`) are defined by the content type extension in use.

## 5. Preload Format

Preloads bridge sessions. Written at exhale, consumed at inhale.

```markdown
---
type: preload
agent: <agent-name>
created: YYYY-MM-DD
session: <session-identifier>
---

## Resume Point
Where we left off. Dense, 1-3 sentences.

## What Shipped
Completed work. Prevents re-doing finished tasks.

## What's Next
Priority-ordered task list for the next session.

## What NOT to Re-read
Files or topics the agent should skip.
```

The "What NOT to Re-read" section is critical — without it, an agent re-reads the same files every session, burning context on already-understood material.

## 6. Flush Pipeline

### 6.1 Trigger
The agent monitors context usage. When usage exceeds a configurable threshold, flush begins.

### 6.2 Steps

```
1. DETECT     — context usage ≥ threshold
2. EXTRACT    — summarize: what was done, what's in progress, what's next
3. PERSIST    — write preload
4. CHECKPOINT — commit both tracks (memory + project)
5. PROMOTE    — if new patterns emerged, write/update content
6. SIGNAL     — indicate continuation is ready
```

### 6.3 What Gets Extracted
Essential state for continuation: current task, decisions made, files modified, unresolved questions, priority order. NOT: full conversation history, file contents, already-known context.

## 7. Checkpoints

### 7.1 Two Tracks

| Track | What | How | Push? |
|-------|------|-----|-------|
| **Internal** | Agent state: preloads, heat, content | Local git inside memory dir | Never |
| **Project** | Code the agent is working on | Checkpoint commits | Squashed before push |

### 7.2 Boot Diff
On inhale, the agent surfaces both tracks via `git diff`, providing immediate orientation without re-reading everything.

## 8. Promotion

### 8.1 Conditions
Content is eligible for promotion when:
- It exists at a narrow scope
- It has been useful across multiple contexts
- Its heat exceeds a threshold
- It is marked as shareable

### 8.2 Direction
Promotion flows upward only. The child decides what to share. The parent does not pull.

## 9. Configuration

```json
{
  "memory": {
    "warm": 3,
    "hot": 8,
    "budget": 2000
  },
  "flush": {
    "threshold": 0.85,
    "autoFlush": true,
    "autoContinue": true
  },
  "checkpoints": {
    "autoCommit": true,
    "diffOnBoot": true
  }
}
```

## 10. Extension Point

AMP defines the memory mechanics. **Content types are defined by extensions.**

An AMP extension:
- Defines named content types (with frontmatter schemas)
- Specifies loading behavior per type (heat-based, on-demand, always-available)
- Specifies directory layout within the AMP structure
- May define evolution paths between types
- May define dependency resolution between content

The reference extension is **[AMPS v1.1](../amps/)**. Navigation and orchestration extensions include **[MAPS](../maps/)** and **[PHASE](../phase/)**.

## 11. Implementation Notes

### 11.1 Requirements
- Filesystem read/write capability
- Markdown parsing (for frontmatter extraction)
- No external services required

### 11.2 Compatibility
Any AI agent framework that can read and write files can implement AMP. The protocol is language-agnostic, LLM-agnostic, and platform-agnostic.

### 11.3 Reference Implementation
[Soma](https://soma.gravicity.ai) — open source.
GitHub: [github.com/meetsoma](https://github.com/meetsoma)

## 12. Anti-Patterns

- **Reaching for a vector DB.** AMP exists because filesystems already work. If the design instinct is to add embeddings or a vector store "for retrieval," the agent is forgetting it already has read/write tools and a hierarchical filesystem.
- **"Filed in my head."** The most common AMP failure isn't structural — it's verbal. When an agent says "I'll remember this," the lesson is not on disk and will not survive the next session. The phrase itself is the tell. (Reinforced in MLX §2.3.)
- **Body files that grow unbounded.** Body discipline matters: under ~3KB for the dispatcher; lazy-load detailed surfaces. A 50KB body.md is a cache-warming tax on every session.
- **Preload as continuation summary.** Preloads are briefings for next-self, not transcripts of last-self. "Continued working on X" is useless; "X is at commit `<sha>`; resume by reading `<path>` line N" is the contract.
- **Heat tracking decay without write-back.** If muscle heat decays in-memory but never persists to disk, the agent forgets what was hot last session.
- **Project-local memory only.** AMP's power comes from the walk-up chain (project → parent → user). Putting everything in `<project>/.soma/` and nothing in `~/.soma/` forfeits cross-project memory.
- **Memory edits without provenance.** When the agent edits a body file or muscle, the edit should carry a SEAMS-style origin marker. Anonymous edits are unrecoverable when something goes wrong.

## 13. Conformance

An AMP-conformant implementation MUST:

1. **Use filesystem as the primary store** — no external database for memory operations. Files are the substrate.
2. **Walk the directory chain at boot** — project → parent → user (`<cwd>/.soma/` → ancestors → `~/.soma/`). Each level contributes.
3. **Produce a preload at exhale** — the preload-out artifact is the cross-session contract. Required fields: resume point, what shipped, read-first targets.
4. **Honor heat tracking and persistence** — if the implementation tracks heat (recency, frequency), it MUST persist heat values across sessions in `state.json` or equivalent.
5. **Support checkpoints** — the agent can save and restore state at arbitrary points within a session.
6. **Make memory inspectable and editable by both agent and human** — every memory file is plain Markdown or simple data; no binary stores.

A conformant tooling layer SHOULD provide:

- Preload assembly + parsing at session boundaries
- Heat tracking with configurable decay
- Body-file slot inspection (sizes, byte costs)
- Walk-up discovery utilities ("which body files apply at this CWD?")
- MLX integration at exhale (§5 of Breath Cycle)

## 14. Attribution

```
This project implements the Agent Memory Protocol (AMP)
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Agent Memory Protocol (AMP) v0.3 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
