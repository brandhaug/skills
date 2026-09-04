---
name: write-agents-md
description: Write and maintain AGENTS.md (or CLAUDE.md) files as an Intent Layer — a hierarchy of small, dense context files at semantic boundaries that auto-load for agents. Use when the user wants to write, generate, sync, audit, or prune AGENTS.md / CLAUDE.md files, agent context, intent nodes, or an intent layer. Two workflows — `build` (initial capture, leaf-first with SME interview) and `sync` (full-tree reconciliation that adds, modifies, and removes content under a hard line budget).
---

# Write AGENTS.md

Write and maintain a hierarchy of **Intent Nodes** (`AGENTS.md` / `CLAUDE.md` files) at semantic boundaries. Agents inherit the full ancestor chain automatically — a T-shaped view with broad context at the top and specific detail where work happens.

Reference: https://intent-systems.com/blog/intent-layer

## Quick start

```
/write-agents-md build          # initial capture across the repo
/write-agents-md build src/api  # scoped to a subtree
/write-agents-md sync           # reconcile every node against current code (add / modify / remove)
/write-agents-md sync HEAD~5    # same, but re-read code only for nodes touched since a ref
```

## Line budget

Frontier models follow roughly 150–200 instructions reliably, and the harness system prompt already spends ~50 of those. Every line in a node competes for that fixed budget, and an agent working in a leaf loads the **whole ancestor chain**.

| Scope | Target | Ceiling |
|---|---|---|
| Root node | 40 lines | 60 lines |
| Any other node | 60 lines | 300 lines |
| Deepest root→leaf chain, summed | 200 lines | 300 lines |

Measure, never estimate:

```sh
find . \( -name AGENTS.md -o -name CLAUDE.md \) -not -path '*/node_modules/*' | xargs wc -l | sort -n
```

Recommended hard stop: a pre-commit hook or CI step that fails on any node over its ceiling. That takes the judgment out of the agent's hands.

## Pre-flight

1. **Detect filename convention** — if repo already has `CLAUDE.md` files, use `CLAUDE.md`; if `AGENTS.md`, use that; otherwise default to `AGENTS.md`.
2. **Detect tooling** — `package.json`, `tsconfig*.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, monorepo manifests (`pnpm-workspace.yaml`, `turbo.json`, `nx.json`). Workspace package roots are strong semantic-boundary candidates.
3. **Inventory documentation** — `docs/`, `adr/`, `rfcs/`, `ARCHITECTURE.md`, `CONTRIBUTING.md`, design docs, diagrams, and any `*.md` that is not an intent node. These are evidence for build and link targets for downlinks. Note them; do not read them all yet.
4. **Confirm scope** — if scope would produce > 30 nodes, surface the plan and ask before proceeding.

## Build workflow

1. **Chunk at semantic boundaries** — not every directory. Chunk when responsibility, patterns, or vocabulary shift. Target 20k–64k tokens per chunk. Strong candidates: workspace packages, domain modules, `src/api`, `src/db`, `src/ui/<feature>`, background jobs, migration dirs.

2. **Leaf-first capture** — start with well-understood subtrees before tangled ones. For each leaf chunk:
   - Read the code in the chunk (entry points, exports, tests).
   - Read the docs from the inventory that cover this chunk. ADRs and design docs hold the "why" that code cannot show. **Code wins on "what"**: a doc claim the code no longer supports is an interview question for the SME, not node content.
   - Draft an Intent Node using the schema below. Omit any section with nothing to say.
   - Track open questions in `.intent-layer-questions.md` at the repo root.

3. **SME interview** — after drafting leaves, batch open questions and ask the user. Target: invariants, hidden contracts, anti-patterns, "never do X" rules, historical landmines. Do not invent these — they don't live in the code.

4. **Hierarchical summarize (bottom-up)** — at each parent directory with ≥2 child Intent Nodes, write a parent node that summarizes the **children's nodes**, not the raw code. This is fractal compression — a 2k-token parent may cover 200k tokens below.

5. **Deduplicate at the Least Common Ancestor** — any fact true for multiple children lives in the shallowest node covering all of them. Remove duplicates from the children, leave a one-line reference if useful.

6. **Downlinks, not inlines** — parents link to children and to external docs (ADRs, architecture diagrams). Point to existing docs from the inventory first. Only when nothing exists does long material — setup walkthroughs, schema references, runbooks — go in `agent_docs/<topic>.md`. Every downlink gets one line saying when to follow it. Prefer `file:line` pointers to copied snippets; copies go stale.

7. **Root node last** — the root loads in every session, so it is the smallest node and the only one every agent is guaranteed to read. It carries exactly: a codebase map (what the apps and packages are, what each is for — critical in monorepos), universal build/test/verify commands, tool preferences (`bun` not `node`), and downlinks. Nothing that applies to only one subtree.

8. **Review** — run the line-count command. Print the tree with per-node counts and the deepest chain total. Refuse anything over ceiling. Ask the user to spot-check the root and the 2–3 largest nodes before committing.

## Sync workflow

Sync always walks **every** node. Scoping sync to changed files lets untouched nodes drift forever, and stale context misleads agents worse than missing context does. Treat each node as a draft to revise, not a floor to add on top of.

1. **Classify nodes** — `git diff --name-only <ref>...HEAD` (default ref: merge-base with default branch). Nodes whose chunk contains changed files are **hot**; the rest are **cold**.
2. **Audit every node, section by section** — flag:
   - **Stale references** — files, functions, modules, or symbols that no longer exist or were renamed. Verify each path and symbol with `ls` / `grep`; this is cheap and runs on cold nodes too.
   - **Drifted claims** — invariants, contracts, or "always/never" rules the code no longer enforces.
   - **Dead anti-patterns** — warnings about pitfalls that are no longer reachable.
   - **Superseded guidance** — usage patterns replaced by a newer canonical approach.
   - **Broken or stale downlinks** — the linked doc is gone, or describes a design the code has left behind. Fix the node; do not edit the doc. If the doc itself is stale, tell the user in the summary.
   - **Hoist/sink violations** — facts duplicated across siblings (hoist to LCA) or that only apply to one child (sink down).
   - **Style rules** — formatting, naming, import order. Delete on sight; linters and hooks own these.
   - **Bloat** — sections that have grown into prose, tutorials, or exhaustive lists; compress or move to `agent_docs/`.
3. **Re-read code for hot nodes only** — cold nodes get the reference checks above; hot nodes get a full re-read of the chunk and a fresh draft with additions, edits, **and deletions**. If a hot node changes materially, re-audit its parent against the updated children.
4. **Enforce the budget** — run the line-count command. Sync must not grow a node by default: every added line is paid for by a removed or compressed line. A node may grow only if the user approves a stated reason (new subsystem, new SME invariant). Any node or chain over ceiling blocks the proposal until split or pruned.
5. **Propose diffs** — node-by-node with `+ added`, `~ modified`, `- removed` called out, plus before/after line count per node and the chain total. For each removal, state *why* in one line (e.g. "function `foo` deleted in abc123", "rule no longer enforced — see `bar.ts:42`"). Do **not** auto-commit — intent nodes are reviewed like code.

When uncertain about a removal, leave `> TODO(intent): verify — <reason>` rather than silently keeping stale content or deleting load-bearing context.

## Intent Node schema

Seven sections. Use only the ones with real content — an empty heading is filler, and a three-section leaf is fine.

```md
# <Area Name>

