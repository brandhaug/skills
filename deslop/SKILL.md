---
name: deslop
description: Deslop a codebase — launch 7 parallel cleanup agents that sweep the scope for slop, from duplicated and dead code to weak types and AI-generation tells, then consolidate findings and apply approved fixes. Use when the user wants to deslop, strip AI slop, run a cleanup or quality sweep, or prep a PR, branch, or directory for review.
---

# Deslop

## Quick start

```
/deslop                  # full codebase
/deslop src/api          # directory
/deslop 1234             # PR number (gh)
/deslop feat/new-auth    # branch (diff vs default branch)
```

## Workflow

1. **Resolve scope** — turn the argument into a concrete file list:
   - Empty → full codebase
   - All digits or `#N` → `gh pr view N --json files` → changed files
   - Branch name → `git diff --name-only <default>...<branch>`
   - Path → use as-is (dir or glob)

2. **Compute excludes** — build an ignore-list and filter the file list through it:
   - Every pattern in `.gitignore` and `.git/info/exclude`
   - Build output: `dist/`, `build/`, `out/`, `.next/`, `target/`, `__pycache__/`
   - Vendored / generated: `node_modules/`, `vendor/`, `.venv/`, `*.generated.*`, `*.gen.go`, `*_pb2.py`, `__snapshots__/`, `*.snap`
   - Lockfiles

3. **Size check** — if scope > 500 files (tunable), stop and prompt the user to narrow: by directory, by churn (`git log --since='3 months' --name-only --pretty=format:`), or by largest N files. Agents blow their context on huge scopes — don't push through.

4. **Detect tooling** — read `package.json`, `tsconfig*.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`. Note installed tools (knip, ts-prune, ruff, vulture, staticcheck, clippy). For polyglot repos, bucket files by language; each language gets its own set of 7 agents.

5. **Launch all 7 agents in parallel** — a single message with 7 `Agent` calls, `subagent_type: "general-purpose"`. Each agent's prompt is its section of [AGENTS.md](AGENTS.md) with `{scope}`, `{excludes}`, `{tools}` filled in, plus the shared output format. This pass is analysis-only — no file edits, so parallel agents can't conflict. Print `[N/7 done: <category>]` as each agent returns.

6. **Consolidate findings** — merge the reports so every finding from all 7 appears exactly once: deduplicate overlap (agents 1 and 2 often flag the same shared-type issue), group by file, sort by severity. Quarantine findings below 0.7 confidence in a "needs review" section — never auto-applied. Write the full report to `deslop-report.md` at the repo root.

7. **Review with user** — summarise from the report: count per category, a few top examples each, total files touched. Point at `deslop-report.md` for the full list. Ask which categories or files to apply.

8. **Apply changes (branch + worktree fan-out)** — when the user approves:
   1. Confirm clean working tree. Create branch `deslop/<yyyy-mm-dd>` off the current branch. Refuse to edit on `main`/`master`.
   2. For each approved category, spawn an `Agent` with `isolation: "worktree"` and that category's findings. Each worktree edits independently — no conflicts.
   3. Merge each worktree back to the cleanup branch in precedence order: deletion (3, 6) → simplification (5, 7) → consolidation (1, 2) → typing (4). Run build + typecheck + tests between each merge; stop and surface on regressions.
   4. Done when the `deslop/<date>` branch holds one commit per applied category and `deslop-report.md` records every finding as applied, deferred, or quarantined.

## The 7 agents

1. **Deduplication**
2. **Type consolidation**
3. **Unused code**
4. **Weak types**
5. **Defensive programming**
6. **Legacy / deprecated code**
7. **AI slop** — comments, over-nesting, style drift

Per-agent prompts: [AGENTS.md](AGENTS.md).

## Guardrails

- **Behavior unchanged.** Cleanups are refactors, not rewrites — no semantic changes smuggled in unless fixing a clear bug.
- **PR scope stays in scope.** Don't propose changes outside the diff unless the user opts in.
- **Bias to deletion over rewriting** (agents 3, 5, 6, 7). Surface anything ambiguous.
