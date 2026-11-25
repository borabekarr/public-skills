# verify-before-claim

A skill that stops an agent from shipping a confident but wrong "remove this", "merge these" or "this is unused" claim. Every claim must carry an 11-row evidence card, and an independent reviewer tries to falsify it.

## What it does

- Requires an evidence card per named component: description, triggers, source body, reverse references, change history, usage, dependencies, siblings, future relevance, a counter-example hunt, and owner context.
- Treats a blank, hand-waved or summary-quoted row as a failure.
- Sends the cards to a fresh reviewer whose job is to disprove them.
- Ships a pattern library of common failure shapes and a short skip-review procedure.

## When to use

- Before proposing to cut, remove, merge, replace or consolidate anything.
- Before calling something duplicate, unused, dead or redundant.
- Before reporting counts about a project.

## Install

Copy the `verify-before-claim` folder into the skills directory your harness reads. Copy only: if a skill with the same name already exists there, keep it and use another folder name instead of replacing it. Never delete anything to make room.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | Evidence card, 11-item checklist, procedure, reviewer instruction |
| `references/hallucination-patterns.md` | Failure shapes, each with a claim, missing evidence and a check |
| `references/skip-review.md` | A lighter second-look procedure for skipped items in a batch |

Each file stands alone inside this folder; nothing links outside it.

## Adapt to your model and harness

Works with Claude, Codex, Gemini, Grok, or any other agent that can read files and run commands. Placeholders used here, defined in `ADAPTING.md` at the repo root:

- `{{SUBAGENT}}` is how to start a fresh helper for the reviewer. If your harness cannot, run the review as a separate fresh pass that starts without your drafting context.
- `{{WORK_DIR}}` is where your harness keeps project notes and logs, used by checklist row 6.

What to tune:

- **Model strength.** A strong model can fill all eleven rows in one pass. A smaller model does better one card at a time, with the reviewer run separately per card.
- **Whether helpers can run.** The independent review is the point of the skill; the same agent re-reading its own cards is a weak substitute, so use a separate session or model when you can.
- **Where notes are written.** Row 6 searches notes and logs; point it at the real location.
- **Phases in row 9.** Replace the listed phases with your project's own.
- **Recorded decisions.** The "components already decided" section is empty by design; fill it with your own locked decisions.
- **Harness specifics.** Install paths and helper mechanics differ per harness. Check your harness's own documentation instead of trusting this README.

## Credits

Credits: based on the claim, check and evidence method of morluto/rea (MIT); see NOTICE.md.
