---
type: spec
status: draft
version: 0.2.0
created: 2026-03-10
updated: 2026-03-10
author: Curtis Mercier
license: CC BY 4.0
---

# Agent Memory Protocol (AMP) — Specification v0.2

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

### 2.3 Protocol
A behavioral rule that shapes how the agent acts. Frontmatter standards, session lifecycle, identity conventions. Where muscles are *learned patterns*, protocols are *enforced rules* — skipping them causes mistakes. Protocols carry heat like muscles, but their loading behavior differs: hot protocols inject their full body into the system prompt, warm ones appear as one-line breadcrumbs, cold ones stay discoverable but dormant.

### 2.4 Preload
A session continuation document. Written at the end of a session (exhale), consumed at the start of the next session (inhale). Contains: where we left off, what was accomplished, what's next, and what to skip.

### 2.5 Heat
A numeric measure of how frequently and recently a memory is accessed. High heat = frequently used = loads first. Heat decays over time — unused memories cool off. This creates a natural relevance ranking without embeddings.

Heat operates on a three-temperature model:

| Temperature | Behavior | When |
|------------|----------|------|
| **Hot** | Full content loaded into agent context | Frequently used, recently referenced |
| **Warm** | Breadcrumb (1-2 sentence summary) loaded | Occasionally used, still relevant |
| **Cold** | Not loaded, but discoverable via search | Unused, decayed, or niche |

Heat events:
- **+2** when explicitly referenced by user
- **+1** when applied in agent action
- **-1** per session if unused (decay)
- **Pin** (user override → hot) and **Kill** (user override → cold)

Thresholds between temperatures are configurable per implementation.

### 2.6 Promotion
Moving a memory from a narrow scope (project-level) to a broader scope (user-level or global). Triggered when a pattern proves useful across multiple contexts.

### 2.7 Flush
The process of detecting context depletion, extracting essential state, and persisting it to disk before the session ends or continues.

### 2.8 Checkpoint
A lightweight version control snapshot of agent state, taken during flush. Two tracks: the agent's internal state (`.soma/` directory, local git) and the project code (local commits, squashed before push). Checkpoints provide session-over-session diffs at boot, answering: "what changed since I was last here?"

## 3. The Memory Hierarchy

AMP recognizes that knowledge has different characteristics. Not everything is a muscle.

### 3.1 Knowledge Types

| Type | Nature | Example | Loading |
|------|--------|---------|---------|
| **Muscle** | Learned pattern from experience | "Always run tests after file removal" | By heat, digest-first |
| **Protocol** | Behavioral rule, enforced | "Every file gets frontmatter" | By heat, hot=full/warm=breadcrumb |
| **Skill** | Domain knowledge, on demand | "How to design SVG logos" | When task matches |
| **Script** | Automated enforcement | Audit scripts, hooks | Executed, not loaded into prompt |
| **Preload** | Session continuation state | "Where we left off" | Always on inhale |
| **Identity** | Who the agent is here | Project context, personality | Always on inhale |

### 3.2 Pattern Evolution

Knowledge matures through use. This is the natural lifecycle:

```
observation (noticed gap, repeated action)
  ↓ seen 2+ times → write it down
muscle (learned pattern, markdown file)
  ↓ loaded repeatedly, applied automatically
muscle memory (subconscious — agent applies without thinking)
  ↓ crystallizes based on nature
protocol   → behavioral rule (how to be)
skill      → domain expertise (how to know)
ritual     → multi-step workflow (how to do)
script     → automated enforcement (how to enforce)
```

Not every pattern climbs the full ladder. Some stay muscles forever. A pattern becomes whatever it naturally is — a testing discipline becomes a protocol, a design technique becomes a skill, a release workflow becomes a ritual, and a protocol whose checks can be automated becomes a script.

**The script doesn't replace the protocol.** A script enforces the *how*. The protocol explains the *why*. An agent with the script but without the protocol can run the automation but can't reason about edge cases, adapt to new situations, or know when to break the rule.

### 3.3 Skills: The Universal Building Block

Skills are unique in the hierarchy — they're **framework-agnostic knowledge sets**. A skill written for any agent framework works in an AMP-compatible system without modification. Skills are plug-and-play.

What makes AMP different: **muscles and protocols refine skills**. A logo design skill teaches the technique. A muscle learns *your* logo preferences. A protocol enforces *your* brand standards. The skill provides raw expertise; AMP's layers personalize and improve it through use — without the user ever asking.

