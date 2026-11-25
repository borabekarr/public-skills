---
name: redteam
description: "Use after a workflow, system or code change is built or changed, to hunt gaps, weak spots and improvements in that change and what depends on it. Works from a change map, a reader pass and a separate judge pass, with a ledger that stops the fix loop. Not a security scan."
---

# /redteam [--lens code|system|product] [--ns <name>]

A gap and weak-spot review of ONE change and what it touches. Not a security scan: for security use your own security tooling (dependency audit, secret scan, static analysis, a security-focused reviewer). The word "red team" for attacking an AI model also belongs there, not here.

Start: a finished change (a diff, a step, an upgrade). Goal: every gap found, fixed, and re-checked next round, or killed or parked with a cause, then one report. `--ns <name>` names the run (a short topic) and is used in file names and ledger rows; default is the branch name or `run`.

Output: one report at the end (step 8 of the Flow) listing found, fixed, rejected, parked, killed and `could-not-look` items, every unwired changed unit, and a "Left out" list.

Detail files, all inside this folder:
- `references/change-map.md`: how to build the change map and the consumer lists.
- `references/lenses.md`: checklist and gap classes per lens.
- `references/ledger.md`: the ledger table, the key, suppression, with a worked example.
- `references/loop.md`: the signal table, the kill procedure, with a worked example.
- `references/fix-plan-template.md`: the fix plan template.
- `references/rea-method.md`: the claim and check method every finding follows.
- `roles/reader.md` and `roles/judge.md`: the two prompts for the two seats.

Placeholders (replace once for your harness): `{{PROJECT_ROOT}}` the repo being reviewed, `{{WORK_DIR}}` where notes live (default `.skill-work/redteam/`), `{{SUBAGENT}}` how to start a fresh helper, `{{FAST_MODEL}}` for the reader, `{{STRONG_MODEL}}` for the judge. No helper available: see Single-agent mode below.

Method: judging follows `references/rea-method.md`. Every finding states what was looked at and ends in a lookup state: `found`, `nothing` or `could-not-look`. Unconfirmed items are marked, never guessed.

## Flow

**1. Change map.** Build it per `references/change-map.md`: `git diff --name-only <base>...HEAD` plus untracked files, each with a kind (code, config, prompt/doc, test, schema) and its readers found by grepping the repo for its path or name, capped at 25 per file with a `CAP <total>` note. Depth 1 by default; depth 2 for schema, shared config or prompt files. Save it to `{{WORK_DIR}}/redteam/change-map.md`. No map (no repo, no base, no file list from the owner) means report `could-not-look: no change map` and stop. Never guess the changed files from memory.

**2. Lens.** From `--lens`, else decide from the map: `code` (source, tests, schemas), `system` (agent prompts, skills, workflows, configs) or `product` (user-facing behavior). Load that lens's checklist from `references/lenses.md`.

**3. Reader pass.** Start `{{SUBAGENT}}` with `roles/reader.md`, in waves of at most 5 readers (split the map files across them). Each brief lists the lens, the exact files and their consumer lists. The reader returns over-inclusive verbatim quotes with `file:line` and a lookup state per thing asked. The reader gathers and never judges.

**4. Judge pass.** A different seat from the reader: a separate helper, or a fresh pass that did not do the reading. Brief it with `roles/judge.md`, the lens checklist, the map, the consumer lists and the reader quotes. It reads the cited files itself (quotes are leads, not proof), applies `references/rea-method.md`, and returns findings: lens, gap class, files, anchor, anchor text, severity, lookup state.

**5. Ledger.** Record each finding in `{{WORK_DIR}}/redteam/ledger.md` per `references/ledger.md`. The key is four plain fields: lens + gap class + sorted files + anchor. A key already fixed or rejected is suppressed (not re-raised) unless its anchored text changed since the row was written.

**6. Signal.** Decide from the ledger with the first-match table in `references/loop.md`, never by feel. Read the signal here, in this order:
- `done`: zero open keys and zero parked keys. The loop ends clean.
- `kill`: a key reopened after it was fixed twice, OR (from round 3) the last 2 closed rounds each had zero newly fixed keys. Stop at once; follow the kill procedure in `references/loop.md`.
- `park`: no open key is left but at least one key is `parked` (needs a choice only the owner can make). Stop and report them with their questions.
- `cap`: round 3 reached (hard round cap: at most 3 rounds, never a round 4). Stop and report the open keys.
- `continue`: otherwise, go to step 7.

**7. Fix plan.** Write `{{WORK_DIR}}/redteam/fix-<topic>-r<n>.md` from `references/fix-plan-template.md`, one step per open finding, then run it and mark each fixed key `fixed` in the ledger (add 1 to its `fixed x` count). Start round n+1 at step 1: the map now also holds the files the fix changed. A fix plan never starts a new redteam.

**8. Report.** Close with: found, fixed, rejected, parked, killed and `could-not-look` items, every changed unit that has no named reader (reported as `unwired`), and a "Left out" list: what was not read, skipped, parked or capped, and why. A report with no left-out line is incomplete; write "Left out: nothing" only when true.

## Stop rules (the whole loop guard, no script needed)

Check these after every judge pass. They live in the ledger and `references/loop.md`; this is the same list in short form:
- Round cap: 3 rounds at most. At round 3, stop and report whatever is still open.
- Kill: 2 rounds in a row with zero newly fixed keys, or any key that returns to open after being fixed twice (`fixed x` is 2 and its status is open again). After a kill, never start a new redteam on the same change.
- Done: zero open and zero parked keys after a judge pass.
- Park: no open key left, at least one key parked for the owner's decision.
- A key marked `fixed` or `rejected` stays suppressed while its anchor text is byte-identical to the stored copy.

Count with the ledger table, by hand. If the ledger is missing or cannot be read, that is `could-not-look`: stop and report instead of continuing without it.

## Single-agent mode

When the harness cannot start helpers, run the reader and the judge as two separate passes of the same agent: first the reader pass, writing its quotes to `{{WORK_DIR}}/redteam/quotes.md`; then a fresh pass (new session, or at least a reset of what you assume) that opens `roles/judge.md` and re-reads every cited file. Never judge text you gathered in the same pass without re-reading the files.

## Rules

- Scope is the change map plus its capped readers. Never the whole repo.
- A finding already fixed or rejected is not re-raised unless its anchored text changed.
- Reader and judge are different seats: the reader gathers quotes, the judge decides.
- Only the final round commits. Fix rounds edit the working tree and leave committing to the end (or to the owner).
- Every changed unit needs a named reader (a consumer, a caller, a doc that points at it) or it is reported as unwired.
- A report must list what was left out.

## Gotchas

- A changed file with no map row is a map defect: report it and fix the map; do not widen the scan to the whole repo.
- The name sounds like security: pentests, injection scans and dependency audits belong to security tooling.
- A fix round that changes a shared config or prompt creates new ripple: the next map carries it (depth 2 for those kinds).
- Quotes that disagree with the file win for the file: the judge trusts what it reads, not what it was handed.
