# Worked examples

Three neutral runs. `{{TARGET_SYSTEM}}` is a small team's coding-agent setup: `PROJECT.md`, `notes/working-rules.md` and `notes/testing.md`. Scores are illustrative. Each carries a line range and a `counter:` line.

## Example 1: a blog post with one useful chunk

**Input:** a long blog post on "getting better results from coding agents".

**Intake:** fetched in full, 420 lines, no license stated. Decision log checked for the URL: `nothing` (searched the decision log).

**Safety scan:**

skill-scan: 0 clean (patterns 1-6 and 8 not found)

One quoted prompt-injection example in a sidebar (post lines 301-304): pattern 7 soft warn, quoted as data, not followed.

**Decompose:** part-level. Chunks: `post:intro-story` (lines 1-40), `post:tip-small-tasks` (90-130), `post:tip-read-errors-first` (188-221), `post:ask-for-a-plan-first` (222-260), `post:model-comparison` (300-360), `post:sponsor-section` (400-420).

**Sketch and scores for chunk 3, the take:**

```
Pattern: read the failing output in full before editing any code.
Our equivalent today: none (searched notes/working-rules.md and PROJECT.md for "error" and "failing output": nothing)
Situational fit: the author works solo; we run several agents on shared code, so a skipped read costs more.
Our version: one rule sentence; the three-step ritual is cut to step one, since steps two and three repeat PROJECT.md:20.
Framework delta: none, no existing file states this rule.
Adapted form: one added line under "Debugging" in PROJECT.md, our wording.
Cost: one file touched, one consumer (the include list at PROJECT.md:8).
I=4, a missed error message caused most repeat fixes (post lines 195-203); target PROJECT.md Debugging section.
counter: one point lower if our agents already read errors by habit.
R=5, a single sentence drops into the existing Debugging section (post lines 205-210).
counter: one point lower if PROJECT.md were near its size limit.
S=4, the rule is concrete and checkable by reading a transcript (post lines 212-221).
counter: one point lower because "in full" needs a definition for long logs.
Total=13/15
```

**Scores for the skipped chunks.** Each had the same seven-line sketch; shown compactly as sketch fields, then scores.

```
chunk 1 post:intro-story
Pattern: a story about a failed project. Our equivalent today: none (searched PROJECT.md). Situational fit: n/a. Our version: no change, no rule to adapt. Framework delta: none. Adapted form: none. Cost: none.
I=1 (post lines 1-40 hold no rule). counter: none, at the floor.
R=3 (post lines 1-40; any rule in it would transfer). counter: lower if it assumed a stack we ban.
S=1 (post lines 1-40). counter: none, at the floor.
Total=5/15, skip

chunk 2 post:tip-small-tasks
Pattern: keep each task small. Our equivalent today: notes/working-rules.md:12 ("one task, one commit"). Situational fit: same team size and tools. Our version: unchanged, ours is stricter than the author's 30 minute bound. Framework delta: none, ours is already the better form. Adapted form: none. Cost: none.
I=2 (post lines 90-130 add only a time bound; ours covers the goal). counter: lower if the bound were unmeasurable here.
R=4 (a bound drops in with light edits, notes/working-rules.md:12). counter: lower if no agent reports elapsed time.
S=4 (post lines 112-120 are concrete). counter: lower because the bound needs a clock we do not run.
Total=10/15, skip after the partial rule check below

chunk 4 post:ask-for-a-plan-first
Pattern: ask for a plan before code. Our equivalent today: notes/working-rules.md:20. Situational fit: same. Our version: unchanged, the post adds no step beyond ours. Framework delta: none, ours covers it. Adapted form: none. Cost: none.
I=2 (post lines 222-260 repeat notes/working-rules.md:20). counter: lower if ours already beat the post's review step.
R=4 (lines 230-240 transfer as written). counter: lower if plans were forbidden somewhere.
S=3 (lines 245-260 partly concrete). counter: lower if the template were vague.
Total=9/15, skip

chunk 5 post:model-comparison
Pattern: a table comparing products. Our equivalent today: none (searched notes/: nothing). Situational fit: products we do not run. Our version: no change. Framework delta: none. Adapted form: none. Cost: none.
I=2 (post lines 300-360 give opinions, no method). counter: lower if its numbers are unsourced.
R=3 (lines 310-340 transfer with a moderate rewrite). counter: lower if the table is tied to vendor accounts we will not open.
S=2 (lines 300-360, nothing concrete). counter: lower if no step is stated.
Total=7/15, skip

chunk 6 post:sponsor-section
Pattern: an advertisement. Our equivalent today: none. Situational fit: n/a. Our version: no change. Framework delta: none. Adapted form: none. Cost: none.
I=1 (post lines 400-420, no content). counter: none, at the floor.
R=3 (lines 400-420; any rule in it would transfer). counter: lower if it were tied to a vendor contract.
S=1 (lines 400-420). counter: none, at the floor.
Total=5/15, skip
```

