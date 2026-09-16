---
name: use-opencode
description: Plan a coding task yourself, then hand the implementation to the opencode CLI (`opencode run`) and verify the result. Use when the user invokes /use-opencode, says "use opencode", "hand this to opencode", "let opencode build it", or otherwise asks for opencode to do the implementation.
---

# use-opencode

You plan. opencode implements. You verify. Never write the implementation
yourself in step B — that is opencode's job, and skipping the handoff defeats
the whole point of this skill.

The user's request is everything after `/use-opencode`. If it is empty, ask
what they want built and stop.

---

## A. Plan (you, in plan mode)

1. Call `EnterPlanMode` (load its schema with `ToolSearch` first if needed).
2. Read the code the change touches before planning it. Trace the real flow
   end to end — every file the change hits.
3. Check the repo's own agent docs (`CLAUDE.md`, `AGENTS.md`, any notes
   directory it keeps). They often kill a wrong plan before it costs anything.
4. Write a plan concrete enough that another agent can execute it without
   guessing: exact file paths, exact function/class names, the behavior
   change, and the tests to add. Vague plans produce vague opencode output.
5. Use `AskUserQuestion` for any decision that changes what gets built.
6. Call `ExitPlanMode` and get the user's approval. **Do not run opencode
   before the plan is approved.**

## B. Hand off to opencode

Run the default model — do **not** pass `-m`/`--model`; the user wants
opencode's configured default.

```bash
opencode run --auto --dir "<repo root>" "<the full prompt>"
```

Build `<the full prompt>` from the approved plan. It must be self-contained,
because opencode does not share your conversation:

- The goal, in one or two sentences.
- Absolute or repo-relative file paths to create or edit.
- The exact change per file.
- The repo's own rules that apply (from `CLAUDE.md` / `AGENTS.md`) — e.g.
  "write pytest tests under `tests/` mirroring the source layout, named
  `test_*.py`, in the same change".
- How to check the work (the test command).

Notes on running it:

- `--auto` auto-approves permissions so opencode can actually write files in a
  non-interactive run. Without it the run stalls.
- Pass the prompt as ONE quoted argument. On PowerShell use a single-quoted
  here-string (`@'` … `'@`, closing delimiter at column 0) for a multi-line
  prompt; on Bash use a heredoc into a variable.
- It is long-running. Use `run_in_background: true` and let the completion
  notification wake you, rather than a short foreground timeout.
- If the prompt is big, write it to the scratchpad directory first and pipe or
  pass it, instead of wrestling with shell quoting.
- Never add `-m`, `--agent`, or `--variant` unless the user explicitly asks.

Report to the user which prompt you sent, in one short block.

## C. Verify

opencode's own claim that it worked is not evidence. Check it yourself.

1. `git status` and `git diff` — see exactly what changed. Confirm it
   matches the approved plan. Flag anything opencode touched that the plan did
   not mention.
2. Read the changed files. Look for: wrong file edited, stubbed/`TODO` bodies,
   dropped error handling, invented APIs that do not exist in this repo, tests
   that assert nothing.
3. Run the tests. Python: `pytest -q`. Node/TS: `npm test`.
   Use whatever the repo actually uses.
4. If opencode skipped the tests the repo's rules require, that is a failure —
   say so.

### If verification fails

Fix it yourself. A second `opencode run` on a broken diff usually compounds
the damage. Keep the fix to the smallest change that makes it correct, and
tell the user exactly what opencode got wrong and what you changed.

### Report

Three things, then stop:

1. What opencode built (files changed, one line each).
2. Whether the tests passed — with the real result, including failures.
3. What the user does next.

Do not commit unless the user asks.
