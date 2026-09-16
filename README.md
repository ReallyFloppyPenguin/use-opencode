# use-opencode

A [Claude Code](https://claude.com/claude-code) skill that splits a coding task
across two agents: **Claude plans and verifies, [opencode](https://opencode.ai)
implements.**

## Why

Planning and reviewing are what a large reasoning model is good at. Typing out
the implementation is what a fast coding agent is good at. This skill keeps each
one doing its own job, and — importantly — makes Claude actually *check* the
result instead of trusting the handoff.

## What it does

1. **Plan** — Claude enters plan mode, reads the code the change touches, checks
   the repo's own agent docs, and writes a plan concrete enough for another
   agent to execute. You approve it before anything runs.
2. **Hand off** — the approved plan becomes a self-contained prompt for
   `opencode run --auto`, using opencode's default model.
3. **Verify** — Claude reads the diff, checks it against the approved plan, and
   runs the test suite. If opencode got it wrong, Claude fixes it and tells you
   what went wrong.

## Requirements

- [Claude Code](https://claude.com/claude-code)
- [opencode](https://opencode.ai) on your `PATH`, already configured with a model

## Install

Clone into your Claude Code skills directory.

**User-global** (available in every project):

```bash
git clone https://github.com/ReallyFloppyPenguin/use-opencode \
  ~/.claude/skills/use-opencode
```

**Single project:**

```bash
git clone https://github.com/ReallyFloppyPenguin/use-opencode \
  .claude/skills/use-opencode
```

Restart Claude Code.

## Usage

```
/use-opencode add a --dry-run flag to the import script
```

Claude plans, you approve, opencode builds, Claude verifies.

## A note on `--auto`

The skill runs `opencode run --auto`, which auto-approves opencode's file
permissions. Without it, a non-interactive run stalls waiting for input. It does
mean opencode edits files without asking — so use this on a repo with a clean
git working tree, where you can read the diff afterward.

## License

MIT
