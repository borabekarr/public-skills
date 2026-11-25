# diagnose

A skill for hard or intermittent bugs. It makes the agent build a fast pass/fail feedback loop first, then reproduce, rank falsifiable hypotheses, instrument one variable at a time, fix, and lock the fix with a regression test.

## What it does

- Forces a reproducible loop before any theorising.
- Requires 3-5 ranked hypotheses, each with a prediction that a probe can disprove.
- Tags debug output so cleanup is one search.
- Ends with a cleanup checklist and a "what would have prevented this" note.

## When to use

- Flaky tests, "works on my machine", timing and ordering bugs.
- Performance regressions.
- Bugs where the first fix did not hold.

Skip it for a known error with a stack trace and no reproduction problem; fix those directly.

## Install

Copy the `diagnose` folder into the skills directory your harness reads. Copy only: if a skill with the same name already exists there, keep it and pick another folder name rather than replacing it. Never delete anything to make room.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The skill: output shape, six phases, cleanup checklist |
| `references/root-cause-tracing.md` | Trace a bad value backward to its original trigger |
| `references/condition-based-waiting.md` | Replace guessed sleeps with condition polling |
| `references/defense-in-depth.md` | Validate at every layer after the fix |
| `scripts/hitl-loop.template.sh` | Scaffold for a loop a human must drive; copy and edit the steps |

Each file stands alone inside this folder; nothing links outside it.

## Adapt to your model and harness

Works with Claude, Codex, Gemini, Grok, or any other agent that can read a folder of markdown and run shell commands. The placeholders used across these skills are listed in `ADAPTING.md` at the repo root; this skill uses none in its text, so it runs as copied. What to tune:

- **Model strength.** The six phases are written for a capable model. If yours drifts, you can try running one phase per turn and pasting the output block into the prompt.
- **Whether helpers can run.** The skill runs in one session. If your harness can start helper agents, you may add an optional step that asks one to challenge the ranked hypotheses from Phase 3.
- **Where notes are written.** Phase 6 asks for a commit or PR message and a prevention note. Point that at wherever your harness keeps project notes.
- **Tooling.** The loop options name common tools (a headless browser, `git bisect`). Delete the ones your environment lacks.
- **Harness specifics.** Install paths and subagent mechanics differ per harness. Check your harness's own documentation instead of trusting this README.

## Credits

Credits: based on the diagnose skill by Matt Pocock (MIT) and debugging techniques from obra/superpowers (MIT); see NOTICE.md.
