---
name: route-work
description: Sizes and routes coding work in a repository: a request to build, change, fix or plan something, or an issue reference ("fix #12"), unless the user named a planning skill. Use it even when the request sounds small, because sizing is this skill's job, and again at a phase boundary when the work proves bigger or smaller. Not for questions, code review, deploys or reading data.
---

# Route work

Size the request, take the **lightest route** that leaves no decision silently assumed, announce it, and go. Every route ends in **a verified, reviewed, merged PR**; when an issue exists, the PR closes it with `Closes #N`. This skill is the single entry point to the `mattpocock-skills` plugin, so the user never has to name its skills; when they do, follow theirs.

## Before the first step

1. **The project's rules win.** Read the repo's `CLAUDE.md` (or `AGENTS.md`): its commands, branch/worktree workflow, commit style, PR template and merge rule override anything here.
2. **Setup** is done when `docs/agents/issue-tracker.md` exists; then the tracker, labels and domain-doc layout are whatever `docs/agents/` says. When it is missing, read [`SETUP.md`](SETUP.md).
3. **Plugin skills.** Skills marked *read* set `disable-model-invocation`, so the Skill tool refuses them: find their `SKILL.md` as [`SETUP.md`](SETUP.md#finding-a-read-skill) describes and follow it exactly. Call the rest with the Skill tool as `mattpocock-skills:<name>`.
4. **Issues.** When the user names an issue (`#12`, a URL), or the route will create or close one, read [`ISSUES.md`](ISSUES.md).

## Size it

Look before judging: an `Explore` sub-agent or a quick grep shows which modules, pages and `CLAUDE.md` rules the request touches. Check every row and take the **largest** size whose signals apply; between two, take the larger and let re-routing step it down. Sizing is done when you can name the signal that decided it.

| Size | Signals | Plan | Issue | Review |
|---|---|---|---|---|
| **Trivial** | The request decides everything: a typo, dictated text, an obvious one-line fix. | None. | None. | Self-check. |
| **Small** | One session, a few open decisions (behaviour, wording, defaults), no change to schema, external/model API calls, auth/security, money/limits or public interfaces. | `grilling`, or `grill-with-docs` (*read*) when a decision deserves an ADR or glossary entry. | None; decisions go in the PR body. | Self-check. |
| **Medium** | One session's build, but it touches schema, an external/model call or prompt, security, cost/limits, a public interface, or several screens. A reviewer will need the decisions written down. | `grill-with-docs` → `to-spec` (*read*). | `to-spec` publishes the spec as one issue; the PR closes it. | `code-review`. |
| **Large** | More than one context window, or parts that can land independently. | `grill-with-docs` → `to-spec` → `to-tickets` (*read*) → run the ticket graph ([`LARGE.md`](LARGE.md)). | Spec issue = parent; one sub-issue per ticket with native `blocked by` links; one PR per ticket closes it. | `code-review` per ticket PR; `retro` offered at the end. |
| **Fog** | The destination is unclear, or decisions depend on each other too deeply to settle in one sitting. | `wayfinder` (*read*) → then `to-spec` → `to-tickets` → build. | A `wayfinder:map` issue with decision tickets as children ([`ISSUES.md`](ISSUES.md#fog)). | As Large. |

**Bugs** follow the same table, sized by the fix. When the cause is not obvious, the first phase is `diagnosing-bugs`, and it **stops once a minimal reproduction confirms the cause**, before its fix phase. Then size the fix, announce it, and fix on a branch with that reproduction as the failing test.

## Toolbox: call these when the situation appears, at any size

| Situation | Skill |
|---|---|
| A cause is unknown: something broken, throwing, flaky or slow | `diagnosing-bugs` |
| Grilling stalls on "would this design / state model / UI even work?" | `prototype` (throwaway; its answer goes into the spec, its code is deleted) |
| A decision needs facts from docs, an API or a library | `research` (writes its findings to a file in the repo) |
| Choosing where a module boundary or test seam goes | `codebase-design` |
| A decision you can't make without someone else | `to-questionnaire` (*read*) |
| A step only the human can do (credentials, CI secrets, provisioning) | `wizard` |
| Writing or editing `CLAUDE.md`, `AGENTS.md` or a skill | `writing-for-agents` |
| Context is filling up mid-task | `handoff` (*read*), then continue in a fresh session |
| The user says the last message didn't land | `wait-what` (*read*) |

Every route builds through a branch and a PR, so it uses only the skills above. That leaves out `implement` / `implement-spec` (they commit to the current branch with no PR or `Closes`), `grill-me` (`grilling` replaces it), the plugin's `in-progress/` skills, and language-specific skills that don't match the repo (`setup-ts-deep-modules`, `migrate-to-shoehorn` and `setup-pre-commit` are TypeScript/JS only).

## Announce, then go

One line: the size, the signal that decided it, the setup state, and the **next phase only**. Each later phase is announced when it starts, so the one in front gets the full attention.

> Size: **Medium** (it changes the interviewer prompt; setup: done). Next: grill-with-docs. Say so if you want a different size.

For a bug whose cause is unknown, the size waits for the cause:

> Size: after diagnosis (cause unknown; setup: done). Next: diagnosing-bugs, stopping at a confirmed reproduction.

Then go. Every routed skill has its own human gate (grilling waits for a shared understanding, `to-spec` checks seams, `to-tickets` has the breakdown approved, `wayfinder` resolves one ticket per session, setup confirms each section), and that approval also covers publishing the issues and files it produces.

## At every phase boundary

Re-check the size, then announce the next phase in the same one-line form, naming any size change:

- Grilling surfaced schema, external-call, security or cost decisions → **Medium** (and run the setup now if it is missing).
- The spec grew independently landable parts or more than one context window of work → **Large**, run `to-tickets`.
- Each answer opens more unknowns than it closes → **Fog**, carry what is settled into the map.
- `wayfinder` finds no fog → **Medium** or **Large**.
- Grilling settles in one round and nothing needs a record → finish as **Small**.

Keep grilling → spec → tickets in **one unbroken context**.

## Build

The build is done by this session, by `Agent` with `subagent_type: "general-purpose"`, or by a `Workflow` when the user has opted into multi-agent orchestration. From **Medium** up, build with `tdd` at the seams `to-spec` already agreed with the user, so `tdd`'s own seam question is already answered. For **Trivial / Small**, the `tdd` skill is more process than the change needs: write the test first in the project's usual style when behaviour changes (a bug, a rule, a calculation); text or style changes need no new test.

- **Trivial / Small / Medium**: build here, or hand the spec issue to one general-purpose agent. A brief for an agent names the exact input, the expected result and the acceptance criteria.
- **Large**: one fresh context, branch and PR per ticket ([`LARGE.md`](LARGE.md)).

Each change: its own branch and worktree (the project's worktree rule if it has one; otherwise `Agent` with `isolation: "worktree"`) → tests first → the project's lint and test commands, reading the output → review → a PR.

**Verification belongs to the dispatching session, not the agent:** re-run lint and tests yourself, `git diff --check`, run the app for a UI change, and scan the diff for secrets and personal data. When the acceptance criteria need tests CI skips (live API, slow, manual), run them yourself and record the result in the PR's Test plan and a comment on the issue: green CI proves only what CI runs.

**Review (Medium and up):** run `code-review` against the default branch (`origin/main`) after verification and before merging. Its **Spec** axis checks the diff against the issue, its **Standards** axis against the repo's documented rules. It only finds `CODING_STANDARDS.md` / `CONTRIBUTING.md` by itself, so when the repo has neither, pass `CLAUDE.md` (or `AGENTS.md`) as the standards source. Fix every missing or wrong requirement; fix or consciously decline (one line in the PR) each standards finding. Re-run verification after fixes.

**PR body**: the project's template, or the `pr` skill's shape, or **Summary** + **Test plan**; then one `Closes #N` line per issue it finishes, each on its own line.

**Merge** only as the project's `CLAUDE.md` allows (e.g. "squash-merge when CI is green"). When it says nothing, stop at an open PR and ask the user to review.

## Finish

- **Report**: what merged, which issues closed, the follow-up list, and the setup reminder if setup was missing.
- **Retro**: after a Large or Fog route, or any route where the agent repeated a mistake, a check failed late, or a file was hard to find, offer `retro` (*read*) in one line. It proposes changes to the environment (`CLAUDE.md`, automated checks, coding standards); apply them only with the user's yes.

## Things found along the way

A bug or idea outside the current issue goes on the **follow-up list**, not into this PR. The dispatching session keeps one running list (from exploration, grilling, `code-review` and every agent's report) and puts it under **Follow-ups** in the next PR it opens, or in its final report if no PR follows. When the user says yes to tracking them, open **one** issue for the whole batch, labelled `follow-up` ([`ISSUES.md`](ISSUES.md)).
