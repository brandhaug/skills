# skills

A collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills, installable via [skills.sh](https://skills.sh).

## Install

Recommended — via the [skills](https://github.com/vercel-labs/skills) CLI:

```bash
npx skills add brandhaug/skills
```

List available skills in this repo first:

```bash
npx skills add brandhaug/skills --list
```

Install a specific skill globally for Claude Code:

```bash
npx skills add brandhaug/skills --skill deslop -g -a claude-code
```

Manual install (symlink into `~/.claude/skills/`):

```bash
git clone git@github.com:brandhaug/skills.git
ln -s "$PWD/skills/deslop" ~/.claude/skills/deslop
```

## Skills

### [deslop](deslop/)

Comprehensive codebase quality sweep. Launches 8 specialized cleanup agents in parallel — deduplication, type consolidation, unused code removal, circular dependencies, weak types, defensive programming, legacy code, and AI slop / unhelpful comments / over-nesting.

Accepts an optional scope argument (PR number, branch name, or directory path). Runs on the full codebase if omitted.

```
/deslop                  # full codebase
/deslop src/api          # directory
/deslop 1234             # PR number (resolved via gh)
/deslop feat/new-auth    # branch (diff vs default)
```

Workflow at a glance:

1. Resolves scope to a file list, computes excludes, size-checks
2. Launches 8 analysis agents in parallel — no edits, no conflicts
3. Consolidates findings into `deslop-report.md`
4. On approval, fans out apply-agents in git worktrees, merges in precedence order, runs build/typecheck/tests between each

See [deslop/SKILL.md](deslop/SKILL.md) for the full workflow and [deslop/AGENTS.md](deslop/AGENTS.md) for each agent's prompt.

## Contributing

New skills welcome. Each skill lives in its own directory at the repo root:

```
skills/
└── <skill-name>/
    ├── SKILL.md       # required — name, description, workflow
    └── ...            # optional reference docs, scripts, examples
```

`SKILL.md` frontmatter must include `name` and `description`. See Anthropic's [skill-creator](https://github.com/anthropics/skills) for the full format. Open a PR adding your skill directory and a one-line entry under [Skills](#skills).
