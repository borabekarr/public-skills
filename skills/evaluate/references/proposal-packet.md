# Proposal packet and verdict

Nothing that survives the run and will be read by future sessions is written until a packet for it is approved. The drafter writes the packet and stops; the approver replies with a verdict. The approver is the main agent or the human, never the agent that drafted the packet. If you have only one agent, draft in one pass, then review in a second pass that starts by reading only the packet and the quotes.

## Packet

```
PROPOSAL v1
id: <short-unique-id>
worker: <name of the drafter>
target: <path of the file to change in {{TARGET_SYSTEM}}>
operation: create | append | replace-section | delete
anchor: <heading or line the change attaches to, or none for create>
form: replaces <path> | merge_into <path> <heading> | additive <why nothing covers it> | revise_ours <path> <heading>: <one-line rule change>
size: <lines of content>
source: <where the idea came from: url or path and line range>
attestation: read-complete
quote-anchors:
- "<verbatim quote from the source, at most 25 words>"
- "<verbatim quote from the source, at most 25 words>"
content:
<<<
<the exact text to write>
>>>
rationale: <at most 3 lines>
our-version: as-written (<reason>) | modified: <what changed versus the source>
check-landed: <the command or search that proves it landed and is read by something>
undo: <how to revert, exactly>
```

Rules for the drafter:

- Two to four quote-anchors, copied verbatim, so the approver can confirm the source says what the packet claims.
- `our-version` must say what changed versus the source. A packet that is verbatim or near verbatim is rejected: adapt, never copy. The author's constants, thresholds and setup requirements are proposals; change them to fit `{{TARGET_SYSTEM}}` and record the change.
- `form` is required, exactly one of four:

| Form | Meaning |
|---|---|
| `replaces <path>` | Full supersession; the old file is removed. |
| `merge_into <path> <heading>` | Partial supersession; merged into that section. |
| `additive <why>` | Pure addition; says why nothing existing covers it. Log it in the decision record so a later audit can challenge it. |
| `revise_ours <path> <heading>: <change>` | Your own rule changes and their text is not copied. Anything that reads that heading is re-read in the same run. |

A chunk with no form fails and becomes a skip with the reason "missing form". The gate applies per chunk; one chunk failing never rejects its siblings.

- A delete packet carries the full bytes of the file being removed, one file per packet, so the approver sees the exact loss.
- A packet over about 300 lines is split into a create packet and append packets, each approved separately.
- Packets are Tier 1: anything that outlives the run needs one. Scratch notes and run logs do not.

## Verdict

```
VERDICT <id>: APPROVE
VERDICT <id>: REVISE: <guidance, at most 5 lines>
VERDICT <id>: REJECT: <reason, at most 3 lines>
```

Before approving, the approver runs these mechanical checks without trusting the packet:

1. Every quote-anchor is found in the source (exact text search).
2. For append and replace-section, the anchor exists in the target (`found`, with the line).
3. The `check-landed` command is runnable and would fail before the change.
4. The `undo` step is exact enough for someone else to run.
5. After the write, the real change is within about 10 percent of the declared `size`.

A failed check is grounds for REVISE, never a silent APPROVE.

## Reject loop

1. REVISE up to twice per packet; the drafter resubmits against the guidance.
2. On the third rejection, stop that drafter and start a fresh one with the full rejection history.
3. The fresh drafter gets two more tries.
4. If still rejected, record the chunk in the decision record as `skip` with reason `proposal-rejected` and the history, and move on.

## Fresh-eyes check (recommended for anything that will be read by many sessions)

A second reader with no memory of the draft greps each quote-anchor in the source and checks that the proposed content follows from the quotes. It answers `PASS` or `FAIL` in under ten lines. A `FAIL` blocks approval.
