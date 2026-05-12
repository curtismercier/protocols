<div align="center">

# Protocols

**Open specifications for AI agent memory, architecture, and identity.**

*No databases. No embeddings. No cloud services. Just files.*

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)
[![Status: Draft](https://img.shields.io/badge/Status-Draft-blue.svg)](#status)

</div>

---

## What This Is

Twelve protocol specifications that define how AI agents can persist memory, navigate knowledge, orchestrate multi-phase work, maintain architecture awareness, discover identity, manage session lifecycles, and project documents across human/agent/tooling surfaces — all without external databases.

These protocols were invented while building **[Soma](https://soma.gravicity.ai)**, an AI coding agent with self-growing memory. They're published here as standalone specifications that any agent framework can implement.

## The Protocols

Twelve specs in five layers. Each spec stands alone; together they form an agent's substrate.

### Substrate

| Protocol | Spec | Description |
|----------|------|-------------|
| **[AMP](./amp/)** | v0.3 | **Agent Memory Protocol** — filesystem-based persistent memory with heat tracking, checkpoints, flush pipeline, and pattern evolution |
| **[AMPS](./amps/)** | v1.1 | **Agent Memory Protocol Stack** — four content types over AMP: Automations, Muscles, Protocols, Scripts |
| **[MAPS](./maps/)** | v0.1 | **My Automation Protocol Scripts** — navigation layer over AMPS. Task-specific paths through knowledge, with progressive phase chains |

### Session lifecycle

| Protocol | Spec | Description |
|----------|------|-------------|
| **[Breath Cycle](./breath-cycle/)** | v0.2 | Inhale (boot) → process (work) → exhale (flush) → rest. Context depletion as design, not bug |
| **[PHASE](./phase/)** | v0.3 | Three-tier brain configuration (T1 prompt-config, T2 phase-folder convention, T3 runtime) + meta-orchestration cycle + delegation-owned worktrees |
| **[MLX](./mlx/)** | v0.1 | **Memory Lane Xtraction** — periodic audit before session close. What's still floating in the agent's head that isn't on disk? |
| **[MLR](./mlr/)** | v0.1 | **Mid-session Learning Review** — in-flight catch. PAUSE → NAME → FILE → RESUME when a pattern emerges during work. Sibling to MLX. |

### Provenance & growth

| Protocol | Spec | Description |
|----------|------|-------------|
| **[SEAMS](./seams/)** | v0.2 | Traceable connections between every artifact. Session seams trace time, document seams trace space |
| **[SEEDS](./seeds/)** | v0.2 | Templates that grow structure with origin provenance and typed scaffolding. Drop a seed in any folder and it tells the agent how to grow |
| **[ATLAS](./atlas/)** | v0.2 | Living system maps with staleness signals, inline maintenance markers, and companion documents |

### Identity

| Protocol | Spec | Description |
|----------|------|-------------|
| **[Identity System](./identity/)** | v0.1 | Contextual identity — agents discover who they are based on where they are |

### Document substrate (sibling repo)

| Protocol | Spec | Description |
|----------|------|-------------|
| **[PRISM](https://github.com/curtismercier/prism)** | v0.1 | **Projected Representations from Inscribed Source Markup** — one Markdown source, three projections (HTML, JSON, agent anchors). Lives in its own repo alongside reference implementations |

### Archived

| Protocol | Spec | Description |
|----------|------|-------------|
| ~~[Git Identity](./_archive/git-identity/)~~ | v0.2 | Folded into Identity System |
| ~~[Capability Model](./three-layer/)~~ | v0.2 | Superseded by AMPS v1.1 |

### How They Fit Together

The protocol family shares DNA — the same letters recombining. They don't just stack linearly. They **cross-sect** through shared letters, with PHASE as the vertical spine:

```
  A M P S           ← what do I know?        (content types)
      H
    M A P S         ← how do I navigate?     (task paths)
      S E A M S     ← how do I trace?        (connections)
  S E E D S         ← how do I grow?         (templates)
      ↑
    PHASE            ← how am I configured?
```

**PHASE is the spine** because when you enter any phase of work, it configures the agent's entire relationship to every other protocol — which AMPS content loads, which MAPS to follow, which SEAMS to trace, which SEEDS to use. Everything flows through PHASE.

The intersection points aren't decorative — they're where the concepts genuinely meet:
- **AMPS** crosses PHASE at **P** — content is loaded per-phase
- **MAPS** crosses PHASE at **A** — every MAP belongs to a phase
- **SEAMS** crosses PHASE at **S** — every seam traces through sessions
- **SEEDS** crosses PHASE at **E** — every seed evolves through experience

The supporting protocols complete the system:

```
┌──────────────────────────────────────────────────┐
│  Identity System     → who am I here?            │
│  Breath Cycle        → session lifecycle          │
│  AMP                 → how do I remember?         │
│  ATLAS               → what does this system      │
│                        look like?                 │
└──────────────────────────────────────────────────┘
```

Before an agent boots, **PHASE** assembles its brain — loading the assigned identity, relevant **AMPS** content, and **MAP** navigation into a system prompt architected for that specific task. The structure was grown from a **SEED** template. The agent boots already shaped for the work, reads the **ATLAS** state, and executes its MAP. Every artifact it creates is stamped with a **SEAM** — traceable back to the session hash that started it. Then it **exhales** — writes preload, commits checkpoints, and refines the next phase's configuration. Each completing agent shapes the next. Over time, the system gets smarter — not just within a session, but across sessions and across agents.

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

These specs are in **draft** (v0.1–v1.1, varying per spec). They describe systems that are implemented and working in Soma, but the spec documents are still being refined. Family-level milestones are tagged (`family-v0.4` at HEAD as of 2026-05-12); per-spec versions live in each spec's frontmatter. See [CHANGELOG.md](./CHANGELOG.md).

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
