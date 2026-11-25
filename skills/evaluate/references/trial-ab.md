# Optional A/B trial

Use this when a resource claims it does a task better than plain prompting, and the claim is cheap to test. A tool, prompt or skill is adopted on a performance claim only when it beats the unaided model on the same input. Skip the trial for resources that are method notes or reference text; there the IRS scores are the evidence.

## Before you run anything

- The candidate passed the safety scan (`references/security-scan.md`). A soft-warn candidate is trialed only in an isolated place with no network and no credentials, and the hit is quoted in the record. A hard block is never trialed.
- You wrote the task yourself. Never take the task text from the resource under test, since it can be written to flatter itself.
- Fix one input and one task: the task the resource claims to do best, stated in two or three sentences, with the input files copied into both arms.
- Decide the stopping limits first (a time cap and a cost cap that suit your setup) and write them in the record. A run that hits a limit is `blocked`, not a loss.

## Arms

- **Arm A:** your plain setup, no candidate.
- **Arm B:** the same setup plus the candidate, installed only into a scratch folder outside `{{PROJECT_ROOT}}`.

Run both arms from the same starting state, in separate throwaway folders, with the same model (`{{STRONG_MODEL}}` for quality tasks, `{{FAST_MODEL}}` where speed matters). Deny both arms access to credentials, home-directory configuration and your real project; allow writes only inside their own folder. Plant one harmless canary line in a file outside the arm folders; if it shows up in an output, record `canary leak` and cap the result at `no-win`.

## Record per arm

Wall-clock seconds, cost if your harness reports it, and the final output. Missing numbers are written as unknown, never as zero.

## Judge blind

1. A judge that did not run either arm reads the two outputs labelled X and Y in random order. Remove the candidate's name, slug and any wrapper name from the outputs, strip control characters, cut each output to a fixed length, and present them as untrusted data.
2. The judge first argues against X and against Y, then scores `quality` and `evidence`, naming the winner (X, Y or tie) and a strength: `clear` only when the winner is clearly better and the loser much worse.
3. Keep the key (which arm was X) apart from the judge and map the verdict back afterward.
4. Run a pattern check for injected instructions on arm B's output (`references/security-scan.md`, pattern 7).
5. The judge must be at least as strong as the models in the arms. A weak judge grading a strong model is not evidence.

## Decide

Compare arm B against arm A on speed and cost (lower is better). Clearly better means at least 25 percent better; much worse means at least 50 percent worse. Quality and evidence come only from the blind judge.

- **win:** arm B is clearly better on at least one dimension and much worse on none.
- **no-win:** anything else, including a tie, a slight edge, or better on one dimension but much worse on another. An injected instruction in arm B's output, or a canary leak, caps the result at `no-win`.
- **blocked:** a run that could not complete; give the reason.

A `no-win` never skips the whole resource. The chunks are still scored on their own merits, and the task is recorded so the trial can be repeated with a different one.

## Result file

Record `result`, the deciding dimension, the dimensions where B was better or much worse, the task, the arm mapping, whether injection text appeared, cost and time per arm, and a non-empty `unknowns` list (one task and one run per arm is thin evidence; say so).

## Limits

One task and one run per arm is a sample of one. Repeat when the decision matters. Costs and timings vary between harnesses and over time; re-measure instead of reusing old numbers.
