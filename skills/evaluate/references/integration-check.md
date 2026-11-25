# Integration check

After an approved write, prove two things: the change landed as approved, and something actually uses it. A file that nothing reads changes nothing. Run this once per take or partial, before the decision record is closed.

## 1. Landed

Run the `check-landed` line from the packet and keep its output. Then confirm:

- The target exists and holds exactly the approved text (compare to the packet `content`, not to your memory of it).
- Any structured file you touched still parses (JSON, YAML, front matter).
- Any index, table of contents or registry row that the packet named is present exactly once.
- The change size is within about 10 percent of the packet's declared `size`.

A failed check means revert with the packet's `undo` line, record `verify-failed`, and stop. Never leave a half-applied change.

## 2. Wired

Name at least one consumer: a skill, command, agent, prompt, hook or document in `{{TARGET_SYSTEM}}` that reads or invokes the new or changed material. Prove it with a search for the target's file name or heading, quoting the matching line. If there is no consumer, add the missing reference in the same run (with its own packet if it is a lasting change), or downgrade the verdict to a note and say so.

Record `consumer: <path:line>` in the decision record.

For `replaces`, search for the removed file's name across `{{PROJECT_ROOT}}` and fix every dangling reference. For `revise_ours`, re-read every consumer of the changed heading and confirm each still agrees with the new rule.

## 3. Neighborhood

Look at the files next to the target for contradictions: two rules that now disagree, two files that now describe the same job. Write the findings as a short list; do not auto-fix, since a fix is a new proposal.

## 4. Closing the loop

- `landed:` in the decision record carries the check you ran and its result.
- Anything you could not verify goes to the unknowns list, with what would verify it.
- If nothing was integrated in the run (all skips), say so in one line; the integration check is then not applicable.

## Edge cases

- **Paywalled or unreadable source:** never score it. Ask the user to paste the content, and give only a thin note until they do.
- **A thread or multi-part post:** treat each part as a chunk and read all of them.
- **A duplicate of something you hold:** follow the partial rule (`SKILL.md`): name the part that differs before skipping.
- **A borderline score:** it is a miss. Record the counter-argument that decided it.
