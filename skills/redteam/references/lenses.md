# Lenses and gap classes

The judge applies the checklist of the chosen lens to the files in the change map and their capped readers. A checklist item is a question to ask of the change, never a reason to widen the scope past the map. Every finding carries one gap class from the lens list below (the class is part of the ledger key, so use the exact spelling).

If a finding fits no class, use the closest one and say why in the claim; do not invent a new class mid-run.

## code lens (source, tests, schemas)

| Class | What to look for |
|---|---|
| caller-break | A changed signature, return shape or default that a reader in the map still uses the old way. |
| error-path | A failure (timeout, empty result, exception) with no handling, or a handler that swallows it. |
| boundary-input | Empty, null, zero, huge or malformed input the change does not guard. |
| concurrency | Two callers or processes touching the same state, file or row with no ordering. |
| resource-leak | A handle, connection, temp file or lock opened and not closed on every path. |
| data-compat | Stored data, saved files or messages written by the old version that the new code cannot read. |
| test-gap | A changed behavior with no test, or a test that passes by checking nothing. |
| config-default | A new setting whose default is wrong for existing users, or a setting read but never defined. |

## system lens (agent prompts, skills, workflows, configs)

| Class | What to look for |
|---|---|
| unwired-unit | A new or changed prompt, skill, step or config that nothing reads or triggers. |
| stale-reference | A rule, doc or prompt that still describes the old behavior, or points at a file or name that moved. |
| single-writer | A file, marker or log with an "only X writes it" rule that a second caller can break. |
| foreign-state | A step that reads another run's marker, scratch file or skip flag as its own. |
| unresolved-path | A path the step reads that does not resolve when it runs (a different directory, user, or a moved link). |
| guard-bypass | A guard or allow-list whose operand is built from a variable or an environment override, so it cannot see the real target. |
| shared-scratch | Concurrent runs writing one fixed scratch path, so one run's file breaks another. |
| test-drift | A test or check that still asserts the old contract, or that cannot fail. |
| weak-check | A claim closed by a check weaker than the claim (a presence grep closing a behavior claim). |

Also ask of every changed system file: who reads it (the consumer list), does a rule or shared reference it changed still match its callers, and does the step's check have a strength that matches its claim (`references/rea-method.md`).

## product lens (user-facing behavior)

| Class | What to look for |
|---|---|
| state-missing | A loading, empty, error or offline state the change leaves undefined. |
| journey-break | A route a user takes through the change that no longer works end to end. |
| consumer-mismatch | A screen, component or client using a changed field, prop or endpoint the old way. |
| data-mismatch | Counts, totals or rows that no longer reconcile after the change; a value written and never read. |
| access-gap | A change that shows or lets a user do something their role should not (permissions, visibility rules). |
| copy-drift | Labels, messages or help text that no longer match what the feature does. |
| layout-break | Responsive, motion or accessibility breakage on the screens the change touches or feeds. |

## Choosing a lens

- Mostly source and tests: `code`.
- Mostly prompts, skills, workflow text, configs, automation: `system`.
- A change a user sees or feels: `product`.
- A mixed change: run the lens that covers most map files first; start a second pass with the other lens only if the owner asks, with its own ledger rows (the lens is part of the key).
