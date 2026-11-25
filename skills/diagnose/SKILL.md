---
name: diagnose
description: Use when a bug is hard or intermittent and needs a reproduce, minimise, hypothesise, instrument, fix, regression-test loop (flaky tests, performance regressions, "works on my machine"). For a known error with a stack trace and no reproduction problem, fix it directly instead.
---

# Diagnose

Credits: based on the diagnose skill by Matt Pocock (MIT) and debugging techniques from obra/superpowers (MIT); see NOTICE.md.

A discipline for hard bugs. Skip a phase only when you can say why it adds nothing.

## Output

Report in this shape when done (earlier phases report only the parts they own):

```markdown
### Diagnosis: <one-line symptom>
- **Loop:** <command or script that reproduces it, and its pass/fail signal>
- **Hypotheses:** <3-5 ranked, each with its prediction; mark the confirmed one>
- **Root cause:** <where the bad value or state originates, with file and line>
- **Fix:** <what changed and why it fixes the cause, not the symptom>
- **Regression test:** <test name and seam, or "no correct seam exists" with the reason>
- **Cleanup:** <debug tags removed, prototypes deleted>
- **Prevention:** <what would have stopped this bug, given after the fix>
```

Start state: a reported bug with no trusted way to reproduce it. Target state: the bug reproduces on demand, the cause is named and confirmed by a probe, the fix is verified by the same loop, and a regression test or a documented reason for none exists.

Before exploring, read the modules around the symptom so you work from a mental model rather than from guesses.

## Phase 1: Build a feedback loop

**This is the skill.** With a fast, deterministic, agent-runnable pass/fail signal, bisection, hypothesis testing and instrumentation all just consume that signal. Without one, reading code harder rarely helps, because nothing tells you whether a theory is right. Spend disproportionate effort here and be creative.

Ways to construct one, roughly in this order:

1. **Failing test** at whatever seam reaches the bug: unit, integration, end to end.
2. **HTTP script** against a running dev server.
3. **CLI invocation** with a fixture input, diffing output against a known-good snapshot.
4. **Headless browser script** that drives the UI and asserts on DOM, console or network.
5. **Replay a captured trace.** Save a real request, payload or event log to disk and replay it through the code path in isolation.
6. **Throwaway harness.** A minimal subset of the system (one service, mocked dependencies) that exercises the bug path with a single call.
7. **Property or fuzz loop.** For "sometimes wrong output", run 1000 random inputs and look for the failure mode.
8. **Bisection harness.** If the bug appeared between two known states (commit, dataset, version), automate "boot at state X, check, repeat" so `git bisect run` can drive it.
9. **Differential loop.** Run the same input through old and new versions, or two configs, and diff the outputs.
10. **Human-in-the-loop script.** Last resort. If a person must click, drive them with `scripts/hitl-loop.template.sh` so the loop stays structured and the captured output comes back to you.

### Iterate on the loop itself

Treat the loop as a product. Once you have one, ask:

- Can it be faster? (Cache setup, skip unrelated init, narrow the test scope.)
- Can the signal be sharper? (Assert on the specific symptom, not "did not crash".)
- Can it be more deterministic? (Pin time, seed the random generator, isolate the filesystem, freeze the network.)

A 30-second flaky loop is barely better than none. A 2-second deterministic loop makes the rest easy.

### Non-deterministic bugs

The goal is a higher reproduction rate, not a clean repro. Loop the trigger 100 times, parallelise, add stress, narrow timing windows, inject sleeps. A 50% flake is debuggable; a 1% flake is not until you raise the rate. For timing-related flakes see `references/condition-based-waiting.md`.

### When you cannot build a loop

Stop and say so, list what you tried, and ask the user for one of: access to an environment that reproduces it, a captured artifact (HAR file, log dump, core dump, screen recording with timestamps), or permission to add temporary instrumentation in production. Hypothesising without a loop produces theories you cannot test, so wait for the loop before Phase 2.

