# IRS: Integration Readiness Score

A three-dimension, five-point rubric that decides whether a chunk of an outside resource is worth adopting into `{{TARGET_SYSTEM}}`. Each dimension scores 1 to 5; the three add to a total out of 15. The rubric is identical for a whole file and for a single chunk; only the unit changes.

A resource built for a different setup is not rejected for that. Work out how it adapts to yours, then score your adapted version, never the original.

## Step 0: Adaptation Sketch (before any score)

Write seven lines first:

- `Pattern:` the environment-independent idea, one sentence.
- `Our equivalent today:` the existing part of `{{TARGET_SYSTEM}}` that overlaps, or `none`. Search to confirm; never guess.
- `Situational fit:` how your situation differs from the author's: who uses it, at what scale, which tools you run or refuse, which rules you already hold. One or two sentences.
- `Our version:` the modified idea for you: changed constants, dropped or substituted requirements, merged steps. The author's exact form is one candidate and must be argued for if you choose it.
- `Framework delta:` `none`, or the path of an existing principle or file of yours that would change, plus the one-line change.
- `Adapted form:` the target path and what changes to fit it. Two to four sentences.
- `Cost:` files touched and consumers to wire, one line.

If `Our equivalent today` is not `none`, score the chunk as a delta over that equivalent, not in isolation. A chunk that only restates what you already do scores low on Impact. "We already have one" lowers Impact only when yours is at least as good at the same goal, judged against the goal and not by the fact that it exists.

## Dimensions

Each score has a one-sentence justification citing a line range from the source, plus one counter-argument: what would make it a point lower.

### I, Impact

How much would your adapted version improve the workflow or project?

- 1 = negligible
- 2 = minor convenience
- 3 = moderate improvement to one workflow or feature
- 4 = meaningful improvement across several areas
- 5 = significant improvement

When `Framework delta:` is not `none`, score Impact on the improvement to your own material: revising yours can outscore adding theirs.

### R, Relevance to {{TARGET_SYSTEM}}

How cleanly does your adapted version transfer into `{{TARGET_SYSTEM}}`? This is about transferability, not about matching the source's stack.

- 1 = no transfer path; you must cite a named blocker (license conflict, platform lock-in, contradicts a rule you hold, needs infrastructure you will not run)
- 2 = transfers only with a rewrite that loses the value
- 3 = transfers with a moderate rewrite
- 4 = drops in with light edits
- 5 = drop-in

A technology label (a vendor, a framework, a language) is never a blocker by itself. A requirement of the author's setup (a paid API, a hosted service, a team size) is not one either, unless a blocker survives after your version drops or substitutes that requirement. Colliding with or duplicating your current approach to the same job is also not a blocker: record it as the `Framework delta:` and score on whether the resource's method does the job better.

### S, Specificity

Is your adapted version actionable as a concrete step?

- 1 = vague or theoretical
- 2 = ideas, nothing concrete
- 3 = partly actionable, needs adaptation
- 4 = mostly ready, minor adaptation
- 5 = ready to apply directly

## Threshold

Pass: total at least 11 of 15 AND no single score below 3. A 12 with S=2 fails. An 11 with all three at 3 or above passes. Do not round up borderline scores. The same gate applies to whole files and to chunks.

## Applying it

1. Read the unit end to end. No sampling. Record how much you read (`full`, `head`, `description-only`); only `full` can be scored.
2. Write the Adaptation Sketch and run the overlap check.
3. Score I, R, S on your version, independently, each with a line range and a counter-argument.
4. Add them up and compare with the threshold.
5. Pass: take or partial. Fail: log the sketch and the reason, then skip.

## Output shape

```
Pattern: <the idea>
Our equivalent today: <path or none>
Situational fit: <how you differ from the author>
Our version: <the modified idea>
Framework delta: <none, or path and the change>
Adapted form: <target and changes>
Cost: <files, consumers>
I=<n>, <justification with line range>
counter: <what would make it a point lower>
R=<n>, <justification with line range>
counter: <...>
S=<n>, <justification with line range>
counter: <...>
Total=<n>/15
```

For part-level runs, print one table row per chunk (number, chunk slug, your version in a few words, I, R, S, total, verdict), then one plain-language justification per chunk that passes.
