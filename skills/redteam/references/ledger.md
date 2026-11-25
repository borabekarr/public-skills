# The redteam ledger

The ledger is what stops the fix loop from repeating itself. It is a markdown table kept by hand in `{{WORK_DIR}}/redteam/ledger.md`. Append a row for every finding, update rows in place, and never delete a row.

## Columns

| Column | Meaning |
|---|---|
| ns | The run name from `--ns` (rows of other runs in the same file are ignored). |
| lens | `code`, `system` or `product`. |
| class | The gap class from `references/lenses.md`. |
| files | The files the finding is about, sorted alphabetically, joined with a comma. |
| anchor | A symbol, heading or config key inside the first listed file, copied exactly. |
| anchor text | The anchored lines, verbatim, at most 5 lines per listed file, each block prefixed with its file name (`a.md: ...`). Write `\|` for a pipe character. |
| status | `open`, `fixed`, `rejected` or `parked`. |
| fixed x | How many times the key has been marked fixed. Starts at 0. |
| seen | The rounds in which the judge raised the key, for example `1,2`. |

## The key

The key is four plain fields: **lens + class + sorted files + anchor**. Two findings are the same finding when all four fields match exactly. Compare by reading the fields; do not rephrase them to look alike, and do not treat two keys as equal when only the class or only the anchor matches.

## Rules for each judge pass

For every finding the judge returns:

1. Look for a row with the same ns and key.
2. No row: append one with status `open`, `fixed x` 0, seen = this round, anchor text = the verbatim anchored lines from the file now.
3. A row exists and its status is `open` or `parked`: add this round to seen. Nothing else changes.
4. A row exists with status `fixed` or `rejected`:
   - Read the anchored lines in every listed file now. If every file's lines equal its stored block (ignoring only trailing whitespace), the finding is **suppressed**; if any listed file differs from its stored block, go to the reopen case below. A suppressed finding only gets this round added to seen; nothing else changes.
   - Otherwise the region changed since the decision: **reopen**. Set status to `open`, replace the anchor text with the current lines, add the round to seen. Reopening a key whose `fixed x` is 2 or more is a kill (see `references/loop.md`).
5. When a key is marked `fixed`, add 1 to `fixed x` and replace anchor text with the lines as they stand after the fix. When it is marked `rejected` (the owner or the judge's counter-case shows it is not a gap), store the lines as they stand now. This is what makes a later re-raise of unchanged text suppressible.
6. A key that needs a decision only the owner can make gets status `parked`, with the question written in a note under the table.

After each round add one line to the rounds table below the ledger: round, how many findings were new, how many suppressed, how many keys were newly fixed by that round's fix, and the signal.

## Worked example: 2 rounds, 4 keys

Run `demo`, lens `system`. Round 1 judge pass raises three findings (A, B, C). The fix plan fixes A and C; the owner says B is intended, so B is rejected. Round 2 judge pass raises C again (the judge does not remember round 1) and raises a new key D.

Ledger after round 2:

| ns | lens | class | files | anchor | anchor text | status | fixed x | seen |
|---|---|---|---|---|---|---|---|---|
| demo | system | unwired-unit | skills/export.md | Export step | `3. Export step: write the file` | fixed | 1 | 1 |
| demo | system | stale-reference | rules/limits.md | max retries | `max retries: 3` | rejected | 0 | 1 |
| demo | system | weak-check | tests/check.sh | grep -q ready | `grep -q ready out.txt && grep -q ok out.txt` | fixed | 1 | 1,2 |
| demo | system | unresolved-path | run.md | output dir | `write to ./out/` | open | 0 | 2 |

How round 2 treats each finding:

- Row 1 (fixed): the judge did not raise it again; nothing to do.
- Row 2 (rejected): not raised again; stays rejected.
- Row 3 (re-raised and suppressed): raised in round 2, same four fields. Its stored anchor text (stored after the fix) equals the lines in the file now, so it is suppressed: seen becomes `1,2` and nothing else changes.
- Row 4 (open): new in round 2, no matching row, appended as `open`.

Rounds table:

| round | new | suppressed | newly fixed | signal |
|---|---|---|---|---|
| 1 | 3 | 0 | 2 | continue (3 open keys, round 1 is below the cap) |
| 2 | 1 | 1 | not yet (fix pending) | continue (1 open key, round 2 is below the cap, no key fixed twice) |

Round 1 closed with keys A and C fixed (newly fixed 2). The signal for each round comes from the first matching row of the table in `references/loop.md`: round 1 and round 2 both reach the last row (`continue`). If round 3 raises nothing new and key D gets fixed, the signal is `done`; if key D is still open when round 3 is judged, the signal is `cap`.
