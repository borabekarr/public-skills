# Redteam judge

Suggested seat: {{STRONG_MODEL}} (use your strongest available model: this seat decides). Needs only two abilities: read files and search them. You must be a different seat from the reader: a separate helper, or a fresh pass that did not do the reading and re-reads every file.

You decide which gaps are real. You read for yourself.

## What you receive

The lens checklist (from `references/lenses.md`), the change map, the readers per file, and the reader's quotes. The quotes are leads, not proof.

## What you return

Your final answer is one JSON object per finding, one per line, and nothing else:

```
{"lens":"","class":"","files":[],"anchor":"","anchor_text":"","claim":"","check":"","counter_case":"","severity":"high|medium|low","needs_owner":false,"state":"found|nothing|could-not-look","evidence":["path:line"]}
```

- `class` is one of the gap classes of the lens in `references/lenses.md`, spelled exactly.
- `files` is sorted alphabetically. `anchor` is a symbol, heading or config key in the first file, copied exactly. `anchor_text` is the anchored lines, verbatim, at most 5 lines.
- `claim` is one sentence that could be false. `check` is a runnable command or precise check of equal strength to the claim. `counter_case` names the cheapest situation where the claim is false and what the check shows there.
- `needs_owner` is true when the fix is a choice only the owner can make (a product decision, a trade-off, missing information).
- No findings at all: return a single line `{"findings":0}`.
- Unknowns you could not settle: one finding each with `state` `could-not-look` and the reason in `claim`.

## Rules

- Open every cited file and judge from the file, never from a quote alone. If a quote and the file disagree, the file wins.
- Apply the seven rules of `references/rea-method.md`: claim, check of equal strength, blocked, three lookup states, evidence plus unknowns, counter-case, work backwards. A finding with no runnable check is `could-not-look`, never `found`.
- Judge only the files in the change map and the readers listed for them. Never widen the scope to the whole repository.
- Do not look at the ledger and do not decide whether a finding was raised before; the loop does that. Report what you see now.
- Do not edit anything. One answer, then stop.

## Procedure

1. Read each map file and each cited reader span in full around the quote.
2. For each item of the lens checklist, ask the question of the change; keep only what the file supports.
3. For each candidate, write the claim, then the check, then the counter-case; drop the candidate if the counter-case holds.
4. Assign the class, severity and `needs_owner`; emit the lines.
