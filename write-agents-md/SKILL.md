---
name: write-agents-md
description: Write and maintain AGENTS.md (or CLAUDE.md) files as an Intent Layer — a hierarchy of small, dense context files at semantic boundaries that auto-load as architectural context for agents. Use when the user wants to write, create, generate, scaffold, seed, sync, trim, or prune AGENTS.md / CLAUDE.md files, agent context files, intent nodes, intent layer, or architectural memory for agents; or mentions "AGENTS.md hierarchy", "intent layer", "intent nodes", "agent context", "T-shaped context", "dark room problem". Two workflows — `build` (initial capture, leaf-first with SME interview) and `sync` (reconcile affected nodes after code changes — additions, modifications, AND removals). Both end by cutting every touched node to a hard word budget; everything must earn its place.
---

# Write AGENTS.md

Write and maintain a hierarchy of **Intent Nodes** (`AGENTS.md` / `CLAUDE.md` files) at semantic boundaries. Agents inherit the full ancestor chain automatically, so every word in a node is paid for on every request that touches its subtree. A node is a tax on context. Every sentence has to earn its place.

Reference: https://intent-systems.com/blog/intent-layer

## Quick start

```
/write-agents-md build          # initial capture across the repo
/write-agents-md build src/api  # scoped to a subtree
/write-agents-md sync           # reconcile nodes affected by recent changes (add / modify / remove)
/write-agents-md sync HEAD~5    # reconcile nodes affected since a ref
```

## The one test

Before a sentence stays in a node, it passes this test: **would an agent editing a file in this subtree make a wrong change without it, and could it not find the fact by grepping for ten seconds?** If the answer to either half is no, delete the sentence. The code is the copy that cannot drift; the node holds only what the code does not say.

## Word budgets

Hard ceilings, checked with `wc -w` after the repo formatter runs. Every workflow ends with the cut-to-budget phase below; a node over budget is a failed step, not a follow-up.

| Node kind                                       | Budget  |
| ----------------------------------------------- | ------- |
| Root                                            | 500     |
| App / worker / deployable                       | 650     |
| Package or bounded-context node that has leaves | 750     |
| Package without leaves                          | 300–600 |
| Leaf (one capability / module / feature)        | 300     |

Budgets are per file, not per section. A node that needs more than its budget is describing two areas; split it, or move the excess into the code as a doc comment.

## Delete on sight

These never earn their place. Remove them while drafting and again in the cut-to-budget phase:

1. **Rationale or history when an ADR exists.** Replace with `(ADR NNNN)`.
2. **History about removed code.** "The old X was removed because…" describes nothing an agent can touch.
3. **Anything greppable in ten seconds.** Export lists, method lists, table and column inventories, file-tree listings, test-file rosters, "already split" lists, constants and their values, plugin option values one line from the source.
4. **Restated code or doc comments.** Point at the file instead.
5. **Facts stated in an ancestor node** or in the root. Replace with a pointer.
6. **README prose.** Tutorials, receiver-facing recipes, numbered walkthroughs of UI behavior, marketing.
7. **Numbers that go stale.** Word counts, byte sizes, before/after figures, line counts.
8. **Headings with one bullet.** Merge into a neighbour; drop any schema section that would be empty.
9. **Reassurance.** "This is deliberate", "this is fine", unless naming the landmine it defuses.
10. **Copied code** beyond a single identifier.

## Keep

- Cross-file invariants: "X must happen before Y", "both adapters must…", "this constant is declared twice and a test pins them equal".
- Never/always rules whose violation compiles and passes tests.
- Landmines: things that look dead but are not, load-bearing ordering, silent fallbacks, fail-closed defaults.
- One pointer per external contract (ADR, sibling node, file) instead of prose about it.
- Open follow-ups, as a single `> TODO(intent): …` line each.

## Pre-flight

