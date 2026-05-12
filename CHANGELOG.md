# Changelog

The Gravicity protocols family is twelve specs, each versioned independently. This file is the **family-level** changelog — what landed when, across the whole repo.

Per-spec versions live in each spec's frontmatter (`version:` field). Per-spec history lives in `git log -- <spec-dir>/`.

Format: newest first. Family milestones tagged at HEAD.

---

## family-v0.4 — 2026-05-12

**Polish pass + PHASE v0.3 canonical + MLR v0.1 born.**

| Spec | Version | Change |
|---|---|---|
| PHASE | **0.2 → 0.3** | T1/T2/T3 tier reframe, meta-orchestration cycle (DO → WATCH → DECIDE → CLOSE), delegation-owned worktrees, adoption path. v0.3-draft companion absorbed and archived. |
| MLR | **— → 0.1** | New spec. *Mid-session Learning Review* — sibling to MLX. PAUSE → NAME → FILE → RESUME when patterns emerge mid-work. |
| All specs | frontmatter | `updated:` bumped to 2026-05-12. Cross-reference graph repaired: every spec's `complements:` / `extends:` refreshed for the May 2026 family. |
| Top-level README | revised | PHASE moves to v0.3 canonical, MLR added, spec count goes from eleven to twelve. |

**Tooling shipped alongside:** `protocols-authoring` meta-skill at `~/.agents/skills/protocols-authoring/` with validators (stdlib-only Python): `validate_spec.py`, `audit_graph.py`, `staleness_check.py`.

**Commits:** `39908cd`

---

## family-v0.3 — 2026-05-12 (earlier)

**MLX graduated to standalone spec; PHASE v0.3 first drafted; top-level index stratified.**

| Spec | Version | Change |
|---|---|---|
| MLX | **— → 0.1** | New spec. *Memory Lane Xtraction* — periodic audit before session close. Graduated from in-practice body discipline to standalone protocol. |
| PHASE | (v0.3-draft) | Companion-doc draft of v0.3 added as `phase/v0.3-draft.md`. Absorbed into canonical v0.3 in family-v0.4 below. |
| Top-level README | restructured | Flat 10-row protocols table → 5-layer stratified index. PRISM listed as sibling repo. Git Identity moved to `_archive/`. |

**Commits:** `40c3a6b`

---

## family-v0.2 — 2026-03-28

**Multi-spec version bump cycle: ATLAS/SEAMS/SEEDS/PHASE all to v0.2; AMPS to v1.1 (Scripts rename).**

| Spec | Version | Change |
|---|---|---|
| ATLAS | **— → 0.2** | First versioned spec. Living architecture maps. |
| SEAMS | **0.1 → 0.2** | Session seams + document seams; multi-agent seams. |
| SEEDS | **0.1 → 0.2** | Template variables, origin/provenance, dependency declarations. |
| PHASE | **0.1 → 0.2** | Phase folders + autonomous execution. |
| AMPS | **1.0 → 1.1** | Scripts replaces Skills as the S in AMPS. |
| Git Identity | → archived | Folded into Identity System. Moved to `_archive/`. |

**Commits:** `dd4f0c1`, `1b909df`, `f747129`, plus cross-ref pass

---

## family-v0.1 — 2026-03-16

**The original family.** AMP v0.3, AMPS v1.0, MAPS v0.1, PHASE v0.1, SEAMS v0.1, SEEDS v0.1, Breath Cycle v0.2, Identity v0.1.

Initial published cohort. Each spec born from extracting patterns in Soma's substrate. Tagged `v0.1.0` at the time.

**Commits:** `2d80a0c`, `2aa3325`, `efec587`, `79ec701`

---

## PRISM (sibling repo, separate versioning)

PRISM lives at [`curtismercier/prism`](https://github.com/curtismercier/prism) because it ships reference implementations alongside the spec, not just the spec itself. Its versioning is independent:

| Spec | Version | Date |
|---|---|---|
| PRISM | **0.1** | 2026-05-12 |

---

## Conventions

- **Family milestones** (`family-vX.Y`) are tagged when multiple specs ship a coordinated update.
- **Per-spec versions** live in each spec's frontmatter and are bumped independently.
- **Status: draft** is the resting state — it does not mean unfinished. Specs in this family stay `draft` even when used in production.
- **Archived specs** move to `_archive/` and get a `superseded-by:` field in their frontmatter.
- **Specs are CC BY 4.0.** Reference implementations of those specs may carry different licenses (e.g. PRISM's renderers are MIT).