**Partial rule on chunk 2:** the post says "keep each task under 30 minutes of agent work"; ours says "one task, one commit". Differs: the time bound. Not taken: the commit rule is stricter and time is not measurable here. Lookup: `found` at `notes/working-rules.md:12`.

**Take (chunk 3):** `form: additive: no existing rule covers reading the failing output before editing`. Packet approved after both quote-anchors were confirmed in the source. Integration check: the new heading is loaded by the include list at `PROJECT.md:8`. Take review (score 13): the reviewer argued for a skip, cited no uncovered part, and agreed.

**skips.tsv excerpt** (columns: unit, reason_class, resource_anchor, our_anchor, salvaged, plain_reason, destinations_checked):

```
post:intro-story	low-impact	post:1-40	none	none	An anecdote with no rule in it.	all
post:tip-small-tasks	dup	post:90-130	notes/working-rules.md:12	none	Differs: a 30 minute bound. Not taken: ours is stricter.	notes
post:model-comparison	low-impact	post:300-360	none	none	Products we do not use.	all
post:ask-for-a-plan-first	have-equivalent	post:222-260	notes/working-rules.md:20	none	Differs: nothing; adds no step beyond ours.	notes
post:sponsor-section	low-impact	post:400-420	none	none	An advertisement.	all
```

**Decision record:** six blocks, each with `lookups:` and `unknowns:` (chunk 3: `unknowns: none; checked the include list and the notes file`). Summary: `Partial rule: 0 partial, 1 adapted, 5 skipped whole`.

`cost: 6 units scored, 0 trials, 1 take review call, 1 reviewer call, money unknown`

## Example 2: a skill repo with an install script flagged by the scan

**Input:** a repository offering a "browser helper" skill with an installer.

**Intake:** cloned read-only, 38 files, MIT license, last change a few weeks ago. Prior decision lookup: `nothing` (decision log searched).

**Safety scan:**

skill-scan: 2 hard block patterns 1,3,4 (install.sh:14-20, helper/run.py:55-61)

The installer runs a downloaded script piped into a shell (pattern 1, `install.sh:14-20`) and the skill reads an API key from the environment and posts it with a request (patterns 3 and 4, `helper/run.py:55-61`). Both confirmed by reading the lines. Hard block.

**Chunk `browser-helper:run-script`, the security skip.** Tried before skipping: read both hit sites in full, searched for an uninstall step and a way to run without the download (nothing). Named blocker: remote execution plus credential exfiltration. Nothing from the installer is run, trialed or copied.

```
Pattern: drive a browser from a script the agent writes.
Our equivalent today: none (searched notes/testing.md and PROJECT.md: nothing)
Situational fit: the author uses a throwaway machine; we run beside real credentials.
Our version: none, no safe form exists while the installer and key upload stay.
Framework delta: none, no file of ours would change.
Adapted form: none.
Cost: none.
I=3, would help UI tests (helper/run.py lines 10-40). counter: one point lower if our tests are mostly API tests.
R=1, blocker named above (install.sh:14-20, helper/run.py:55-61). counter: none, at the floor.
S=1, the floor; a hard block forces a skip whatever the score (install.sh:14-20). counter: none, at the floor.
Total=5/15, skip (security)
```