1. **Detect filename convention.** Existing `CLAUDE.md` files → `CLAUDE.md`; existing `AGENTS.md` → `AGENTS.md`; otherwise `AGENTS.md`.
2. **Detect tooling.** `package.json`, `tsconfig*.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, monorepo manifests. Workspace package roots are strong boundary candidates. Note the repo's formatter so nodes go through it.
3. **Confirm scope.** If scope would produce more than 30 nodes, surface the plan and ask before proceeding.
4. **Never install anything.** No `bun install`, `npm install`, `pnpm install`. A subagent that runs an installer in a repo pinned to another package manager rewrites `package.json` and leaves a lockfile; after delegated work, check `git status` for exactly that and revert it.

## Build workflow

1. **Chunk at semantic boundaries**, not every directory. Chunk when responsibility, patterns, or vocabulary shift. Target 20k–64k tokens of source per chunk.

2. **Leaf-first capture.** For each leaf chunk: read entry points, exports, tests; draft a node with the schema below; apply the delete-on-sight list before saving; check the budget. Track open questions in `.intent-layer-questions.md` at the repo root.

3. **SME interview.** Batch open questions and ask the user. Target invariants, hidden contracts, "never do X" rules, historical landmines. Do not invent these.

4. **Hierarchical summarize, bottom-up.** At each parent with two or more child nodes, write a parent that summarizes the **children's nodes**, not the raw code. The parent holds the map (a link per child), the facts true across several children, and nothing a single child owns.

5. **Deduplicate at the least common ancestor.** A fact true for several children lives in the shallowest node covering all of them and nowhere else.

6. **Downlinks, not inlines.** Parents link to children and to external docs.

7. **Cut to budget** (phase below), then print a tree of nodes with word counts and ask the user to spot-check the root and the two largest before committing.

## Sync workflow

Sync is a three-way reconciliation: **current code ↔ existing node ↔ ideal node**. Always propose all three kinds of edits: additions, modifications, and removals. Treat the existing node as a draft to revise, not a floor to build on. Stale context misleads agents; missing context merely slows them.

1. **Diff scope.** `git diff --name-only <ref>...HEAD` (default ref: merge-base with the default branch). If the diff is empty because the branch is level with the default branch, fall back to the last five commits and say so.
2. **Map files to nodes.** Each changed file belongs to its nearest ancestor node.
3. **Audit existing content first.** Walk each affected node section by section and flag: stale references (verify every symbol and path with grep or ls, never from memory), drifted claims, dead anti-patterns, superseded guidance, hoist/sink violations, and everything on the delete-on-sight list.
4. **Leaf-first re-draft.** Re-read the chunk, produce the revised node with additions, edits, and deletions. If content changes materially, re-audit the parent against the updated children.
5. **Cut to budget** (phase below). A sync that only adds is a failed sync.
6. **Propose diffs.** Node-by-node with `+ added`, `~ modified`, `- removed`, one-line reason per removal ("symbol `foo` deleted in abc123", "rule no longer enforced, see `bar.ts:42`"). Do not auto-commit.

When uncertain about a removal, leave `> TODO(intent): verify - <reason>` rather than keeping stale content silently or deleting load-bearing context.

## Cut-to-budget phase

The last phase of both workflows, never skipped and never deferred to "a later cleanup".

1. **Measure.** `wc -w` every touched node, sorted. Start with the largest.
2. **Calibrate on one.** Rewrite the single largest node yourself to budget. It becomes the exemplar every other pass reads first.
3. **Fan out with a shared brief.** Write the delete-on-sight list, the keep list, the budgets, and the exemplar path into one brief file; give each worker the brief plus its files and their budgets. Workers verify every kept symbol with grep, run the formatter, and iterate until under budget.
4. **Leaves before parents.** A parent is re-audited against its cut children so it does not keep facts the leaves now own, and leaves drop anything the parent states.
5. **Check the cuts.** For each worker's report, look for a deleted cross-file invariant and restore it. Expect about one per worker; a budget met by dropping a real landmine is a regression.
6. **Verify.** All relative links resolve, no `TODO(intent)` was invented, `git status` shows only node files (plus any code comments you deliberately fixed).

## Intent Node schema

Six sections. Drop any section that would be empty rather than padding it.

```md
# <Area Name>

## Purpose & Scope
What this area owns, in two or three sentences. What it explicitly does not own.

## Entry Points & Contracts
Where behavior enters and the invariants it enforces. Pointers, not inventories.
("All writes go through `repo.save()`; direct DB writes bypass the audit log.")

## Usage Patterns
The two or three most common tasks here, as numbered steps. No code beyond an identifier.

## Anti-patterns
Negative rules whose violation compiles. "Never call X from controllers."

## Dependencies & Edges
Downlinks to child and sibling nodes, ADR numbers, one line each.

## Patterns & Pitfalls
Landmines and non-obvious constraints. One fact per bullet.
```

## Guardrails

- **Small and dense.** A node that reads like a README is wrong.
- **Never duplicate code.** Nodes describe intent; the code describes implementation.
- **Facts at the LCA.** Duplicated across siblings → hoist to the parent; true for one child → sink to it.
- **Semantic boundaries, not directories.** Most directories are not boundaries.
- **Capture what code cannot say.** "Never do X", "Y before Z", "this looks dead but isn't".
- **Verify before you write.** Every path, symbol, and ADR number in a node is checked against the tree. Wrong ADR numbers and renamed symbols are the most common defects found in practice.
- **Review like code.** Propose diffs; never auto-commit.
- **Leaf-first, always.** Parents written first summarize raw code and break the compression.
- **Open questions are first-class.** `> TODO(intent): <question>` beats an invented answer.
- **Prune as hard as you write.** Budgets are ceilings, not targets.

## Anti-patterns of the skill itself

- A 15k-token monolithic root.
- One node per directory.
- Nodes written for humans: onboarding prose, tutorials.
- Function signatures, export tables, column lists, test rosters. Use downlinks.
- Running sync on every save. Sync belongs at merge or post-commit.
- **Append-only sync.** Leaving stale references and dead anti-patterns in place while adding new sections.
- **Budget by amputation.** Meeting a budget by deleting the one cross-file invariant instead of the five greppable inventories around it.
- **Delegating without a calibration example.** Workers given a rubric alone converge on different densities; workers given a rewritten exemplar converge on the right one.