## Purpose & Scope
What this area owns. What it explicitly does NOT own.

## Entry Points & Contracts
Public APIs, jobs, events. Invariants and enforcement points.
(e.g. "All writes go through `repo.save()` — direct DB writes bypass audit log.")

## Commands & Verification
How to build, test, and verify work here. Tool preferences. Only what differs from the parent node.

## Usage Patterns
Canonical examples for the 2–3 most common tasks here.

## Anti-patterns
Negative examples. "Never call X directly from controllers." "Don't import Y from Z."

## Dependencies & Edges
Related areas (downlinks to child/sibling intent nodes) and external docs (ADRs, diagrams, `agent_docs/`).

## Patterns & Pitfalls
Repeatedly confusing aspects, historical landmines, non-obvious constraints.
```

## Guardrails

- **Small but dense.** A node that looks like README prose is wrong. No marketing, no exhaustive API lists — link to generated docs for those.
- **Budget is measured, not felt.** Run the `wc -l` command; never estimate. Over ceiling is a bug: split at a semantic boundary, hoist to the parent, or move detail to `agent_docs/`.
- **No code style rules.** Never send an LLM to do a linter's job. Nodes carry architecture and intent only.
- **Never duplicate raw code.** Describe *intent*, not implementation. Point with `file:line`; don't copy.
- **Facts at the LCA.** Duplicated facts across siblings → hoist to the parent.
- **Semantic boundaries, not directories.** A node at every directory is naive and will drift.
- **Docs are evidence, not truth.** ADRs and design docs supply the "why" and shorten the SME interview, but every claim is verified against code before it becomes an invariant. Unverified → `TODO(intent)`.
- **Capture invariants invisible in code.** "Never do X", "Y must happen before Z", "this looks dead but isn't". This is the whole point.
- **Review like code.** Propose diffs, never auto-commit.
- **Leaf-first, always.** Parents summarize compressed children, not raw code. Root last.
- **Open questions are first-class.** `> TODO(intent): <question>` beats an invented answer.
- **Line count flat or falling.** Growth needs a stated reason and user approval.

## Anti-patterns (of the skill itself)

- A 15k-token monolithic root — the naive approach both source posts call out.
- One node per directory.
- Writing for humans — tutorials, onboarding prose. Agents read this; write for tokens.
- Mandatory-section filler — padding empty headings with restated obvious facts.
- Copy-pasting function signatures instead of downlinking.
- Running sync on every save. Sync belongs at merge / post-commit.
- **Diff-scoped sync.** Only visiting nodes near changed files. Cold nodes rot.
- **Append-only sync.** "What's missing?" without "what's wrong or dead?". Sync is reconciliation, not accretion.
- **Ratcheting growth.** A few lines per sync "just in case". Ten syncs later the node is a 400-line README.
- **Style rules in nodes.** "Use single quotes", "sort imports" — linter territory.
