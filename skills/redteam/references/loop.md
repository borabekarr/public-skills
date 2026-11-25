# The redteam loop: signals, cap and kill

The signal is decided from the ledger table (`references/ledger.md`) by the rules below, never by feel. Compute it after every judge pass, once the ledger has been updated for that round. R is the round number being judged (1, 2, 3).

## Counts to take from the ledger

- open: rows of this run with status `open` (new keys, reopened keys and keys carried over).
- parked: rows with status `parked`.
- fixed twice: a row with status `open` whose `fixed x` is 2 or more (it was fixed twice and has come back).
- newly fixed in round k: the number in the rounds table for that closed round.

## Signal table

First matching row wins. Read top to bottom and stop at the first row whose condition is true.

| # | Condition | Signal | Action |
|---|---|---|---|
| 1 | open is 0 and parked is 0 | `done` | End the loop. Report clean. |
| 2 | any key is fixed twice and open again | `kill` | Stop. Run the kill procedure below. |
| 3 | R is 3 or more and the last two closed rounds both had zero newly fixed keys | `kill` | Stop. Run the kill procedure below. |
| 4 | open is 0 and parked is 1 or more | `park` | Stop. Report the parked keys and the question for each. |
| 5 | R is 3 or more | `cap` | Stop. Report the open keys. Never start round 4. |
| 6 | otherwise | `continue` | Write and run the fix plan, then start round R+1. |

The cap is a hard limit: at most 3 rounds. Rows 2 and 3 are checked before the cap, so a loop that is going nowhere is reported as killed, not capped.

A key counts as fixed only when the fix plan was run and its check passed. A key the fix plan did not touch stays open.

## Worked example: 2 rounds, 4 keys

The same run as `references/ledger.md` (`demo`, lens `system`).

Ledger after round 2:

| ns | lens | class | files | anchor | anchor text | status | fixed x | seen |
|---|---|---|---|---|---|---|---|---|
| demo | system | unwired-unit | skills/export.md | Export step | `3. Export step: write the file` | fixed | 1 | 1 |
| demo | system | stale-reference | rules/limits.md | max retries | `max retries: 3` | rejected | 0 | 1 |
| demo | system | weak-check | tests/check.sh | grep -q ready | `grep -q ready out.txt && grep -q ok out.txt` | fixed | 1 | 1,2 |
| demo | system | unresolved-path | run.md | output dir | `write to ./out/` | open | 0 | 2 |

Signal for round 1 (taken before the fix, when the ledger held keys A, B, C all open):

1. Row 1: open is 3, so no.
2. Row 2: no key has `fixed x` of 2 or more, so no.
3. Row 3: R is 1, so no.
4. Row 4: open is 3, so no.
5. Row 5: R is 1, so no.
6. Row 6: `continue`. The fix plan fixes A and C, the owner rejects B. Newly fixed in round 1: 2.

Signal for round 2 (the ledger above; the judge re-raised C and it was suppressed because its stored anchor text is unchanged, and key D is new):

1. Row 1: open is 1 (key D), so no.
2. Row 2: no open key has `fixed x` of 2 or more, so no.
3. Row 3: R is 2, so no.
4. Row 4: open is 1, so no.
5. Row 5: R is 2, so no.
6. Row 6: `continue`. The fix plan fixes D; round 3 follows.

What would come next: at round 3 with D fixed and no new key, row 1 gives `done`. With D still open, rows 1 to 4 do not match and row 5 gives `cap`. If rounds 1 and 2 had both fixed nothing, row 3 would already give `kill` at round 3.

## Kill procedure

1. Append an event line under the ledger: `event: kill, round R, reason: <which row matched and which key or rounds>`.
2. Write down the cause in one sentence: the loop bug (for example "the judge keeps re-raising an anchor whose text the fix never changes" or "the fix plan does not touch the file the finding names") and the file it lives in.
3. Write `{{WORK_DIR}}/redteam/fix-<topic>-loop.md` from `references/fix-plan-template.md`: one fix step for the cause and one regression step whose check replays the looping state. Run it only when the owner agrees; do not start another redteam.
4. Report what looped, the cause and the fix. Stop.

After a kill, never start a new redteam on the same change in the same session.

## No-recursion rule

A fix plan written from a finding, and the loop-bug fix plan, never start a redteam of their own. Their header says so (see the template). The next round is started by this loop at step 1 of the skill, not by the fix.

## When the ledger cannot be read

If the ledger file is missing, unreadable or lost its header, the signal is not computable. Report `could-not-look: ledger` and stop. Do not rebuild it from memory and do not continue the loop without it.

## Single-agent runs

The signal table is the same. The reader and judge run as separate passes of one agent (see Single-agent mode in the skill), and the same counts are taken from the same ledger.
