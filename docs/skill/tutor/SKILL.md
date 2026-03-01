---
name: concepts-curator
description: >
  Weekday agent that curates a private `concepts` GitHub repo using GitHub MCP
  and shell tools. Scans the repo tree, proposes new nodes (dirs or files) to
  fill knowledge gaps, and interactively refines the owner's interest scope to
  prevent drift. Use this skill whenever the task involves auditing, expanding,
  or scoping a personal knowledge repository or concept tree, especially on a
  recurring/scheduled basis. Runs in Claude Code or Cowork with MCP access.
---

# Concepts Repo Curator Agent

## Purpose

Automated weekday agent on Matt's private `concepts` GitHub repo.

Two jobs:
1. **[A] Gap Detection** — Scan the tree, identify missing concepts, propose new nodes (dirs or leaf files).
2. **[B] Scope Refinement** — Maintain a canonical interest profile so proposals stay on-target.

---

## Environment

- **Runtime**: Claude Code or Cowork
- **Tools**: GitHub MCP, shell (`tree`, `git`), filesystem read/write
- **Repo**: `github.com/matt/<concepts>` (private)
- **Working dir**: clone the repo locally at start of each run

---

## Context: Matt's Domain

Matt works in ML, robotics, and systems engineering:
- Neural network training, multi-GPU (JAX/Flax), distributed/HPC
- Robotics: controls, perception, planning
- Systems: compiler backends, memory optimization, collective comms
- Applied researcher — bridges theory and production

The `concepts` repo is a private knowledge tree: markdown notes and structured
directories representing topics Matt has studied or intends to study.

---

## Phase 0: Bootstrap Check

```python
# Pseudologic at run start
if not exists(".curator/interests.md"):
    run_phase_B(full=True)
else:
    drift = check_drift()  # see Drift Detection below
    if drift:
        run_phase_B(full=False)  # lightweight scope check
run_phase_A()
```

---

## Phase A: Gap Detection

### A.1 — Clone and Scan

```bash
# Clone via GitHub MCP or git
git clone git@github.com:matt/<concepts>.git /tmp/concepts
cd /tmp/concepts

# Scan tree (exclude .git and .curator meta dir)
tree -a --dirsfirst -I '.git|.curator' > /tmp/concepts_tree.txt
```

Parse `concepts_tree.txt`:
- **Directories** = topic clusters (`reinforcement_learning/`, `cuda/`)
- **Leaf files** = specific concepts (`ddpm.md`, `world_models.md`)
- **Depth** = specificity signal
- **Density** = coverage signal (sparse dirs = weak coverage)

### A.2 — Load Interest Profile

```bash
cat .curator/interests.md
```

Extract:
- `core` domains → propose leaf-level nodes
- `peripheral` domains → propose dir-level only
- `out-of-scope` list → hard filter

### A.3 — Gap Analysis

For each **in-scope** domain:

1. List existing nodes (from tree).
2. Generate a set of **canonical concepts** in that domain that a practitioner should know.
3. Diff: canonical − existing = candidate gaps.
4. Filter against `out-of-scope`.
5. Rank candidates by:
   - **Foundational importance** — is it a prerequisite for many other things?
   - **Adjacency** — does it directly connect two existing nodes?
   - **Recency** — is it actively used/cited in current research in Matt's field?

### A.4 — Generate Proposals

Hard cap: **10 proposals per run**. Fewer is better. Quality > quantity.

Output format (write to `.curator/proposals/{YYYY-MM-DD}.md`):

```markdown
# Knowledge Gap Proposals — {DATE}

## High Priority
- [ ] `optimization/shampoo.md`
  - **Anchor**: existing `optimization/adam.md`
  - **Why**: second-order optimizer now standard in large-scale JAX training; direct gap

- [ ] `systems/collective_comms/`
  - **Anchor**: existing `cuda/` and `distributed_training/`
  - **Why**: missing cluster: NCCL, ring-allreduce, tree-reduce — bridges two existing dirs

## Medium Priority
- [ ] `robotics/contact_rich_manipulation.md`
  - **Anchor**: existing `robotics/motion_planning.md` and `rl/`
  - **Why**: gap between planning and RL nodes; active area in sim-to-real

## Exploratory (borderline — confirm scope before actualizing)
- [ ] `math/information_geometry/`
  - **Anchor**: existing `math/stat_mech/`
  - **Why**: adjacent to stat-mech; relevant to natural gradient methods — but peripheral

---
**Filtered out (out-of-scope):** LLM prompt engineering, RAG, web dev tooling

**Drift flag**: {X}% of proposals this week were exploratory → scope check recommended
```

### A.5 — Commit and Open GitHub Issue

```bash
# Write proposal file
git add .curator/proposals/{DATE}.md
git commit -m "curator: knowledge gap proposals {DATE}"
git push
```

Use **GitHub MCP** to open an issue:
- Title: `[curator] {DATE} — {N} knowledge gap proposals`
- Body: full proposal markdown
- Labels: `curator`, `knowledge-gap`
- Assignee: Matt

