# agent-skills

My skills for Claude Code (and other agents that read `SKILL.md` files).

| Skill | What it is for |
|---|---|
| [`route-work`](skills/route-work/SKILL.md) | The one entry point for coding work. Say what you want ("fix #12", "add X"); it sizes the request (Trivial → Small → Medium → Large → Fog) and runs only the [Matt Pocock skills](https://github.com/mattpocock/skills) that size needs: grilling, spec, tickets, tdd, code-review, retro. Tiny and small work makes no issues; bigger work gets one issue per piece, closed by its PR (`Closes #N`). |

## Install

`route-work` drives Matt Pocock's skills, so install those too.

**As Claude Code plugins** (recommended):

```bash
claude plugin marketplace add mattpocock/skills
claude plugin install mattpocock-skills@mattpocock
claude plugin marketplace add JinhoKim46/agent-skills
claude plugin install agent-skills@jinho
```

Update later with `claude plugin update agent-skills@jinho`. With several Claude config directories, run the commands once per directory with `CLAUDE_CONFIG_DIR=<dir>` set.

**With the `skills` CLI** (works for other agents too):

```bash
npx skills add JinhoKim46/agent-skills -g
```

Start a new Claude Code session afterwards; skills load at session start.

## Use

Just describe the work. In each repo, the first Medium-or-bigger task runs Matt's one-time setup (`docs/agents/issue-tracker.md`), which asks a few questions about where issues live.

## Maintaining (owner)

On the owner's machine, `~/.agents/skills/route-work` is a symlink into this repo, so edits here are live in every session. Commit and push after each change. `backup/skill-lock.json` is a copy of `~/.agents/.skill-lock.json`, the list of third-party skills installed with `npx skills`, for reinstalling on a new machine.