## 4. Directory Structure

```
.soma/                          # AMP-compatible directory
├── memory/
│   ├── muscles/                # Learned patterns
│   │   ├── git-workflow.md
│   │   └── api-patterns.md
│   ├── preload-next.md         # Session continuation state
│   └── sessions/               # Optional: session logs
├── protocols/                  # Behavioral rules
│   ├── breath-cycle.md
│   └── frontmatter-standard.md
├── skills/                     # Domain knowledge
│   └── logo-design/
│       └── SKILL.md
├── scripts/                    # Automated enforcement
│   └── audit.sh
├── templates/                  # Scaffolding for new projects
├── identity.md                 # Agent identity (see Identity System spec)
├── STATE.md                    # Architecture truth (see ATLAS spec)
└── settings.json               # Configuration
```

### 4.1 Memory Hierarchy (Scope Resolution)

Memory exists at multiple scopes. Resolution follows most-specific-first:

```
1. project/.soma/                ← project-level (most specific)
2. workspace/.soma/              ← workspace/parent level
3. ~/.soma/                      ← user-global
```

Settings, protocols, muscles, identity — all merge up the chain. Project overrides workspace overrides global. A muscle with the same name at a narrower scope shadows the broader one.

The rule for where to put things: **"Where does this pattern die?"**
- Dies with this repo → project `.soma/`
- Dies with this workspace → workspace `.soma/`
- Never dies → global `~/.soma/`

## 5. Muscle Format

Muscles are Markdown files with YAML frontmatter.

### 5.1 Required Frontmatter

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

### 5.2 Body Structure

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

### 5.3 Heat Rules

- When a muscle is loaded into agent context: `loads += 1`
- Heat rises on use, decays on neglect (see §2.5)
- Minimum heat for loading: implementation-defined
- Heat is informational — it guides the agent's memory loader, not a hard rule

## 6. Protocol Format

Protocols share the muscle format but serve a different purpose.

### 6.1 Required Frontmatter

```yaml
---
type: protocol
name: <protocol-name>
status: active
heat-default: <cold|warm|hot>     # starting temperature
applies-to: [<signal>, ...]       # domain scoping (empty = always)
breadcrumb: "<1-2 sentence TL;DR>" # injected when warm
---
```

### 6.2 Body Structure

```markdown
# <Protocol Name>

## TL;DR
<3-5 bullets. The essential rules. Enough to follow if this is all you read.>

## Rule
<Full protocol. The reasoning, the edge cases, the examples.>

## When to Apply
<Triggers: when should the agent follow this?>

## When NOT to Apply
<Boundaries: when should the agent ignore this?>
```

### 6.3 Loading Behavior

Protocols load based on heat:
- **Hot** → full body injected into system prompt
- **Warm** → only the `breadcrumb` field injected (one line)
- **Cold** → name only, discoverable via search

The `applies-to` field enables domain scoping. A protocol with `applies-to: [typescript]` only loads in TypeScript projects. A protocol with `applies-to: [always]` or an empty array loads everywhere.

## 7. Preload Format

Preloads bridge sessions. Written at exhale, consumed at inhale.

### 7.1 Required Structure

```markdown
---
type: preload
agent: <agent-name>
created: <YYYY-MM-DD>
session: <session-identifier>
---

# Preload — <Session Description>

## Resume Point
<Where we left off. Dense, 1-3 sentences.>

## What Shipped
<Completed work. Prevents re-doing finished tasks.>

## What's Next
<Priority-ordered task list for the next session.>

## What NOT to Re-read
<Files, topics, or docs the agent should skip.>

## Key Files
<Quick reference table of important paths.>
```

### 7.2 Negative Context

The "What NOT to Re-read" section is critical. Without it, an agent that has processed a large codebase will re-read the same files every session, burning context on already-understood material.

### 7.3 Lifecycle

Only one preload exists at a time. Each session overwrites the previous. Historical preloads can be archived to `sessions/` if desired.

## 8. Checkpoints

Checkpoints add version control to the memory layer, providing session-over-session visibility.

### 8.1 Two Tracks

| Track | What | How | Push? |
|-------|------|-----|-------|
| **Internal** (`.soma/`) | Agent state: preloads, heat, muscles | Own local git repo inside `.soma/` | Never |
| **Project** | Code the agent is working on | Checkpoint commits on working branch | Squashed before push |

