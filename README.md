<div align="center">

# Protocols

**Open specifications for AI agent memory, architecture, and identity.**

*No databases. No embeddings. No cloud services. Just files.*

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: Draft](https://img.shields.io/badge/Status-Draft-blue.svg)](#status)

</div>

---

## What This Is

Five protocol specifications that define how AI agents can persist memory, maintain architecture awareness, discover identity, and manage session lifecycles — all without external databases.

These protocols were invented while building **[Soma](https://soma.gravicity.ai)**, an AI coding agent with self-growing memory. They're published here as standalone specifications that any agent framework can implement.

## The Protocols

| Protocol | Spec | Description |
|----------|------|-------------|
| **[AMP](./amp/)** | v0.1 | **Agent Memory Protocol** — filesystem-based persistent memory with muscles, preloads, heat tracking, and promotion |
| **[ATLAS](./atlas/)** | v0.1 | **Architecture Truth Layered Across Stacks** — living system maps with frontmatter standards and hierarchy |
| **[Breath Cycle](./breath-cycle/)** | v0.1 | **Session lifecycle** — inhale (boot) → process (work) → exhale (flush) → rest. Context depletion as design |
| **[Three-Layer Model](./three-layer/)** | v0.1 | **Extensibility** — Extensions (code) + Skills (knowledge) + Rituals (workflows) |
| **[Identity System](./identity/)** | v0.1 | **Contextual identity** — agents discover who they are based on where they are |

### How They Fit Together

```
┌─────────────────────────────────────────────┐
│  Identity System    → who am I here?        │
│  Breath Cycle       → session lifecycle     │
│  AMP                → what do I remember?   │
│  ATLAS              → what does this system  │
│                       look like?            │
│  Three-Layer Model  → how am I extended?    │
└─────────────────────────────────────────────┘
```

An agent boots, discovers its **identity**, **inhales** (loads preload + muscles via **AMP**), reads the **ATLAS** state, works with its **three-layer** capabilities, then **exhales** (writes preload) when context fills up. The cycle repeats.

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

These specs are in **draft v0.1**. They describe systems that are implemented and working in Soma, but the spec documents are still being formalized.

Feedback, questions, and implementation reports welcome — [open an issue](https://github.com/curtismercier/protocols/issues).

## Implement These Protocols

You're free to implement these protocols in your own agent framework. Licensed under **CC BY 4.0** — use them however you want, just credit the source.

```
This project implements the Agent Memory Protocol (AMP)
by Curtis Mercier (https://github.com/curtismercier/protocols)
```

## Reference Implementation

**[Soma](https://soma.gravicity.ai)** is the reference implementation. MIT-licensed, open source.

- GitHub: **[github.com/meetsoma](https://github.com/meetsoma)**
- Website: **[soma.gravicity.ai](https://soma.gravicity.ai)**

## License

**CC BY 4.0** — see [LICENSE](./LICENSE).

Moral rights asserted under the Canadian Copyright Act.

---

<div align="center">

*By [Curtis Mercier](https://github.com/curtismercier) · [Gravicity](https://gravicity.ai)*

</div>