This is the primary async review mechanism — Matt triages the issue, closes it
when proposals are handled.

### A.6 — Update Run Log

Append to `.curator/run_log.md`:

```
{DATE} | phase=A | proposals={N} | exploratory={X}% | drift={true/false}
```

---

## Phase B: Scope Refinement

### When it runs
- **Full**: first run ever (no `interests.md`)
- **Lightweight**: drift detected (>30% exploratory proposals over last 5 runs)
- **Manual**: Matt opens issue with label `curator-scope-review`

### B.1 — Infer Scope from Tree

Analyze the tree *before* asking Matt anything:

```python
# Heuristics
core     = dirs/files with density > threshold (many nodes, deep hierarchy)
peripheral = dirs/files with low density (few nodes, shallow)
absent   = canonical domains in Matt's field with zero nodes
```

Produce a **draft interest profile** purely from structural analysis.

### B.2 — Open GitHub Issue: Scope Confirmation

Use GitHub MCP to open an issue:
- Title: `[curator] Scope refinement — please confirm interest profile`
- Label: `curator-scope`
- Body:

```markdown
## Inferred Interest Profile

### Core (dense coverage — propose leaf concepts)
- Distributed ML training (JAX, multi-GPU, collective comms)
- Reinforcement learning (model-free, model-based, offline)
- Robotics: perception, planning

### Peripheral (sparse — propose dir-level only)
- Probabilistic ML
- Optimization theory

### Absent (not in tree — assume out-of-scope unless noted)
- NLP / LLMs
- Pure CV (non-robotics)
- Web/systems infra

---
**Reply with corrections.** Format:
- Promote X to core
- Demote Y to peripheral
- Add Z as out-of-scope
- X is absent but should be peripheral
```

### B.3 — Parse Reply and Update Profile

When Matt replies to the issue (or comments), parse corrections and rewrite
`.curator/interests.md`. Use GitHub MCP to fetch issue comments.

> If running in interactive mode (Claude Code session with Matt present),
> ask one scoped question at a time instead of issue-based async flow.
> Example: "You have `cuda/` but no `triton/` — intentional or oversight?"

### B.4 — Interest Profile Format

`.curator/interests.md`:

```markdown
# Interest Profile
Last updated: {DATE}

## Core (leaf-level proposals OK)
- Distributed ML training: collective comms, memory optimization, XLA/compiler
- Reinforcement learning: policy gradient, model-based, offline RL, RLHF
- Robotics: perception, planning, contact manipulation, sim-to-real

## Peripheral (dir-level proposals only)
- Probabilistic ML
- Optimization theory (beyond first-order)

## Out-of-Scope
- LLM prompt engineering / RAG
- Web development
- Pure DevOps unrelated to ML infra

## Depth Rules
- Core: leaf nodes OK (specific algorithms, papers, systems)
- Peripheral: directories only — no leaf proposals unless foundational
- If a concept spans Core + Peripheral, follow Core depth rules

## Explicit Exclusions (Matt-added)
-
```

---

## Drift Detection

After each Phase A run, compute:

```
drift_score = (exploratory_proposals / total_proposals) over last 5 runs
if drift_score > 0.30: flag drift → trigger Phase B (lightweight)
```

Store per-run stats in `.curator/run_log.md` for this calculation.

---

## Scheduling

| Cadence | Action |
|---|---|
| Mon–Fri daily | Phase A — gap detection + proposals |
| Every 10 business days | Phase B — lightweight scope check |
| On drift detection | Phase B — lightweight scope check |
| On `curator-scope-review` issue | Phase B — full refinement |
| On `curator-accept: path/to/node` comment | Create stub file and commit |

---

## Optional: Stub Creation

If Matt comments on a proposal issue:
```
curator-accept: optimization/shampoo.md
```

The agent creates a minimal stub:

```markdown
# Shampoo

> Stub — to be expanded.

**Category**: optimization / second-order methods
**Related**: [[adam]], [[distributed_training]]
**References**:
-
```

Commits and pushes. Closes that item in the issue checklist.

---

## Guardrails

- **Never create actual concept files without explicit `curator-accept`** — proposals only.
- **Max 10 proposals per run.** If more candidates exist, rank and cut.
- **Every proposal must have an anchor** — an existing node it connects to.
- **Respect `out-of-scope` strictly.** Borderline → exploratory tier + ask, not silently include.
- **No motivational filler.** Terse rationale only: anchor + one-line reason.
- **Don't re-propose** nodes already in open curator issues (check before generating).

---

## GitHub MCP Operations Used

| Operation | When |
|---|---|
| `get_file_contents` | Read `.curator/interests.md`, `run_log.md` |
| `create_or_update_file` | Write proposals, run log, interests profile |
| `create_issue` | Daily proposals, scope review |
| `list_issues` | Check for open curator issues (dedup) |
| `get_issue_comments` | Parse Matt's scope refinement replies |
| `add_issue_comment` | Acknowledge stub creation |
| `create_pull_request` | (optional) for stub batch commits |
