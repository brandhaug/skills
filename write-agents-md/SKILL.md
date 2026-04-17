---
name: write-agents-md
description: Write and maintain AGENTS.md (or CLAUDE.md) files as an Intent Layer — a hierarchy of small, dense context files at semantic boundaries that auto-load as architectural context for agents. Use when the user wants to write, create, generate, scaffold, seed, or sync AGENTS.md / CLAUDE.md files, agent context files, intent nodes, intent layer, or architectural memory for agents; or mentions "AGENTS.md hierarchy", "intent layer", "intent nodes", "agent context", "T-shaped context", "dark room problem". Supports two workflows — `build` (initial capture, leaf-first with SME interview) and `sync` (re-summarize affected nodes after code changes).
---

# Write AGENTS.md

Write and maintain a hierarchy of **Intent Nodes** (`AGENTS.md` / `CLAUDE.md` files) at semantic boundaries. Agents inherit the full ancestor chain automatically — a T-shaped view with broad context at the top and specific detail where work happens.

Reference: https://intent-systems.com/blog/intent-layer

## Quick start

```
/write-agents-md build          # initial capture across the repo
/write-agents-md build src/api  # scoped to a subtree
/write-agents-md sync           # re-summarize nodes affected by recent changes
/write-agents-md sync HEAD~5    # re-summarize nodes affected since a ref
```

## Pre-flight

1. **Detect filename convention** — if repo already has `CLAUDE.md` files, use `CLAUDE.md`; if `AGENTS.md`, use that; otherwise default to `AGENTS.md`.
2. **Detect tooling** — `package.json`, `tsconfig*.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, monorepo manifests (`pnpm-workspace.yaml`, `turbo.json`, `nx.json`). Workspace package roots are strong semantic-boundary candidates.
3. **Confirm scope** — if scope would produce > 30 nodes, surface the plan and ask before proceeding.

## Build workflow

1. **Chunk at semantic boundaries** — not every directory. Chunk when responsibility, patterns, or vocabulary shift. Target 20k–64k tokens per chunk. Strong candidates: workspace packages, domain modules, `src/api`, `src/db`, `src/ui/<feature>`, background jobs, migration dirs.

2. **Leaf-first capture** — start with well-understood subtrees before tangled ones. For each leaf chunk:
   - Read the code in the chunk (entry points, exports, tests).
   - Draft an Intent Node using the schema below.
   - Track open questions in `.intent-layer-questions.md` at the repo root.

3. **SME interview** — after drafting leaves, batch open questions and ask the user. Target: invariants, hidden contracts, anti-patterns, "never do X" rules, historical landmines. Do not invent these — they don't live in the code.

4. **Hierarchical summarize (bottom-up)** — at each parent directory with ≥2 child Intent Nodes, write a parent node that summarizes the **children's nodes**, not the raw code. This is fractal compression — a 2k-token parent may cover 200k tokens below.

5. **Deduplicate at the Least Common Ancestor** — any fact true for multiple children lives in the shallowest node covering all of them. Remove duplicates from the children, leave a one-line reference if useful.

6. **Downlinks, not inlines** — parents link to children and to external docs (ADRs, architecture diagrams). Progressive disclosure: agents follow links only when needed.

7. **Review** — print a tree of created nodes with token counts. Ask the user to spot-check the 2–3 largest and the root before committing.

## Sync workflow

1. **Diff scope** — `git diff --name-only <ref>...HEAD` (default ref: merge-base with default branch).
2. **Map files to nodes** — each changed file belongs to the nearest ancestor Intent Node.
3. **Leaf-first re-summarize** — for each affected leaf node, re-read the chunk and re-draft. If content changes materially, propagate upward: re-summarize parent from its (now-updated) children.
4. **Propose diffs** — show node-by-node diffs. Do **not** auto-commit — intent nodes are reviewed like code.

## Intent Node schema

Every node has six sections. Keep it small but dense — distill the area to minimum high-signal tokens. Aim for 300–2000 tokens per node.

```md
# <Area Name>

## Purpose & Scope
What this area owns. What it explicitly does NOT own.

## Entry Points & Contracts
Public APIs, jobs, events. Invariants and enforcement points.
(e.g. "All writes go through `repo.save()` — direct DB writes bypass audit log.")

## Usage Patterns
Canonical examples for the 2–3 most common tasks here.

## Anti-patterns
Negative examples. "Never call X directly from controllers." "Don't import Y from Z."

## Dependencies & Edges
Related areas (downlinks to child/sibling intent nodes) and external docs (ADRs, diagrams).

## Patterns & Pitfalls
Repeatedly confusing aspects, historical landmines, non-obvious constraints.
```

## Guardrails

- **Small but dense.** A node that looks like README prose is wrong. No marketing, no exhaustive API lists — link to generated docs for those.
- **Never duplicate raw code.** Intent nodes describe *intent*, not implementation. If you're copying code, you're doing it wrong.
- **Facts at the LCA.** Duplicated facts across siblings → hoist to the parent.
- **Semantic boundaries, not directories.** A node at every directory is naive and will drift. Only place nodes where responsibility/vocabulary shifts.
- **Capture invariants invisible in code.** The whole point is institutional knowledge — "never do X", "Y must happen before Z", "this looks dead but isn't".
- **Review like code.** Propose diffs, never auto-commit. Intent nodes are versioned alongside implementation.
- **Leaf-first, always.** Summarizing parents before leaves means parents summarize raw code instead of compressed children — fractal compression breaks.
- **Open questions are first-class.** If the SME can't clarify now, leave the question in the node as `> TODO(intent): <question>` rather than inventing an answer.

## Anti-patterns (of the skill itself)

- Dumping a 15k-token monolithic root `AGENTS.md`. That's the naive approach the blog calls out explicitly.
- One node per directory. Most directories aren't semantic boundaries.
- Writing nodes for humans (tutorials, onboarding prose). Write for tokens — agents read this.
- Copy-pasting function signatures. Use downlinks to generated docs.
- Running sync on every save. Sync belongs at merge / post-commit, not mid-edit.
