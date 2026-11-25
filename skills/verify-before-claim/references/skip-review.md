# Skip review

When a batch of items is triaged and some are skipped, the person or agent that decided to skip is the worst judge of whether the skip was wrong. A skip review is a second look at the skipped items, aimed at false negatives: items that were useful and got dropped.

Use it when the verify-before-claim checklist is applied to skips (rows 1 to 3) and the cost of a full card per item is too high.

## Output

One line per reviewed skip:

```markdown
| Item | Skip reason given | Second-look result | Verdict |
|---|---|---|---|
| <item> | <reason the first pass gave> | <what the item actually contains, one specific fact from its body> | stands / reverse |
```

## Procedure

1. **Record the reason.** Every skip carries a one-sentence reason and a pointer to the part of the item that supports it (file and line range). A skip with no reason is returned to the first pass.
2. **Name the difference.** A skip labelled "duplicate" or "already have it" must say which part of the item differs from what exists and why that part is not taken. Skipping a whole item because it overlaps in part loses the useful remainder.
3. **Sample.** Review every skip in risky classes (security, "low impact", "no place to put it") and every item that was only skimmed. Add a random quarter of the remaining skips, at least three.
4. **Independent read.** A fresh helper (`{{SUBAGENT}}`, or a separate fresh pass) reads each sampled item in full, not the first pass's summary, and states the verdict. It looks for the reason to reverse, not for agreement.
5. **Reverse or keep.** A reversed skip goes back into the batch. A stands verdict is recorded with the specific fact that justified it.

Cap each review call at about fifteen items; split larger samples across calls so the reader does not skim.