**Chunk `browser-helper:selector-heuristics`**, a plain text document with no code (`docs/selectors.md`, lines 1-90), passes the scan and scores as a partial:

```
Pattern: prefer stable selectors (roles and labels) over generated class names.
Our equivalent today: none (searched notes/testing.md and PROJECT.md for "selector": nothing)
Situational fit: the author tests one app; we test several, so stability matters more.
Our version: keep only the stable-selector paragraph; drop the author's tool-specific commands.
Framework delta: none, notes/testing.md has no selector rule.
Adapted form: three lines added to notes/testing.md under "Selectors", our wording.
Cost: one file touched, one consumer (the include list at PROJECT.md:8).
I=4, flaky tests are our main test cost (docs/selectors.md lines 12-30); target notes/testing.md. counter: one point lower if our flaky tests come from timing, not selectors.
R=3, the paragraph transfers but the tool-specific commands must be cut (docs/selectors.md lines 31-60). counter: one point lower if the paragraph leaned on the tool's commands.
S=4, three concrete rules (docs/selectors.md lines 12-30). counter: one point lower because the examples assume a web page.
Total=11/15, partial
```

Only the stable-selector paragraph is kept. Skip row for the security skip:

```
browser-helper:run-script	security	install.sh:14-20,helper/run.py:55-61	none	selector-heuristics kept (docs only)	Remote execution and a key upload; no safe form.	all
```

Skip review sampled every `security` skip; the reviewer opened both cited files and agreed. The record quotes the two hit lines; only the user may override.

`lookups:` `nothing` for prior decisions; `found` at the two cited files. `unknowns:` whether a newer release removed the installer (`could-not-look`: only the cloned commit was read).

`cost: 2 units scored, 0 trials, 1 reviewer call`

## Example 3: a duplicate of an existing practice, skipped

**Input:** a short article, "write a test before you fix a bug".

**Intake:** whole-file mode (under 200 lines, one idea). Fetched in full. Prior decision lookup: `nothing` (searched the decision log).

**Safety scan:**

skill-scan: 0 clean (no code)

**Sketch and scores:**

```
Pattern: reproduce the bug with a failing test, then fix it.
Our equivalent today: PROJECT.md:20 ("every bug fix starts with a failing test")
Situational fit: same stack and team size as the author.
Our version: identical to the author's form, since the article's rule and our line say the same thing; nothing to modify.
Framework delta: none, PROJECT.md:20 already states the rule and no other file of ours covers it.
Adapted form: none for the main rule; one added clause in PROJECT.md:20 after the partial rule check below.
Cost: one line edited, no new consumers.
I=1, the article adds nothing over PROJECT.md:20 (article lines 5-60). counter: none, at the floor.
R=5, the article's rule transfers unchanged (article lines 5-30). counter: one point lower if our test setup could not run a failing test first.
S=4, steps are concrete (article lines 31-60). counter: one point lower if "failing test" were undefined.
Total=10/15, below 11 and Impact under 3: skip
```

**Partial rule check:** the reviewer reads the article against `PROJECT.md:20` and finds one differing paragraph (article lines 70-80): "keep the failing test as a regression test afterward". Ours does not say that. The scorer converts the skip to a partial, form `revise_ours PROJECT.md "Debugging": add one clause, keep the failing test afterward`. Nothing else is taken.

```
article:main-rule	have-equivalent	article:5-60	PROJECT.md:20	regression clause taken as a partial	Differs: only the regression clause; the rest repeats ours.	all
```

Decision record verdict: `partial`; `lookups: found PROJECT.md:20; nothing in the decision log`. `unknowns: none; checked PROJECT.md and notes/working-rules.md for a regression clause`.

`cost: 1 unit scored, 0 trials, 1 reviewer call`