## Phase 2: Reproduce

Run the loop and watch the bug appear. Confirm:

- [ ] The loop shows the failure the **user** described, not a different failure nearby. A wrong bug leads to a wrong fix.
- [ ] It fails across multiple runs (or, for non-deterministic bugs, at a rate high enough to debug against).
- [ ] You captured the exact symptom (error message, wrong output, slow timing) so later phases can check the fix addresses it.

Move on only once the bug reproduces; otherwise every later result is uninterpretable.

## Phase 3: Hypothesise

Write **3-5 ranked hypotheses** before testing any, because a single hypothesis anchors you on the first plausible idea.

Each one must be falsifiable and state its prediction:

> "If <X> is the cause, then <changing Y> will make the bug disappear / <changing Z> will make it worse."

A hypothesis with no prediction is a vibe; sharpen it or drop it.

Also compare against a working example: find similar code that works and list every difference from the broken path, alongside any old-versus-new diff. To trace a bad value back through the call chain, see `references/root-cause-tracing.md`.

Show the ranked list to the user before testing. They often know something that re-ranks it at once ("we just deployed a change to #3") or have already ruled items out. It is a cheap checkpoint; if the user is away, proceed with your ranking.

## Phase 4: Instrument

Map each probe to a specific prediction from Phase 3 and **change one variable at a time**, because changing several at once hides which one explains the result.

Tool preference:

1. **Debugger or REPL inspection** if the environment supports it. One breakpoint beats ten logs.
2. **Targeted logs** at the boundaries that distinguish hypotheses.
3. Avoid "log everything and grep": it buries the signal instead of testing a prediction.

**Tag every debug log** with a unique prefix such as `[DEBUG-a4f2]`, so cleanup is a single grep and none survive by accident.

**Performance branch.** Logs mislead on performance regressions. Establish a baseline (timing harness, profiler, query plan), then bisect. Measure first, fix second.

## Phase 5: Fix and regression test

Write the regression test **before the fix**, but only at a **correct seam**: one where the test exercises the real bug pattern as it occurs at the call site. A seam that is too shallow (a single-caller test when the bug needs several callers) gives false confidence.

If no correct seam exists, that is itself a finding: the architecture prevents locking the bug down. Note it for the cleanup phase.

With a correct seam:

1. Turn the minimised repro into a failing test there.
2. Watch it fail.
3. Apply the fix.
4. Watch it pass.
5. Re-run the Phase 1 loop against the original, un-minimised scenario.

Once the fix lands, validate at every layer the bad data passed through, not only the one nearest the fix: `references/defense-in-depth.md`.

## Phase 6: Cleanup and review

Before declaring done:

- [ ] The original repro no longer reproduces (re-run the Phase 1 loop).
- [ ] The regression test passes, or the absence of a seam is documented.
- [ ] All `[DEBUG-...]` instrumentation is removed (grep the prefix).
- [ ] Throwaway prototypes are deleted or moved somewhere clearly marked as debug.
- [ ] The hypothesis that proved correct is stated in the commit or PR message, so the next debugger learns from it.

Then ask what would have prevented this bug. If the answer involves architecture (no good test seam, tangled callers, hidden coupling), raise it with specifics. Make that recommendation after the fix, when you know more than you did at the start.

## Related files

- `references/root-cause-tracing.md`: trace backward to the original trigger instead of fixing where the error surfaces (Phases 3 and 4).
- `references/condition-based-waiting.md`: replace an arbitrary sleep with a poll on the real condition when the loop is flaky under timing (Phase 1).
- `references/defense-in-depth.md`: validate at every layer after the fix (Phase 5).
- `scripts/hitl-loop.template.sh`: scaffold for a human-driven loop (Phase 1, option 10).

## Adapt

Gates that block a change (lint, type check, test suite) are whatever your test runner or CI provides; use those as the pass/fail signal where they fit. The folder README (optional) has tuning notes per model and harness.
