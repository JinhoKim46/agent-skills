# agent-skills

Agent skills for Claude Code and other agents that read `SKILL.md` files. The main skill, **route-work**, is a single entry point for coding work: describe what you want, and it sizes the request and runs only the planning, building and review steps that size needs, ending in a verified, reviewed pull request.

| Skill | Purpose |
|---|---|
| [`route-work`](skills/route-work/SKILL.md) | Sizes a coding request (Trivial → Small → Medium → Large → Fog) and drives [Matt Pocock's skills](https://github.com/mattpocock/skills) (grilling, spec, tickets, tdd, code-review, retro) through GitHub issues and PRs. |
| [`code-flow-report`](https://github.com/JinhoKim46/code-flow-report) | Builds one offline HTML page that traces how a Python codebase works, call by call, re-checked against the source on every build. Maintained in its own repository; listed here in the marketplace. |

## Why route-work

Skill collections are powerful, but they shift work onto you: you have to remember which skill exists, when each one applies and in what order to chain them. Running the full chain on every request is slow, and skipping it on a change that needed it leaves decisions silently assumed.

route-work makes that choice for you. It looks at the code a request touches, picks the lightest route that still writes every decision down, tells you in one line, and goes. A typo fix becomes one PR; a schema or prompt change gets a grilling session, a spec issue and a code review; a multi-part feature becomes a graph of tickets built in parallel.

## Requirements

- [Claude Code](https://claude.com/claude-code), or another agent that reads `SKILL.md` files
- The [`mattpocock-skills`](https://github.com/mattpocock/skills) plugin, which route-work drives
- A Git repository; for issue tracking, the [GitHub CLI](https://cli.github.com/) signed in (`gh auth status`). Without GitHub, issues are kept as local Markdown files.

## Installation

### As Claude Code plugins (recommended)

```bash
claude plugin marketplace add mattpocock/skills
claude plugin install mattpocock-skills@mattpocock
claude plugin marketplace add JinhoKim46/agent-skills
claude plugin install agent-skills@jinho
claude plugin install code-flow-report@jinho   # optional
```

Update with `claude plugin update agent-skills@jinho`. If you use several Claude config directories, run the commands once per directory with `CLAUDE_CONFIG_DIR=<dir>` set.

### With the `skills` CLI (any agent)

```bash
npx skills add JinhoKim46/agent-skills -g
```

Skills load at session start, so open a new session after installing or updating.

## Usage

Describe the work in plain words, or point at an issue (`fix #12`). You don't need to name a skill:

```text
> Make the interviewer ask exactly one follow-up question after each answer.

Size: Medium (it changes the interviewer prompts and the engine's follow-up rule; setup: done).
Next: grill-with-docs. Say so if you want a different size.
```

The announcement names the size, the signal that decided it, and only the next phase; each later phase is announced when it starts. Reply with a different size at any point to override it.

The first Medium-or-larger task in a repository runs Matt Pocock's one-time setup, which asks where issues live and writes `docs/agents/`. Smaller tasks never wait for it.

## How it works

| Size | When | Route | Issue |
|---|---|---|---|
| **Trivial** | The request decides everything (a typo, dictated text) | Build | None |
| **Small** | A few open decisions; no schema, external calls, security, cost or public interfaces | Grill → build | None; decisions go in the PR |
| **Medium** | Touches schema, a model/API call or prompt, security, cost, a public interface or several screens | Grill → spec → build with TDD → code review | One spec issue, closed by the PR |
| **Large** | More than one context window, or parts that can land independently | Grill → spec → tickets → parallel builds | A parent issue with one sub-issue and PR per ticket |
| **Fog** | The destination itself is unclear | Wayfinder map → spec → tickets | A map issue with decision tickets |

Bugs are sized by their fix. When the cause is unknown, diagnosis comes first and stops at a confirmed reproduction, which becomes the failing test. At every phase boundary the size is re-checked and may step up or down.

Every route ends in a verified, reviewed PR. A project's own `CLAUDE.md` or `AGENTS.md` (commands, branch workflow, PR template, merge rule) always takes precedence over the skill.

### Layout

```text
skills/route-work/
├── SKILL.md    # loaded every run: sizing, toolbox, announcement, build and review
├── ISSUES.md   # read when a route names, creates or closes an issue
├── LARGE.md    # read for Large routes: the parallel ticket graph
└── SETUP.md    # read when the repo's setup is missing, or a read-only skill must be found
```

### Design

The skill follows the checklist in Matt Pocock's [`writing-for-agents`](https://github.com/mattpocock/skills/tree/main/skills/productivity/writing-for-agents) skill, presented in his talk [Building Great Agent Skills](https://www.youtube.com/watch?v=UNzCG3lw6O0):

- **Trigger:** model-invoked, so you never have to remember it; the description stays short because it sits in context on every turn.
- **Structure:** `SKILL.md` holds only what every route needs; material for some branches (issues, Large routes, setup) lives in files it points to.
- **Steering:** rules state the behaviour to produce rather than the one to avoid, and the announcement shows only the next phase so later phases don't pull the current one short.
- **Pruning:** each rule lives in one place; restatements and instructions the model already follows by default are removed.

## Customising per project

Put project-specific routing in the repository's `CLAUDE.md` (or a document it links), not in a project-level skill called `route-work`: Claude Code loads a personal skill over a project skill of the same name, so a project copy would be silently ignored. A project that needs a different router should give it another name and say so in its `CLAUDE.md`.

## Development

Change the skill on a branch and open a pull request. To check a change, run a fixed set of requests (a typo, a wording change, a prompt change, a vague bug, a multi-part feature) against the old and the new `SKILL.md` in a real repository, and compare the announcement lines:

```bash
claude -p --permission-mode plan "Read <path>/skills/route-work/SKILL.md and follow it for this request only up to its announcement line, then stop. Request: <request>"
```

With each release, bump `version` in `.claude-plugin/plugin.json` and add an entry to [`CHANGELOG.md`](CHANGELOG.md).

## License

[MIT](LICENSE)
