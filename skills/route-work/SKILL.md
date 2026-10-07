---
name: route-work
description: Use when the user asks to build, add, change, fix, debug, redesign, migrate or plan something in a code repository, or points at an issue ("fix #12", "work on issue 12"), and has not named a planning skill themselves, even when the request sounds small, because deciding that it is small is this skill's job. Also use at a phase boundary when the work turns out bigger or smaller than first judged. Not for pure questions, code review, deploys or reading data.
---

# Route work

Size the request, pick the lightest route that leaves no decision silently assumed, say which one in one line, and go. Every piece of work ends in **a verified, reviewed, merged PR; when an issue exists, the PR names it with `Closes #N` so the issue closes itself.** This skill is the single entry point to the `mattpocock-skills` plugin: the user should never have to remember or name its skills (`/to-spec`, `/tdd`, `/code-review`, ...); if they do name one, follow theirs.

## Before the first step

1. **The project's rules win.** Read the repo's `CLAUDE.md` (or `AGENTS.md`): its commands, branch/worktree workflow, commit style, PR template and merge rule override anything below. Put project-specific routing there (or in a doc it links), not in a project-level `route-work` skill: Claude Code loads a personal skill over a project skill of the same name, so a project copy is silently ignored. A project that needs a different router gives it another name and says so in its `CLAUDE.md`.
2. **Check the Matt Pocock setup.** It is done when `docs/agents/issue-tracker.md` exists; that file is the only test. Report it in the announcement line (`setup: done` / `setup: missing`).
   - **Done** → the tracker, labels and domain-doc layout are whatever those files say. Use them; don't guess.
   - **Missing, and the route is Trivial or Small** → don't block. Build, and add one line to the final report: "This repo has no `docs/agents/` setup; run it before the next Medium+ task (I can do it)."
   - **Missing, and the route is Medium, Large or Fog, or the user named an issue** → run `setup-matt-pocock-skills` (*read*) first. It asks its own questions and writes `docs/agents/*` plus the `CLAUDE.md` block; land those as their own small PR (or in the work's first PR if the project allows), then continue. If the user declines, use GitHub Issues when `git remote -v` points at GitHub and `gh auth status` succeeds, else local Markdown under `.scratch/`, and tell every routed skill which one it is.
3. **Labels (GitHub):** before the first `gh issue create`, check `gh label list` and create any missing label this route will apply (`ready-for-agent`, `follow-up`, `wayfinder:*`, or the names in `docs/agents/triage-labels.md`).
4. **Run a plugin skill.** Skills marked *read* below set `disable-model-invocation`, so the Skill tool refuses them: read their `SKILL.md` and follow it exactly. Use the plugin's copy (`ls -d ${CLAUDE_CONFIG_DIR:-~/.claude}/plugins/cache/mattpocock/mattpocock-skills/*/skills/*/<name>`, newest version); only when the plugin is not installed, fall back to the project's `.claude/skills/<name>/`, then `~/.agents/skills/<name>/`. (Project copies of Matt's skills are usually older vendored copies; a project that really customises one should say so in its `CLAUDE.md`, and then that copy wins.) The others are called with the Skill tool as `mattpocock-skills:<name>`.

## Two entry points

- **The user names an issue** (`#12`, a URL): `gh issue view 12 --comments`. Restate its acceptance criteria in one line. If it has none, or they cannot fail (report-only, no threshold, "improve X"), propose checkable ones as a comment on the issue and get a yes before building. Size it like any request. At **Medium**, run `to-spec` but override its publish step: replace this issue's body with the spec (`gh issue edit N --body-file <spec.md>`, keeping the original report under a closing `## Original report` heading) instead of creating a new issue; at **Large** or **Fog**, this issue becomes the parent (or the map), and `to-tickets` makes the tickets its sub-issues. Either way there is one issue for the work, and the final PR closes it.
- **The user describes work in chat:** size it. Whether it gets an issue depends on the size (table below).

## Size it

Look before judging: an `Explore` sub-agent (or a quick grep) to see which modules, pages and CLAUDE.md rules the request touches. Check every row and pick the **largest** size whose signals apply; when unsure between two, pick the larger and let re-routing step it down.

| Size | Signals | Plan | Issue | Review |
|---|---|---|---|---|
| **Trivial** | The request decides everything: a typo, dictated text, an obvious one-line fix. | None. | None. | Self-check. |
| **Small** | One session, a few open decisions (behaviour, wording, defaults), no change to schema, external/model API calls, auth/security, money/limits or public interfaces. | `grilling`, or `grill-with-docs` (*read*) when a decision deserves an ADR or glossary entry. | None; decisions go in the PR body. | Self-check. |
| **Medium** | One session's build, but it touches schema, an external/model call or prompt, security, cost/limits, a public interface, or several screens. A reviewer will need the decisions written down. | `grill-with-docs` → `to-spec` (*read*). | `to-spec` publishes the spec as one issue; the PR closes it. | `code-review`. |
| **Large** | More than one context window, or parts that can land independently. | `grill-with-docs` → `to-spec` → `to-tickets` (*read*) → run the ticket graph. | Spec issue = parent; one sub-issue per ticket with native `blocked by` links; one PR per ticket closes it. | `code-review` per ticket PR; `retro` offered at the end. |
| **Fog** | The destination is unclear, or decisions depend on each other too deeply to settle in one sitting. | `wayfinder` (*read*) → then `to-spec` → `to-tickets` → build. | A `wayfinder:map` issue with decision tickets as children; when the map clears, the spec is published as a new parent issue that links the map, and the map is closed with a comment pointing to that parent. | As Large. |

**Bugs** follow the same table, sized by the fix. When the cause is not obvious, run `diagnosing-bugs` but **stop once the cause is confirmed by a minimal reproduction, before its fix phase**; then announce the size and do the fix on a branch, starting from that reproduction as the failing test.

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

Never route to: `implement` / `implement-spec` (commit to the current branch, no PR or `Closes`), `grill-me` (use `grilling`), the plugin's `in-progress/` skills, or language-specific skills that don't match the repo (`setup-ts-deep-modules`, `migrate-to-shoehorn`, `setup-pre-commit` are TypeScript/JS only).

## Announce, then go

> Size: **Medium** (setup: done): grill-with-docs → to-spec (spec issue) → build with tdd → code-review → one PR closing it. Say so if you want a different route.

Do not wait for approval: every routed skill has its own human gate (grilling waits for a shared understanding, `to-spec` checks seams, `to-tickets` has the breakdown approved, `wayfinder` resolves one ticket per session, setup confirms each section). That approval also covers publishing the issues and files it produces.

## Re-route at every phase boundary

- Grilling surfaced schema, external-call, security or cost decisions → step up to **Medium** (and run the setup now if it is missing).
- The spec grew independently landable parts or more than one context window of work → **Large**, run `to-tickets`.
- Each answer opens more unknowns than it closes → **Fog**, carry what is settled into the map.
- `wayfinder` finds no fog → step down to **Medium** or **Large**.
- Grilling settles in one round and nothing needs a record → finish as **Small**.

Say the change in one line and continue. Keep grilling → spec → tickets in **one unbroken context**.

## Build

The build is done by this session, by `Agent` with `subagent_type: "general-purpose"`, or by a `Workflow` when the user has opted into multi-agent orchestration. From **Medium** up, build with `tdd` (red → green → refactor) at the seams `to-spec` already agreed with the user, so `tdd`'s own seam question is already answered. For **Trivial / Small**, don't run the `tdd` skill: write the test first in the project's usual style when behaviour changes (a bug, a rule, a calculation); text or style changes need no new test.

- **Trivial / Small / Medium**: build here, or hand the spec issue to one general-purpose agent.
- **Large**: one fresh context, branch and PR per ticket (see below).

Each change: its own branch and worktree (the project's worktree rule if it has one; otherwise `Agent` with `isolation: "worktree"`) → tests first → the project's lint and test commands, reading the output → review → a PR.

**Verification belongs to the dispatching session, not the agent:** re-run lint and tests yourself, `git diff --check`, run the app for a UI change, and scan the diff for secrets and personal data. If the acceptance criteria need tests CI skips (live API, slow, manual), run them yourself and put the result in the PR's Test plan and a comment on the issue: green CI does not prove those criteria.

**Review (Medium and up):** run `code-review` against the default branch (`origin/main`) after verification and before merging. Its **Spec** axis checks the diff against the issue, its **Standards** axis against the repo's documented rules plus a code-smell baseline. It only finds `CODING_STANDARDS.md` / `CONTRIBUTING.md` by itself, so when the repo has neither, pass `CLAUDE.md` (or `AGENTS.md`) to it as the standards source. Fix every missing or wrong requirement it finds; fix or consciously decline (one line in the PR) each standards finding. Re-run verification after fixes.

**PR body**: the project's template, or the `pr` skill's shape, or **Summary** + **Test plan**; then one `Closes #N` line per issue it finishes, on its own line. GitHub closes them only when the PR merges into the default branch. With a **local Markdown tracker** nothing closes by itself: the PR also sets the ticket file's status to done (as `issue-tracker.md` describes), and the dispatcher does the same for the parent. A ticket PR also says `Part of #<parent>`; a PR that only advances an issue says `Part of #N` and nothing closes.

**Merge** only as the project's `CLAUDE.md` allows (e.g. "squash-merge when CI is green"). If it says nothing, stop at an open PR and ask the user to review. Never close an issue by hand when a PR fixes it. Hand-closes, always with a comment saying why: won't-fix, duplicate, a Large/Fog parent once all its children are closed, and the decision tickets and map that `wayfinder` resolves.

## Large: run the ticket graph in parallel

The **frontier** is every open ticket whose blockers are all closed (GitHub: `issue_dependencies_summary.blocked_by == 0`). Start every frontier ticket at once, each as a background general-purpose `Agent` in its own worktree cut from the default branch, launched in one message. Each builds with `tdd`, opens its own PR with `Closes #<ticket>` and does not merge. Two tickets that rewrite the same function or file block are dependent even if neither says so; add the edge. Shared files touched only by small additions (a config field, a model import) are not a reason to serialize: let the later PR rebase.

**Dispatcher loop:** on each completion, verify, `code-review`, merge (when allowed), and confirm the ticket closed → recompute the frontier → start what became available → repeat. When every child is closed, run the full suite once on the default branch, then close the parent yourself with a comment listing the merged PRs. That is the one manual close this skill makes: `to-tickets` leaves parents alone, the dispatcher does not. Never build two tickets in one context.

## Finish

- **Report**: what merged, which issues closed, the follow-up list, and the setup reminder if it was missing.
- **Retro**: after a Large or Fog route, or any route where the agent repeated a mistake, a check failed late, or a file was hard to find, offer `retro` (*read*) in one line. It proposes changes to the environment (`CLAUDE.md`, automated checks, coding standards); apply none without the user's yes.

## Things found along the way

A bug or idea outside the current issue is not fixed in this PR. The dispatching session keeps one running follow-up list (findings from exploration, grilling, `code-review` and every agent's report) and puts it under **Follow-ups** in the next PR it opens, or in its final report if no PR follows. When the user wants them tracked, open **one** issue for the batch (label `follow-up`), never one issue per finding, and never without the user's yes.

## Common mistakes

| Mistake | Instead |
|---|---|
| Running the whole chain (grill → spec → tickets) for a small change | Size first; most work is Trivial or Small and needs no issue. |
| Calling `mattpocock-skills:setup-matt-pocock-skills` (or any *read* skill) through the Skill tool | It is refused; read its `SKILL.md` and follow it. |
| Letting `diagnosing-bugs` apply the fix before sizing and branching | Stop at the confirmed cause; fix on the branch. |
| Blocking a typo fix on the Matt Pocock setup | Setup is required from Medium up; below that, remind in the report. |
| An issue for a typo fixed this minute | Trivial and Small need no issue; the PR is the record. |
| `Fixes #12` buried mid-paragraph or `#12` alone | Own line, `Closes #12`; one per issue. |
| Closing the issue by hand after merging | Let the merge close it; a still-open issue means the PR missed `Closes`. (Only a Large/Fog parent is closed by hand.) |
| A second spec issue for a bug that already has one | Rewrite the named issue into the spec; one issue per piece of work. |
| "Report how many…" accepted as acceptance criteria | Criteria must be able to fail: add a threshold or an expected result. |
| CI green taken as proof when the criteria need live or manual tests | Run those yourself and record the result on the PR and issue. |
| Merging a Medium+ PR without `code-review` | The Spec axis is the only check that the diff matches the issue. |
| Guessing at a bug's cause and patching the symptom | `diagnosing-bugs` first; its reproduction is the failing test. |
| Vague issue ("guard sometimes wrong") handed to an agent | Exact input, expected result, acceptance criteria first. |
| Secrets, personal data or full logs in an issue body | Issues on a public repo are public: describe, excerpt, redact. |
| Merging because CI is green when the project rule says the user merges | Follow the project's merge rule; default is user review. |
