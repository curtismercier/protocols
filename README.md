<div align="center">

# Protocols

**Open specifications for AI agent memory, architecture, and identity.**

*No databases. No embeddings. No cloud services. Just files.*

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: Draft](https://img.shields.io/badge/Status-Draft-blue.svg)](#status)

</div>

---

## What This Is

Nine protocol specifications that define how AI agents can persist memory, navigate knowledge, orchestrate multi-phase work, maintain architecture awareness, discover identity, and manage session lifecycles — all without external databases.

These protocols were invented while building **[Soma](https://soma.gravicity.ai)**, an AI coding agent with self-growing memory. They're published here as standalone specifications that any agent framework can implement.

## The Protocols

| Protocol | Spec | Description |
|----------|------|-------------|
| **[AMP](./amp/)** | v0.3 | **Agent Memory Protocol** — filesystem-based persistent memory with heat tracking, checkpoints, flush pipeline, and pattern evolution |
| **[AMPS](./amps/)** | v1.0 | **Agent Memory Protocol Stack** — four content types that extend AMP: Automations, Muscles, Protocols, Skills |
| **[MAPS](./maps/)** | v0.1 | **My Automation Protocol Scripts** — navigation layer over AMPS. Task-specific paths through knowledge, with progressive phase chains |
| **[PHASE](./phase/)** | v0.1 | **Prompt Handoff for Agent Session Evolution** — plan-driven brain configuration and cascading refinement across phases |
| **[ATLAS](./atlas/)** | v0.1 | **Architecture Truth Layered Across Stacks** — living system maps with frontmatter standards and hierarchy |
| **[Breath Cycle](./breath-cycle/)** | v0.2 | **Session lifecycle** — inhale (boot) → process (work) → exhale (flush) → rest. Context depletion as design constraint |
| **[Identity System](./identity/)** | v0.1 | **Contextual identity** — agents discover who they are based on where they are |
| **[Git Identity](./git-identity/)** | v0.2 | **Multi-repo attribution** — identity zones, path-based resolution, agent vs human commits |

| Archived | | |
|----------|------|-------------|
| ~~[Capability Model](./three-layer/)~~ | v0.2 | Superseded by **AMPS v1.0** |

### How They Fit Together

```
┌──────────────────────────────────────────────────┐
│  Identity System     → who am I here?            │
│  Breath Cycle        → session lifecycle          │
│  AMP                 → how do I remember?         │
│  AMPS                → what do I remember?        │
│                        (Skills, Muscles,          │
│                         Protocols, Automations)   │
│  MAPS                → how do I navigate?         │
│  PHASE               → how do I evolve?           │
│  ATLAS               → what does this system      │
│                        look like?                 │
└──────────────────────────────────────────────────┘
```

The protocol family shares DNA — the same letters recombining at each layer:

```
AMP  → AMPS → MAPS → PHASE
store   define  navigate  orchestrate
```

Before an agent boots, **PHASE** assembles its brain — loading the assigned identity, relevant **AMPS** content, and **MAP** navigation into a system prompt architected for that specific task. The agent then boots already shaped for the work, reads the **ATLAS** state, and executes its MAP. Then it **exhales** — writes preload, commits checkpoints, and refines the next phase's configuration. Each completing agent shapes the next. Over time, the system gets smarter — not just within a session, but across sessions and across agents.

## The Core Idea

Most frameworks treat agent memory as a retrieval problem — vector databases, embeddings, RAG pipelines.

These protocols take a different approach: **the agent reads and writes plain files.** Like a human with a notebook.

This works because:
- LLMs are excellent at reading and following written instructions
- Filesystem operations are universal — any language, any OS, any framework
- No infrastructure required
- The agent can inspect and curate its own memory
- Humans can read and edit it too (it's just Markdown)

## Status

These specs are in **draft** (v0.1–v0.2). They describe systems that are implemented and working in Soma, but the spec documents are still being refined.

Feedback, questions, and implementation reports welcome — [open an issue](https://github.com/curtismercier/protocols/issues).

## Implement These Protocols

You're free to implement these protocols in your own agent framework. Licensed under **CC BY 4.0** — use them however you want, just credit the source.

```
This project implements the Agent Memory Protocol (AMP)
by Curtis Mercier (https://github.com/curtismercier/protocols)
```

## Reference Implementation

**[Soma](https://soma.gravicity.ai)** is the reference implementation. Open source.

- GitHub: **[github.com/meetsoma](https://github.com/meetsoma)**
- Website: **[soma.gravicity.ai](https://soma.gravicity.ai)**

## License

**CC BY 4.0** — see [LICENSE](./LICENSE).

Moral rights asserted under the Canadian Copyright Act.

---

<div align="center">

*By [Curtis Mercier](https://github.com/curtismercier) · [Gravicity](https://gravicity.ai)*

</div>
