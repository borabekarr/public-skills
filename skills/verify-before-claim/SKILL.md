---
name: verify-before-claim
description: Use before shipping any claim that names a thing to cut, remove, merge, replace, consolidate, or call duplicate, unused, dead or redundant. Requires an 11-item evidence card per claim, checked by an independent reviewer who fails the proposal if any item is blank or hand-waved.
---

# verify-before-claim

Credits: based on the claim, check and evidence method of morluto/rea (MIT); see NOTICE.md.

Starting state: a draft proposal that names an existing component (skill, agent, command, config, file, package, dependency, route, table, function) for removal, replacement, merger, or a "duplicate / unused / dead" label. Target state: every named component carries a filled evidence card, an independent reviewer has tried to falsify each card, and only claims that survived ship.

## Output

One card per challenged component, shown to the reader alongside the proposal (not summarised), filled before the rest of the proposal text:

```markdown
### Evidence card: <component-name>

| # | Check | Evidence |
|---|---|---|
| 1 | Description read in full | "..." |
| 2 | Triggers / when-to-use | ... |
| 3 | Source body | ... |
| 4 | Reverse references | ... |
| 5 | Change history | ... |
| 6 | Usage in notes and logs | ... |
| 7 | Dependencies | ... |
| 8 | Siblings | ... |
| 9 | Future relevance | database: ?, backend: ?, AI: ?, web: ?, mobile: ?, security: ?, ops: ?, QA: ? |
| 10 | Counter-example | ... |
| 11 | Owner context | ... |

**Claim:** <remove-from-project | merge | replace | keep>
**Confidence:** <high | medium | low>
**Blocker (if any):** <text>
```

## The contract

Each card has exactly eleven rows. A row cannot be blank. A row marked N/A says why it is N/A in the same row. The reviewer fails the proposal if a row is blank, hand-waved, or copied from a helper's summary instead of a fresh read, because summaries are where invented counts and wrong conclusions enter.

Terminology: "remove" means removal from this project (unlink or unregister it), not deletion of the source, unless the owner confirms deletion. Echo this in any proposal that uses informal words such as kill, drop or cut, so nobody reads a project-level removal as a destructive one.

## The 11-item checklist

| # | Check | Required output |
|---|---|---|
| 1 | **Description read in full** | Quote the description or header text verbatim |
| 2 | **All triggers or when-to-use entries examined** | List or quote them |
| 3 | **Source skimmed past the header** | Cite the line count plus one specific behavior observed in the body |
| 4 | **Reverse reference scan** | Quoted search results for the component's name across top-level instructions, settings, other components, scripts and references. Zero references strengthens the claim; one or more weakens or kills it |
| 5 | **Change history** | Quoted output of `git log --oneline -10 -- <path>`; note the last-touched date and commit subject |
| 6 | **Usage in notes and logs** | Searched project notes and logs (`{{WORK_DIR}}` or wherever your harness keeps them) for invocations. Quote any hit, or "no hits" with the search command cited |
| 7 | **Dependency map** | Links, imports and requires that pull this component in, quoted |
| 8 | **Sibling files in the same directory** | Listed, with a statement of whether the siblings form a unit (remove the unit or none) |
| 9 | **Future relevance** | For each phase on the card (database, backend, AI, web, mobile, security, ops, QA) mark yes, no or unknown. Unknown is a blocker; resolve it before the claim ships |
| 10 | **Counter-example hunt** | Actively search for one reason the claim is wrong. Quote the result even when empty ("searched X, found nothing") |
| 11 | **Owner-context check** | Did the owner mention this component in the conversation or in notes? If yes, quote it. If the owner said "keep", the claim ends there |

Edit the phases in row 9 to match your project; the rule that unknown blocks the claim stays.

## Procedure

1. **Read the pattern library.** Read `references/hallucination-patterns.md` end to end and check whether the current proposal matches any listed pattern. If it does, run that pattern's check on the proposal before going on.
2. **Fill one card per component.** Use primary sources: open the files, run the commands, paste real output. Counts in the proposal come from a command whose output you paste; otherwise write "[count uncited]" and resolve it.
3. **Hand the cards to an independent reviewer.** Spawn a fresh helper through `{{SUBAGENT}}`; if your harness has none, do a separate fresh pass that starts from the cards alone, without your drafting context. Give it the instruction below.
4. **Resolve findings.** A failed row gets re-done from the primary source. A claim the reviewer falsified is dropped or downgraded to keep.
5. **Ship.** Show the cards, the reviewer's result, and the surviving claims together.

Skipped items in a bulk review (for example inputs triaged out of a batch) get a lighter version: rows 1 to 3 plus a second look, described in `references/skip-review.md`.

## Reviewer instruction

Pass this to the reviewer verbatim:

> Falsify, do not approve. For each evidence card in this proposal: read the cited files yourself, run the cited commands, confirm the quoted outputs are real. Flag any row that quotes a helper's summary instead of a primary source. Flag any row marked N/A without a reason. If row 11 (owner context) names a message you cannot find in the conversation history, fail the card. A "looks good" return with no probes means the review failed and is re-run.

## Components already decided

If your project has recorded owner decisions (components that are locked to keep or locked for removal), list them here in your own copy of this skill and skip the full checklist for them by citing the decision. Any component not on that list runs all eleven rows.

## Failure shapes this prevents

| Failure shape | How the checklist catches it |
|---|---|
| Recommending a removal from a helper's one-paragraph summary | Rows 3 and 4 force fresh reads |
| Calling two components "duplicates" without comparing them | Rows 1, 2 and 3 produce a side-by-side that shows specialization versus duplication |
| Removing something a future phase needs | Row 9 forces yes, no or unknown per phase |
| Repeating a mistake already corrected | Row 11 checks for an earlier "keep" |
| Confident numbers with no source | Procedure step 2: counts need pasted command output |

## Adapt

Placeholders in this skill: `{{WORK_DIR}}` (where notes and logs live) and `{{SUBAGENT}}` (how to start a fresh helper). They are listed in `ADAPTING.md` at the repo root. See this folder's README for tuning by model and harness.
