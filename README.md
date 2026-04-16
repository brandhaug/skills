# skills

A collection of [Claude Code](https://docs.claude.com/en/docs/claude-code) skills.

## Skills

### [deslop](deslop/)

Comprehensive codebase quality sweep. Launches 8 specialized cleanup agents in parallel — deduplication, type consolidation, unused code removal, circular dependencies, weak types, defensive programming, legacy code, AI slop / unhelpful comments / over-nesting.

Accepts an optional scope argument (PR number, branch name, or directory path); runs on the full codebase if omitted.

```
/deslop                  # full codebase
/deslop src/api          # directory
/deslop 1234             # PR number (gh)
/deslop feat/new-auth    # branch (diff vs default branch)
```

See [deslop/SKILL.md](deslop/SKILL.md) for the full workflow and [deslop/AGENTS.md](deslop/AGENTS.md) for per-agent prompts.

## Install

Symlink individual skills into `~/.claude/skills/`:

```bash
git clone git@github.com:brandhaug/skills.git
ln -s "$PWD/skills/deslop" ~/.claude/skills/deslop
```

Or copy:

```bash
cp -r skills/deslop ~/.claude/skills/
```
