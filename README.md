# use-opencode

A [Claude Code](https://claude.com/claude-code) skill that splits a coding task
across two agents: **Claude plans and verifies, [opencode](https://opencode.ai)
implements.**

## Why

**To spend your Claude tokens where they actually matter.**

Most of the tokens in a coding session get burned on the boring part — typing
out the implementation. That part does not need a frontier model. Planning the
change and reviewing the diff do.

So this skill puts Claude in charge and sends the typing somewhere cheap:
opencode pointed at a budget API model, or a model you self-host and run for
free. Claude stays in the loop for the two jobs worth paying for — deciding what
to build, and checking what came back — while the bulk of the output tokens come
from the cheap model.

You keep frontier-quality judgement on both ends of the task, and your Claude
usage goes a lot further.

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

The skill uses whatever model opencode defaults to, so set that to the cheap or
self-hosted one — that is the whole point. opencode talks to hosted APIs and to
local runtimes like Ollama or anything OpenAI-compatible, so a model running on
your own machine costs you nothing per token.

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
