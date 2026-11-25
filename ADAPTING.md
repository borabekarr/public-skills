# Adapting these skills

One file holds every placeholder and every per-harness value, so the skill READMEs stay short and nothing drifts. Harness paths change fast: each path below carries its docs link and the date it was checked. If a link and this file disagree, trust the link.

## Placeholders

Grep `skills/` for `{{` to confirm this table is complete. Replace each one once, in your installed copy, never in a way that changes the method steps.

| Placeholder | Meaning | Default if you have no preference |
|---|---|---|
| `{{PROJECT_ROOT}}` | The repo or folder being reviewed or protected. | The current working folder. |
| `{{WORK_DIR}}` | Where notes, ledgers, change maps and logs are written. | `.skill-work/` in the project root (add it to your ignore file). |
| `{{TARGET_SYSTEM}}` | What the evaluate skill scores resources against: the project, agent setup or workflow you are improving. | Your project's agent instructions file plus its skills. |
| `{{SUBAGENT}}` | How to start a fresh helper with no memory of the main session. | A separate fresh pass (see Helpers). |
| `{{STRONG_MODEL}}` | The model for seats that decide (the redteam judge, quality trials). | Your most capable available model. |
| `{{FAST_MODEL}}` | The model for seats that only gather (the redteam reader) or where speed matters. | Your fastest cheap model, or the same model as above. |

Write model names the way your harness expects them. This repo names no specific model on purpose.

## Helpers

If your harness has subagents, use them for the reader or judge role; otherwise run the role as a separate fresh pass that starts without your drafting context. The method needs the seats to be different readers, not a particular API.

## Frontmatter

Keep `name` and `description` in each SKILL.md. Other frontmatter keys may be rejected outside the harness that defines them. The `name` must match the folder name.

## Claude Code

- Install path: `~/.claude/skills/<name>/SKILL.md` for all your projects, or `.claude/skills/<name>/SKILL.md` inside one repo.
- Docs: https://code.claude.com/docs/en/skills
- Status: verified 2026-11.
- Helpers: Claude Code can start subagents; set `{{SUBAGENT}}` to "start a subagent".
- Self-test: ask your agent to list your skills and check the new name appears.

## Codex CLI

- Install path: `.agents/skills/<name>/` inside a repo, or `$HOME/.agents/skills/<name>/` for all projects. `name` and `description` are required.
- Docs: https://developers.openai.com/codex/skills
- Status: verified 2026-11.
- Self-test: ask your agent to list your skills and check the new name appears.

## Gemini CLI

- Install path: `.gemini/skills/<name>/` or `.agents/skills/<name>/` in a workspace; `~/.gemini/skills/<name>/` or `~/.agents/skills/<name>/` for all projects. Docs: https://geminicli.com/docs/cli/skills/
- Status: verified 2026-11. The neutral target that Codex and Gemini both read is the `.agents/skills/` folder (https://developers.openai.com/codex/skills).
- Self-test: run `gemini skills list`, or ask your agent to list your skills.

## Grok

- Install path: unverified. The Grok Build CLI says AGENTS.md and skills work, but its skills folder was not confirmed. Check https://x.ai/news/grok-build-cli and your CLI's own help for the current folder.
- Status: unverified, check https://x.ai/news/grok-build-cli
- Fallback that does not need a skills folder: paste the SKILL.md text into your AGENTS.md (or the system prompt) and keep the skill folder in your project so the `references/` files can be read.
- Self-test: ask your agent to list your skills. If the name is missing, use the fallback and ask it to quote the first line of the skill to prove it was loaded.

## Any other agent

- Paste the SKILL.md text into the system prompt or the agent's AGENTS.md style instruction file, and keep the folder in the project so relative links to `references/` and `roles/` resolve. If the agent reads a skills folder, `.agents/skills/` is the likeliest neutral choice (see https://geminicli.com/docs/cli/skills/), but that is unverified for your tool: check its own docs.
- Self-test: ask your agent to list your skills, or to quote the skill's description line. If it cannot, the skill is not loaded.

## Prompt for your agent

Paste this, with the skill name filled in:

```text
Read skills/<name>/README.md and skills/<name>/SKILL.md, then install this skill for my harness.

Placeholders to resolve, and nothing else:
- {{PROJECT_ROOT}}: the repo I am working in
- {{WORK_DIR}}: a notes folder (default .skill-work/ in the project root)
- {{TARGET_SYSTEM}}: what I am improving (ask me if unclear)
- {{SUBAGENT}}: how to start a fresh helper in this harness, or "a separate fresh pass" if it has none
- {{STRONG_MODEL}}: the strongest model available to me
- {{FAST_MODEL}}: a fast cheap model, or the same one

Rules:
1. Keep every method step in the skill. Change only paths, helper spawning and model names.
2. Keep SKILL.md frontmatter to name and description only.
3. Copy the folder into my harness skills directory per ADAPTING.md. Never delete or overwrite an existing skill of the same name; if the name is taken, stop and ask me.
4. Do not install anything from the network.
5. When done, list your skills and show me the new one, and list every line you changed.
```
