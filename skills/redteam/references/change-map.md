# The change map

The change map is the scope of a redteam run. Nothing outside it (and its capped reader lists) is reviewed. Build it by hand with plain commands; no script is needed.

## 1. List the changed files

Run, from the project root, with `<base>` the branch or commit the change started from:

- `git diff --name-only <base>...HEAD`
- `git ls-files --others --exclude-standard` (untracked files; add them)
- Uncommitted edits: `git diff --name-only` and `git diff --name-only --cached`

Merge the lists, remove duplicates. If the project is not a git repository, or no base is known, ask the owner for the file list; a list the owner gives is a valid map. With neither a git base nor an owner list, there is no map: report `could-not-look: no change map` and stop. Never guess changed files from memory.

## 2. Give each file a kind

| Kind | Examples |
|---|---|
| code | source files, scripts |
| config | settings, build and CI files, environment templates |
| prompt/doc | agent prompts, skill files, workflow text, README and guides |
| test | test files and fixtures |
| schema | database migrations, API schemas, data formats, shared types |

## 3. Find the readers of each file

A reader (consumer) is any file that names the changed one: imports it, calls it, links to it, loads it by path, or quotes its name. Find them by searching the repo for the file's path and for its base name, and for the main symbols it defines when it is code. Exclude the file itself and the map's own work directory.

- Cap: at most 25 readers per file. If more exist, keep the first 25 and add a note `CAP <total>` (for example `CAP 61`) to that file's row, and mention it in the final report.
- Depth 1 (default): the files that name the changed file.
- Depth 2: for schema, shared config and shared prompt or doc files, also find the readers of those readers, again capped at 25 per file (the cap applies to the depth 2 list as a whole).

A changed file with zero readers is not an error: write `readers: none` and the final report lists it as unwired (a unit nothing uses, or a reader the search missed; the judge decides which).

## 4. Save the map

Write a table to `{{WORK_DIR}}/redteam/change-map.md`:

| file | kind | depth | readers |
|---|---|---|---|
| skills/export.md | prompt/doc | 1 | run.md, skills/index.md |
| src/limits.py | code | 1 | src/api.py, tests/test_limits.py |
| db/schema.sql | schema | 2 | src/db.py, src/api.py, tests/test_db.py CAP 31 |

## Later rounds

Round n+1 reads the map of round n and adds every file the round n fix changed (take them from the fix plan's file list, then confirm with `git diff --name-only`). Give each added file a kind and readers as above. Files from earlier rounds stay in the map.

## Defects

- A changed file with no row in the map is a map defect: add the row. Do not widen the scan to the whole repo.
- A reader list that was cut by the cap is reported with its `CAP` note, never presented as complete.
