# public-skills

Four agent skills for reviewing changes, vetting outside resources, debugging hard bugs and checking claims before they ship. Each skill is a plain folder of Markdown (plus one shell template) that any coding agent can read and follow. Nothing here depends on one vendor: you give a skill folder to your agent, it adapts a handful of placeholders and paths to your harness, and installs it by copying.

## The skills

| Skill | What it does | Use it when | Folder |
|---|---|---|---|
| redteam | Reviews one change and what depends on it: change map, reader pass, separate judge pass, a ledger and a 3-round cap on the fix loop. Not a security scan. | After you build or change a workflow, system or code, to hunt gaps and weak spots. | [skills/redteam](skills/redteam) |
| evaluate | Decides chunk by chunk whether an outside resource (article, repo, skill, prompt pack) is worth adopting: safety scan, IRS scoring, take or skip verdicts, a proposal for approval, a landing check. | Before you adopt anything from outside into your own setup. | [skills/evaluate](skills/evaluate) |
| diagnose | Builds a fast pass/fail loop first, then reproduces, ranks hypotheses, instruments one variable at a time, fixes and adds a regression test. | A bug is hard, flaky or intermittent, or "works on my machine". | [skills/diagnose](skills/diagnose) |
| verify-before-claim | Requires an 11-row evidence card for every "cut", "merge", "unused" or "duplicate" claim, checked by an independent reviewer. | Before you ship any claim that something is dead, redundant or safe to remove. | [skills/verify-before-claim](skills/verify-before-claim) |

## Quick start

Give your agent this repo URL, or one skill folder plus `ADAPTING.md` (the skill folders point to it), and say:

> Read skills/<name>/README.md, adapt it per ADAPTING.md, install it.

Replace `<name>` with one of the four folder names. The agent reads the skill, fills the placeholders listed in [ADAPTING.md](ADAPTING.md), and copies the folder into your harness's skills directory. Then ask the agent to list your skills to confirm the new one shows up.

## Install

Copy or symlink one skill folder into the skills directory your harness reads. The exact path per harness is in [ADAPTING.md](ADAPTING.md). Never delete or overwrite an existing skill of the same name: if the name is taken, stop and pick a different folder name or ask the owner.

## Adapt to your model and harness

The method steps in each skill are the same everywhere. Only a few things change per setup: paths, how helpers are started, and which model fills which seat. Claude Code, Codex CLI and Gemini CLI each read skills from their own folder (documented in ADAPTING.md with the docs link). Grok is covered too, but its skills folder is unverified, so ADAPTING.md gives a fallback and a self-test. Any other agent can be given the SKILL.md text in its system prompt or AGENTS.md. Everything is in [ADAPTING.md](ADAPTING.md), including a paste-ready prompt you can hand to your agent.

## Safety

- Skills read files in your project and may run `git` and `grep`. Read a skill before installing it, as you would any script.
- The evaluate skill reads outside content you point it at and scans that content for injected instructions, remote execution and credential theft before using any of it.
- Nothing in this repo installs from the network: installs are a local copy. The one shell file is a template that you read and run yourself.
- To report a vulnerability or an accidentally published secret, see [SECURITY.md](SECURITY.md).

## License and credits

MIT, see [LICENSE](LICENSE). Some skills build on open source work from other authors; every upstream and its license is credited in [NOTICE.md](NOTICE.md).
