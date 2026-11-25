# Hallucination patterns

A library of failure shapes an agent produces when it proposes removing, merging or replacing something, or reports on a source it did not fully read. Read it end to end before filling an evidence card, and check whether the current proposal is about to repeat any shape. If one matches, run its check first and rewrite the card with the new evidence.

Each pattern has the same three labelled lines:

- `Claim:` the statement as it tends to appear in a draft.
- `Missing evidence:` what was never actually looked at.
- `Check:` the concrete step that exposes the gap.

Add a new pattern whenever a review catches a shape that matches none of these. Keep the three-line form and keep nouns neutral (a config file, a docs page, a helper script).

## Pattern: Trusting a helper's summary

Claim: "A helper reports that many of these components are unused, so this list of removals is safe."
Missing evidence: No cited source file was reopened. The helper read only headers and inferred non-use from a short window of history.
Check: For each named item, list the file, search the whole project for its name, and read its change history; quote all three outputs on the card.

## Pattern: Similar names taken for duplicates

Claim: "These four components share a prefix, so they overlap and can be merged into one."
Missing evidence: No comparison of what each one actually checks. A shared naming convention shows hierarchy, not redundancy.
Check: Place the two descriptions side by side and name one problem each catches that the other does not. If you can name one in either direction, they are specialists and the merge claim is dropped.

## Pattern: Absence of references taken as safety

Claim: "Nothing references this config file, so it can be removed."
Missing evidence: The search never opened the places where wiring lives (settings files, build config, scripts, other components), so "no references" meant "searched in too few places".
Check: Name every file and pattern searched, include settings and build files and scripts, and label the claim low confidence until that scope is stated.

## Pattern: Unsourced count

Claim: "The project has dozens of docs pages and most are stale."
Missing evidence: No command produced the number, and "stale" has no definition such as "not changed in N days".
Check: Paste the command and its output for every number (for example a directory listing piped to a line count), and state the criterion behind any qualifier before the count is used.

## Pattern: Summary written without reading the source

Claim: "This source says X, so the summary entry records X."
Missing evidence: Only the opening section or headers were read; the rest of the summary was inferred and sounds plausible.
Check: State that the whole source was read, then for each claim in the summary point to the line or section that supports it. A claim with no locatable supporting line is removed or marked unverified.

## Pattern: Fix that moves a problem

Claim: "Replacing this helper script with that one removes the overlap."
Missing evidence: The replacement was never run on the cases the original handled, so its edge behavior is unknown.
Check: Run both on the same inputs, diff the outputs, and list any case where they differ before calling the replacement equivalent.

## How to use this file

1. Read it before drafting any evidence card.
2. For each pattern, ask whether the current proposal is about to commit that shape.
3. On a match, stop, run that pattern's check, and rewrite the card with the result.
4. A new shape that matches none is appended as a new `## Pattern:` section with the same three lines.
