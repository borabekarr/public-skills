# evaluate

A skill that decides, chunk by chunk, whether an outside resource (an article, a repo, a skill, a prompt pack) is worth adopting into your own system. It scans for safety, scores each chunk with the IRS rubric (Impact, Relevance, Specificity), proposes the adapted change for approval, checks it landed, and records the decision.

## What it does

- Reads the resource in full and scans it for injected instructions, remote execution and credential theft before scoring.
- Splits it into named chunks so one weak part never hides a strong one, and never skips a whole source.
- Scores each chunk 1 to 5 on Impact, Relevance to your target system, and Specificity; passes at 11 of 15 with no score under 3.
- Issues take, partial or skip per chunk, with a named reason and a lookup state (`found`, `nothing`, `could-not-look`).
- Drafts a proposal packet for every take: what changes, where, why, how to check it landed, how to undo it. Nothing is written before approval.
- Optionally runs a blind A/B trial against the plain model.
- Proves the change is wired in, has a second reader re-check the skips, and appends a decision record.

## When to use

- Someone shares a link and asks "should we use this?".
- Before copying a prompt, skill or workflow from the internet into your setup.
- When reviewing a bundle of tools and only some parts may fit.

## Pick your target system

The skill scores everything against one thing, `{{TARGET_SYSTEM}}`: the project, agent setup or workflow you are trying to improve. Examples: "my team's coding-agent setup (project instructions, skills, notes)", "our customer-support prompt library", "a personal research workflow". Replace `{{TARGET_SYSTEM}}` in your copy of the skill with a one-line description and the path where its files live, or state it at the start of each run. Relevance scores are only meaningful against a concrete answer; "everything" or "generally useful" is not one.

## Install

Copy the `evaluate` folder into the skills directory your harness reads. Copy only: if a skill with the same name already exists there, keep it and use another folder name instead of replacing it. Never delete anything to make room.

## Files

| File | Purpose |
|---|---|
| `SKILL.md` | The ten-step flow, take, partial and skip verdicts, decision record format |
| `roles/worker.md` | Role for the helper that reads, scores and drafts packets |
| `references/irs-rubric.md` | Adaptation Sketch, the three scores, the threshold |
| `references/rea-method.md` | Claim, check, counter-case, evidence and the three lookup states |
| `references/security-scan.md` | Eight-pattern safety checklist for every resource |
| `references/decomposition.md` | How to split a resource into chunks |
| `references/proposal-packet.md` | Packet format, verdicts, reject loop |
| `references/trial-ab.md` | Optional blind A/B trial |
| `references/integration-check.md` | Proof that a change landed and is read by something |
| `references/skip-review.md` | Second look at skips to catch false negatives |
| `references/take-review.md` | The same for high-scoring takes |
| `references/examples.md` | Three worked runs |

Each file stands alone inside this folder. The only pointers outside it are the repo-root `ADAPTING.md` (the shared placeholder list, also given in SKILL.md) and `NOTICE.md` (credits); the skill runs without either.

## Adapt to your model and harness

Works with Claude, Codex, Gemini, Grok, or any other agent that can read files and search. Placeholders used here, defined in `ADAPTING.md` at the repo root:

- `{{TARGET_SYSTEM}}` is what you score against (see above).
- `{{WORK_DIR}}` is where the decision log and review ledgers live (default `.skill-work/evaluate/`).
- `{{PROJECT_ROOT}}` is the root of the project searched for consumers.
- `{{SUBAGENT}}` is how to start a fresh helper for the worker and reviewers. If your harness cannot, run each role as a separate fresh pass that starts without your drafting context.
- `{{STRONG_MODEL}}` and `{{FAST_MODEL}}` name the models for judging and for cheap passes.

What to tune:

- **Model strength.** A strong model can score all chunks in one pass. A smaller model does better one chunk at a time. Whatever you use, the judge in the trial and the final skip judge should be at least as strong as the models being judged.
- **Whether helpers can run.** The independent reviewer is the point of the skip review. The same agent re-reading its own work is a weak substitute, so use a separate session or model when you can.
- **The threshold.** 11 of 15 with nothing under 3 is a default. Raise it if adoption is costly, lower it if changes are easy to undo.
- **Trial and sandbox.** The A/B trial needs somewhere safe to run a candidate. If you have no isolated place, skip the trial and rely on the scores.
- **Harness specifics.** Install paths, tool names and helper mechanics differ per harness. Check your harness's own documentation instead of trusting this README.

## Credits

Credits: the claim, check and evidence method is based on morluto/rea (MIT); see NOTICE.md.