### 8.2 Internal Track

The `.soma/` directory has its own git repo. On every exhale:
1. Stage all changes: `git add -A`
2. Commit with timestamp: `checkpoint: <ISO timestamp>`

On next inhale, `git diff HEAD~1` surfaces what changed — protocol heat shifts, new muscles, preload deltas. This feeds into the boot sequence as session context.

### 8.3 Project Track

Project code uses lightweight local commits during work sessions. These are never pushed raw — they're squashed into clean, meaningful commits before shipping. This gives the agent checkpoint-level granularity for session diffs without polluting the published git history.

### 8.4 Boot Diff

On inhale, the agent surfaces both tracks:

```
── .soma changes ──
  modified: STATE.md (3 lines)
  modified: memory/preload-next.md
  added: protocols/new-protocol.md

── project changes (since checkpoint) ──
  modified: src/core/settings.ts (+12 -3)
  new file: src/components/Filter.tsx
```

This gives immediate orientation without re-reading everything.

## 9. Flush Pipeline

Flush is the process of persisting state when context is depleted.

### 9.1 Trigger

The agent monitors context usage. When usage exceeds a configurable threshold, flush begins.

### 9.2 Steps

```
1. DETECT    — context usage ≥ threshold
2. EXTRACT   — summarize: what was done, what's in progress, what's next
3. PERSIST   — write preload
4. CHECKPOINT — commit both tracks
5. PROMOTE   — if new patterns emerged, write/update muscles
6. SIGNAL    — indicate continuation is ready
```

### 9.3 What Gets Extracted

Essential state for continuation:
- Current task and progress
- Decisions made this session
- Files modified
- Unresolved questions
- Priority order for next session

What does NOT get extracted:
- Full conversation history
- File contents (they exist on disk)
- Already-known context (in muscles or preloads)

## 10. Promotion

Promotion moves a memory from a narrow scope to a broader one.

### 10.1 Conditions

A muscle is eligible for promotion when:
- It exists at project scope
- It has been useful across multiple contexts
- Its heat exceeds a threshold
- It has `scope: shared`

### 10.2 Mechanism

```
project-A/.soma/memory/muscles/pattern.md
    ↓ promote
~/.soma/memory/muscles/pattern.md
```

The project-level copy can be kept (for overrides) or removed.

### 10.3 Automatic vs Manual

- **Manual promotion** (recommended for initial implementations): user or agent explicitly copies a muscle to a broader scope.
- **Automatic promotion** (future): agent detects cross-project patterns and promotes autonomously.

## 11. Configuration

AMP configuration lives in `settings.json`.

```json
{
  "memory": {
    "flowUp": false,
    "flowFilter": null,
    "flowMode": "copy",
    "autoPromote": false
  },
  "flush": {
    "threshold": 0.85,
    "autoFlush": true,
    "autoContinue": true
  },
  "checkpoints": {
    "soma": { "autoCommit": true },
    "project": { "style": "commit", "autoCheckpoint": false },
    "diffOnBoot": true
  }
}
```

### 11.1 Flow Settings

When multiple AMP instances exist in a hierarchy, memories can flow upward.

- **`flowUp`**: Whether this instance shares muscles with its parent (default: `false`)
- **`flowFilter`**: Rules for which muscles flow up
- **`flowMode`**: `"copy"` or `"reference"`
- **Flow is child-controlled.** The child decides what to share. The parent does not pull.

## 12. Implementation Notes

### 12.1 Requirements
- Filesystem read/write capability
- Markdown parsing (for frontmatter extraction)
- No external services required

### 12.2 Compatibility
Any AI agent framework that can read and write files can implement AMP. The protocol is language-agnostic, LLM-agnostic, and platform-agnostic.

### 12.3 Reference Implementation
[Soma](https://soma.gravicity.ai) — open source.
GitHub: [github.com/meetsoma](https://github.com/meetsoma)

## 13. Attribution

When implementing AMP, include attribution:

```
This project implements the Agent Memory Protocol (AMP)
by Curtis Mercier (https://github.com/curtismercier/protocols)
Licensed under CC BY 4.0
```

---

*Agent Memory Protocol (AMP) v0.2 — Curtis Mercier — CC BY 4.0*
*Reference implementation: Soma (soma.gravicity.ai)*
