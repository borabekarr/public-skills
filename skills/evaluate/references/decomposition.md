# Decomposition

Split a resource into named chunks before scoring so one weak section never hides a strong one. Never skip a whole source, and never score a repo or a long article as one lump.

## Mode

- **Whole-file:** short (about 200 lines or less) and about one idea. Fewer than three units also means whole-file.
- **Part-level:** longer, with distinct sections, or a bundle of sub-tools and patterns. Split to the finest named unit (a single skill, pattern or rule), never to plugin, folder or chapter level.

## Chunk types

| Type | Example |
|---|---|
| `approach` | A named strategy, such as "option type for errors" |
| `step` | One discrete action in a multi-step process |
| `pattern` | A code or workflow pattern, such as debounced autosave |
| `rule` | A single constraint or guideline |
| `snippet` | A code block that works alone |
| `section` | A heading-delimited block when nothing smaller fits |
| `tool-entry` | One entry in a tools or skills list |
| `whole-resource` | One extra unit naming what the resource is as a whole (`tool`, `skill-pack`, `method`, `article`, `dataset`); score what adopting it as a unit would be worth |

## Splitting heuristics

- Respect headings first: each second or third level heading is a candidate boundary.
- Respect numbered lists: each top-level step is a candidate chunk.
- Keep a code block with the prose that explains it; never split mid-pattern.
- If two adjacent sections argue the same point, merge them.
- Typical size is 5 to 80 lines. Larger: split further. Smaller than one idea: merge with a neighbor.
- No cap on the number of chunks. Every unit is read in full and gets its own row.

## Naming

Each chunk gets a slug `resource-slug:chunk-slug`, for example `style-guide:error-messages`. The slug is used in the score table, the decision record and any attribution line.

## Inventory first

Before scoring, list every unit, one per line (a plain file listing is enough for a repo). Then keep a coverage table: unit, size, how it was read (`full`, `head`, `description-only`), verdict (`scored`, `skipped`, `unread`). A unit can be scored or skipped only when it was read in full. Anything else is `unread` and is reported as not read.

## Independence

Each chunk is scored and decided on its own. One run can produce three different forms: one chunk replaces a file, another merges into a section, a third is an addition. If one chunk passes but has no form, only that chunk fails; its siblings are unaffected.
