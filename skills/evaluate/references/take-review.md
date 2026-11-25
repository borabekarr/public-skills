# Take review

`references/skip-review.md` samples the skips. This file adds the take side, so a high score is never only the scorer's word. The IRS math is unchanged (`references/irs-rubric.md`).

Reviewer reply shape, one line per sampled take: `unit | VERDICT | evidence`, VERDICT being AGREE, DISAGREE or UNVERIFIED.

## 1. Sampler addition

The skip-review sample also holds every take or partial with a total of 13 or more. The sample is frozen at the first run, and the 15-units-per-call limit still applies.

## 2. Reviewer duties

For each sampled take, the reviewer (same brief and hard read rule as the skip reviewer):

- Argues against the take first: states the strongest reason it should be a skip or a smaller partial.
- Cites one uncovered part before it may AGREE: a part of the source that neither the score nor the Adaptation Sketch accounts for, as `path:lines`. If it cannot, the verdict is DISAGREE or UNVERIFIED.
- Replies in the same line format.

DISAGREE goes through the re-read and judge path in the skip-review contract.

## 3. Closest neighbor

Every take names the closest existing material in `{{TARGET_SYSTEM}}` with a quote: the file path plus one verbatim line from it. If nothing is close, write `nothing` and say where you looked.

## 4. Lookup states

Every coverage, duplicate, prior-decision or availability lookup (a package registry, a release tag, an installer) ends in exactly one state, written on the line that reports it:

- `found`: the path and line where it was found.
- `nothing`: the paths or queries searched, so the absence can be re-checked.
- `could-not-look`: why the lookup could not run.

Silence is not `nothing`. See `references/rea-method.md`, rule 4.

## 5. Additions log

An `additive` take is logged where a later audit can find it. Keep `{{WORK_DIR}}/evaluate/additions.jsonl`, one JSON object per line with these keys: timestamp, source, target_added, justification, irs_score, and a unit id of the form `<resource>:<chunk>`. The audit asks, for each row, whether something existing could have been extended instead.
