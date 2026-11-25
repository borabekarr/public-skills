# Fix plan template

A fix plan turns one round's open findings into steps that can be run and checked. Name it `{{WORK_DIR}}/redteam/fix-<topic>-r<n>.md` for round n. A plan for a killed loop is `{{WORK_DIR}}/redteam/fix-<topic>-loop.md`.

## Fill rules

- One step per open finding. Number the steps in order. Repeat the step block as often as needed.
- `<files>` are repo-relative paths, the files the step may edit (and only those).
- `<check>` is a runnable command or a precise manual check that fails before the fix and passes after it, as strong as the finding's claim. A check that cannot fail does not count.
- The counter-case names the cheapest situation in which the finding would be a false positive and what the check shows there. If the check passes on the unfixed tree, the finding is rejected, not fixed.
- Name each stale spot by its quoted text and its file. A line number is only a hint: the fix may run after other edits shifted the lines.
- The plan never starts a redteam. It carries the no-recursion line below. The next round is started by the loop, not by the fix.
- No commits inside the plan. The final round, when the loop ends, is the only one that commits.
- A loop-bug plan adds a last step whose check replays the looping state (for example, count the ledger again on the same table).
- After the fix runs: mark each fixed key `fixed` in the ledger, add 1 to its `fixed x`, and store its anchor text as it stands after the fix (`references/ledger.md`).

## Template

```markdown
# Redteam fix <topic> round <n>

Run: <ns>    Lens: <lens>    Round: <n>
No-recursion: this plan never starts a redteam. Do not commit.

## Context

Round <n> found gaps in the change `<topic>` and what it touches. Each step below fixes one finding. The next round re-checks the result.

## Steps

### Step 1: Fix <finding title>
- Key: <lens> + <class> + <files> + <anchor>
- Files you may edit: <files>
- Claim: <one sentence that could be false>
- Check: <command or precise check>
- Counter-case: <cheapest situation where the finding is wrong, and what the check shows there>
- Done when: the check exits 0 and the anchored text no longer shows the gap.

### Step 2: Fix <next finding title>
(same fields)

## Report back

For each step: fixed or not, the check output, and the anchored lines after the fix.
```
