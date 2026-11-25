# Skip review

The agent that scores a resource must not be the only reader of its skips: a confident wrong skip silently throws away something useful. This file is the whole contract for that second look. It needs no script; the ledgers are small tables you keep in `{{WORK_DIR}}/evaluate/<slug>/`.

## 1. Partial rule

- No unit is skipped as "we already have it" or "duplicate" until the scorer names which part of the unit differs from your equivalent, and says why that part is not taken or adapted.
- Every run summary prints one line: `Partial rule: N partial, M adapted, K skipped whole`.

Scoring follows `references/irs-rubric.md`; do not restate it in briefs.

## 2. Coverage ledger

Before scoring, list every unit of the resource, one per line. The scorer then writes `coverage.tsv` with columns: `unit, files, bytes, read_mode (full|head|description-only), line_ranges, verdict (scored|skipped|unread)`.

A unit may be scored or skipped only when `read_mode` is `full`; otherwise its verdict is `unread` and the summary lists it as not read. A manifest from a quick overview only locates units; the scorer still reads each one.

## 3. Skip ledger

The scorer writes `skips.tsv` with columns: `unit, reason_class, resource_anchor, our_anchor, salvaged, plain_reason, destinations_checked`.

- `reason_class` is one of `dup`, `have-equivalent`, `blocked`, `low-impact`, `security`, `no-destination`.
- `resource_anchor` is a source-relative `path:lines`.
- `our_anchor` is your own `path:lines`; required for `dup` and `have-equivalent`.
- A `dup` or `have-equivalent` row's `plain_reason` must carry a `Differs:` clause naming what the unit has that yours lacks.
- `plain_reason` is one or two short plain sentences someone with no context can read.
- `destinations_checked` lists where else the unit could be useful (another project of yours, a watch list for later, a different part of `{{TARGET_SYSTEM}}`), or `all`. Use `all` only after really checking.
- `salvaged` names the part you kept, or `none`.

## 4. Sample

Choose the units the reviewer must check before it sees any rationale. The sample holds:

- every edge score (Impact 3, Relevance 3, or a total of 9 to 12)
- every `dup` and `have-equivalent` skip
- every unit read only in part
- every `security`, `low-impact`, `blocked` and `no-destination` skip
- about a quarter of the remaining skips, at least three, chosen by a random draw rather than by hand

Freeze the sample at the first run; a re-read by the scorer never redraws it. At most 15 units per reviewer call; split a larger sample across calls. Runs where everything was skipped are reviewed the same way.

## 5. Reviewer brief

Start one fresh, read-only reviewer with `{{SUBAGENT}}` (no memory of the scoring, and no tool that can write or run commands, since it reads stranger text). Without a helper, do the review yourself in a separate pass that begins by closing your drafting notes.

The reviewer reads the Adaptation Sketch, the score table, `skips.tsv`, `coverage.tsv` and each sampled unit. For each sampled unit it reads the cited ranges plus the unit's whole file (up to 200 lines), and for `dup` and `have-equivalent` rows it also opens your cited equivalent. It applies these checks:

- Quote the unit's own description verbatim; list every trigger or when-to-use entry; read past the opening and cite the line count plus one specific behavior from the body.
- Put the unit and your equivalent side by side. Confirm they serve the same goal and that yours is not worse on the part the unit adds.
- Confirm each skip states what was tried, names a real blocker (not a technology label or an author's setup requirement), and carries a salvage entry.

Before it may agree with a skip it answers in writing: "What is the cheapest situation where this skip is wrong, and where would the unit then be useful?" It also names one part of the unit the anchor does not cover. If it cannot do both, or is unsure, it never agrees to a drop: the verdict is DISAGREE or UNVERIFIED, and it says whether the unit should be trialed or kept for another destination.

Hard read rule: the reviewer opens each sampled unit and each cited equivalent itself. The sketch and the ledgers are claims under test, not evidence. A unit it did not open is UNVERIFIED. Its reply ends with the line `# reads: N`; a reply with fewer reads than sampled units is run again.

Output: one line per unit, exactly `unit | VERDICT | evidence`, VERDICT being AGREE, DISAGREE or UNVERIFIED, at most 25 lines.

## 6. Disagreement

- On DISAGREE the scorer re-reads that unit once, then fixes the verdict or defends it with a new anchor.
- A unit still disputed, every UNVERIFIED unit, and a whole resource where more than 20 percent of the reviewed units are DISAGREE go to a final judge: one fresh read-only reader, at least as strong as `{{STRONG_MODEL}}`.
- Give the judge a case file per unit: an excerpt, then Position A and Position B (each side's points and anchors) in random order with no role labels. Record which side is which and map the verdict back afterward.
- The judge returns `unit | TAKE|PARTIAL|SKIP|TRIAL|KEEP-FOR-LATER | reason`. Its call is final; there is no automatic re-score.

## 7. Summary block

Every run summary prints, in order:

1. `Partial rule: N partial, M adapted, K skipped whole`
2. `Read in full: n/m units (p%), h head-only, u unread`
3. `Coverage: PASS|FAIL`. FAIL when any listed unit is missing, or unread units exceed 10 percent.
4. `Reviewed: k units, a agree, d disagree, j judged`
5. A skip block grouped by reason class, one plain line per skip with the reviewer and judge verdict.
6. Unknowns and the lookup end states.

## 8. Order

Review and judge finish before the decision record is closed. Keep `coverage.tsv`, `skips.tsv`, the review and the judge output beside the decision record: they are what lets a later reader re-check a skip.
