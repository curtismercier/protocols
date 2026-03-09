# Protocols

> Protocol specifications for AI agent memory, architecture, and identity.  
> By [Curtis Mercier](https://github.com/curtismercier).

## What This Is

A collection of protocol specifications that define how AI agents can persist memory, maintain architecture awareness, discover identity, and manage session lifecycles — all without external databases.

These protocols were invented while building [Soma](https://soma.gravicity.ai), an AI coding agent with self-growing memory. They're published here as standalone specifications that any agent framework can implement.

## The Protocols

| Protocol | Spec | Status | Description |
|----------|------|--------|-------------|
| **AMP** | [amp/](./amp/) | Draft | Agent Memory Protocol — filesystem-based persistent memory with muscles, preloads, heat tracking, and promotion |
| **ATLAS** | [atlas/](./atlas/) | Draft | Architecture Truth Layered Across Stacks — living system maps with frontmatter standards and hierarchy |
| **Three-Layer Model** | [three-layer/](./three-layer/) | Draft | Extensions (code hooks) + Skills (knowledge) + Rituals (workflows) |
| **Breath Cycle** | [breath-cycle/](./breath-cycle/) | Draft | Session lifecycle: inhale (boot) → process (work) → exhale (flush) → rest |
| **Identity System** | [identity/](./identity/) | Draft | Contextual agent identity discovery and inheritance |

## Status

These specs are in **draft**. They describe systems that are implemented and working in [Soma](https://soma.gravicity.ai), but the spec documents are still being formalized.

Feedback, questions, and implementation reports are welcome — open an issue.

## Using These Protocols

You're free to implement these protocols in your own agent framework. The specs are licensed under **CC BY 4.0** — use them however you want, just credit the author.

**Attribution example:**
```
This project implements the Agent Memory Protocol (AMP)
by Curtis Mercier (https://github.com/curtismercier/protocols)
```

## Reference Implementation

[Soma](https://soma.gravicity.ai) is the reference implementation. It's MIT-licensed and open source.

- GitHub: [meetsoma](https://github.com/meetsoma)
- Website: [soma.gravicity.ai](https://soma.gravicity.ai)

## License

CC BY 4.0 — see [LICENSE](./LICENSE).

Moral rights asserted under the Canadian Copyright Act.
