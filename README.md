# regression-testing

An agent-agnostic [Agent Skill](https://agentskills.io) for committed end-to-end regression testing.

Keep a re-runnable e2e suite that gates merges, organized around an explicit **invariant catalog**,
so changing a shared seam can't silently break existing behavior. Self-contained and pluggable: it
depends on no other skill and exposes a single pass/fail command that any merge gate or delivery
pipeline can call.

The skill definition lives in [`SKILL.md`](./SKILL.md).

## Install

**As an Agent Skill (any tool that follows the open standard):** clone or copy this repo to a skill
directory your agent reads — e.g. `~/.agents/skills/regression-testing/` (user-level) or
`<project>/.agents/skills/regression-testing/` (project-level). For Claude Code specifically, place
or symlink it under `.claude/skills/regression-testing/`.

**As a git submodule (recommended for projects):**

```bash
git submodule add git@github.com:ja-zoe/agent-skill-regression-testing.git \
  .agents/skills/regression-testing
```

Then bridge it into any agent that needs its own directory (Claude Code reads `.claude/skills/`):

```bash
ln -s ../../.agents/skills/regression-testing .claude/skills/regression-testing
```
