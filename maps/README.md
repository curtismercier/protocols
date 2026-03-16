---
type: spec
status: draft
version: 0.1.0
created: 2026-03-16
updated: 2026-03-16
author: Curtis Mercier
license: CC BY 4.0
extends: amps/1.0
---

# MAPS — My Automation Protocol Scripts v0.1

> The navigation layer over AMPS. Before starting any task, check if a MAP exists. A MAP tells you which muscles to load, which protocols to follow, which scripts to run, and in what order.

*Extends: [AMPS v1.0](../amps/) (Agent Memory Protocol Stack)*

## 1. The Problem

AMPS gives an agent the raw materials — automations, muscles, protocols, skills. But materials without a plan produce inconsistent results. One session the agent remembers to run the tests. Next session it forgets. One session it checks the docs. Next session it skips that and ships drift.

This happens because AMPS content is **loaded by heat, not by task**. Heat reflects what's been used recently. But what's been used recently isn't necessarily what's needed *right now*. A refactoring session needs `incremental-refactor` hot — but if the agent spent the last three sessions writing blog posts, that muscle is cold.

MAPS fix this. A MAP is a tested path through AMPS for a specific task. It's not documentation. It's not a checklist someone else wrote. It's the agent's own navigation — built from experience, refined when it fails, trusted because it works.

## 2. What a MAP Is

A MAP is a Markdown file with YAML frontmatter that declares:

1. **What AMPS content to load** — muscles, protocols, scripts by name
2. **What steps to follow** — ordered, verifiable, referencing AMPS content
3. **What's still broken** — a living Gaps section that gets smarter with use

```yaml
---
type: map
name: refactor
status: active
triggers: [refactor, extract, move, split]
reads:
  muscles: [incremental-refactor, code-navigator]
  protocols: [quality-standards, workflow]
  scripts: [soma-refactor.sh, soma-code.sh]
requires: [identified extraction target, validated plan]
produces: [refactored code, updated tests]
---
```

### 2.1 MAPS Reference, Not Repeat

A MAP points to AMPS content by name. It says "read `ship-cycle` before step 3" and trusts the muscle to provide the knowledge. It never inlines muscle content — that creates two copies that drift.

### 2.2 Every MAP Has Gaps

```markdown
## Gaps

- No automated test for route catalog sync (noticed 2026-03-16)
- Step 6 assumes git hooks are set up — should verify first
```

Gaps are discovered during use. They're the MAP getting smarter. Fix critical gaps immediately. Log non-critical ones for next refinement.

### 2.3 MAPS Have Triggers

```yaml
triggers: [refactor, extract, move, split]
```

Keywords the agent recognizes. When a task matches a trigger, the MAP surfaces. This is how MAPS get discovered without the agent memorizing them all.

## 3. MAP Lifecycle

### 3.1 Discovery

Before beginning any task, check for an existing MAP. If one exists, read it. Follow it. If it's wrong or incomplete, fix it after.

### 3.2 Creation

The first time you do something, just do it. The second time, you'll notice "I did this before." That's the signal. Build a MAP. Don't wait for the third time — by then you've already repeated the mistake the MAP would have prevented.

### 3.3 Refinement

When a MAP is wrong, update it. Don't delete and rewrite. The history of what changed tells you what the process actually needs — not what you thought it needed when you first wrote it.

### 3.4 Tracking

```yaml
last-run: 2026-03-16
runs: 3
estimated-turns: 15-30
```

MAPs track how often they're used and how long they take. `estimated-turns` refines over time as the agent learns actual cost. A MAP with zero runs is untested — treat it with appropriate skepticism.

## 4. Phase Chains

MAPs can link into chains for multi-phase work:

```yaml
next-map: runtime-p1-interface
refine-after: runtime-p0-plan
```

- `next-map` — what MAP follows this one
- `refine-after` — which MAP's completion triggers refinement of this one

### 4.1 Progressive Detail

In a chain, earlier MAPs are more detailed than later ones. This is intentional.

| Phase | Detail Level | Why |
|-------|---|---|
| Phase 0 | Full — exact steps, verification | You know this work |
| Phase 1 | Structural — steps clear, details TBD | Phase 0 will fill gaps |
| Phase 2 | Ordered — sequence known, methods TBD | Phase 1 will reveal patterns |
| Phase N | North star — vision only | Too far out to plan |

Don't fight the fog. Later phases are unclear because earlier ones haven't happened yet. The chain trusts that future-you will be smarter after completing each phase.

### 4.2 Cascading Refinement

When an agent completes Phase N, it refines Phase N+1's MAP — adding steps, fixing gaps, adjusting the `reads` list based on what was actually needed. This connects to [PHASE](../phase/) for full agent configuration handoff.

## 5. Relationship to AMPS

| AMPS Layer | How MAPS Use It |
|------------|-----------------|
| **A**utomations | MAPS *are* manual automations. Automated MAPS become `trigger: event/cron` automations. |
| **M**uscles | MAPS load muscles for context. "Read ship-cycle before shipping." |
| **P**rotocols | MAPS follow protocols. "Follow quality-standards during review." |
| **S**cripts | MAPS run scripts. "Run soma-refactor.sh routes for audit." |

MAPS are the connective tissue. They don't add new knowledge — they organize existing knowledge into a path.

## 6. Format

```markdown
---
type: map
name: <name>
status: active | draft | complete
created: YYYY-MM-DD
updated: YYYY-MM-DD
triggers: [keyword1, keyword2]
reads:
  muscles: [muscle-names]
  protocols: [protocol-names]
  scripts: [script-names]
last-run: YYYY-MM-DD | null
runs: 0
estimated-turns: 5-10
requires: [preconditions]
produces: [outputs]
next-map: <name> | null
refine-after: <name> | null
---

# MAP Name

One-line description.

## Steps

Numbered steps. Reference AMPS content by name.
Each step should be verifiable — "run X, expect Y."

## Gaps

Living list of what's missing or broken.
```

## 7. Anti-Patterns

- **MAP that inlines muscle content** — two copies that drift. Reference by name.
- **MAP with no Gaps section** — never been stress-tested. Add `## Gaps` even if empty.
- **MAP that's never updated** — stale MAP is worse than no MAP. False confidence.
- **Skipping the MAP because "I know this"** — you knew it last session too. You missed step 4.
- **MAP for a one-time task** — overkill. MAPS are for recurring processes.
- **Fully detailed Phase 3 MAP from Phase 0** — over-planning. Later phases get refined by earlier completions.

## 8. Relationship to PHASE

MAPS provide the navigation. [PHASE](../phase/) provides the agent configuration. Together:

- The MAP says *what to do* — steps, tools, AMPS content
- PHASE says *who to be* — which protocols are hot, which muscles load, what identity to assume
- The completing agent refines both for the next phase

A MAP without PHASE works fine — the agent follows steps with whatever brain configuration it has. PHASE without MAPS also works — the agent gets configured but navigates freely. Together they're strongest.

---

*MAPS v0.1 — Curtis Mercier — CC BY 4.0*
*Extends: AMPS v1.0 (Agent Memory Protocol Stack)*
*Reference implementation: Soma (soma.gravicity.ai)*
