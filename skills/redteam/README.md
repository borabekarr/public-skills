# redteam

A skill that reviews one change and what it touches for gaps and weak spots, then runs a guarded fix loop. Scope comes from a change map, a reader gathers quotes, a separate judge decides, and a ledger table plus a hard round cap stop the loop. It is not a security scan.

## What it does

- Builds a change map: the files a change touched plus the files that read them, capped per file.
- Runs a reader pass (quotes only) and a judge pass (decides, reading the files itself) as two different seats.
- Records findings in a markdown ledger keyed by lens, gap class, sorted files and anchor, so a finding already fixed or rejected is not raised twice.
- Decides stop or continue from a first-match signal table (done, kill, park, cap, continue), by counting ledger rows, not by feel.
- Writes a fix plan per round, runs it, and re-checks in the next round, for at most 3 rounds.
- Reports what was found, fixed, rejected, parked and killed, which changed units are unwired, and what was left out.

## When to use

- After a workflow, prompt set, skill, config or code change is built or changed, before you call it finished.
- When you want the gaps in a change and its ripple found, not the whole repository audited.
- For security work, use your own security tooling instead (dependency audit, secret scan, static analysis, a security reviewer).

## Install

Copy the `redteam` folder into the skills directory your harness reads. Copy only: if a skill with the same name already exists there, keep it and use another folder name instead of replacing it. Never delete anything to make room.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The flow, the stop rules and the rules in one place |
| `references/change-map.md` | How to build the change map and the reader lists by hand |
| `references/lenses.md` | Checklists and gap classes for the code, system and product lenses |
| `references/ledger.md` | The ledger table, the key, suppression, with a worked example |
| `references/loop.md` | The signal table and kill procedure, with a worked example |
| `references/fix-plan-template.md` | The fix plan template |
| `references/rea-method.md` | The claim and check method findings follow |
| `roles/reader.md` | Prompt for the reader seat |
| `roles/judge.md` | Prompt for the judge seat |

Each file stands alone inside this folder. The only pointers outside it are the repo-root `ADAPTING.md` (the shared placeholder list, also given in SKILL.md) and `NOTICE.md` (credits); the skill runs without either.

## Adapt to your model and harness

Works with Claude, Codex, Gemini, Grok, or any other agent that can read files, search them and run a few commands. Placeholders used here, defined in `ADAPTING.md` at the repo root:

- `{{SUBAGENT}}` is how to start a fresh helper for the reader and the judge. If your harness cannot, use Single-agent mode below.
- `{{WORK_DIR}}` is where the change map, ledger and fix plans are written (default `.skill-work/redteam/`).
- `{{FAST_MODEL}}` is the seat for the reader, `{{STRONG_MODEL}}` the seat for the judge. Name your own models there; the role files carry no model names.
- `{{PROJECT_ROOT}}` is the repository being reviewed.

What to tune:

- **Model strength.** The reader only gathers, so a fast model works. Give the judge your strongest model; a weak judge raises false findings and misses real ones.
- **Whether helpers can run.** The two-seat split is the point of the skill. Use separate helpers when you can; otherwise use Single-agent mode.
- **Version control.** The change map uses `git`. For another system, replace the commands in `references/change-map.md` or hand the agent a file list.
- **Lenses.** Edit the classes in `references/lenses.md` to match the kinds of gaps your project actually has; keep the class names stable inside one run, because they are part of the ledger key.
- **The round cap.** Three rounds is the default hard cap. Lower it if rounds are expensive; raise it only together with the kill rule, never alone.
- **Harness specifics.** Install paths and helper mechanics differ per harness. Check your harness's own documentation instead of trusting this README.

## Single-agent mode

When the harness cannot spawn helpers, run the reader and the judge as two separate passes. Pass one: do the reader job from `roles/reader.md` and write its quotes to `{{WORK_DIR}}/redteam/quotes.md`. Pass two: start fresh (a new session if you can), open `roles/judge.md`, and re-read every cited file. Never judge text you gathered in the same pass without re-reading the files. Everything else, including the ledger and the signal table, is unchanged.

## Credits

Credits: the claim and check method in `references/rea-method.md` is based on morluto/rea (MIT); see NOTICE.md.
