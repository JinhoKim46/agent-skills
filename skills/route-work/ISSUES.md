# Issues

Reached from [`SKILL.md`](SKILL.md) when the user names an issue, or the route will create or close one. The rule throughout: **one issue per piece of work**, closed by the PR that finishes it.

## A named issue

1. `gh issue view N --comments`, and restate its acceptance criteria in one line.
2. Criteria must be able to fail: a threshold, an expected result, an exact input. When the issue has none, or only report-only ones ("report how many…", "improve X"), propose checkable criteria as a comment on the issue and get a yes before building.
3. Size it like any request. The named issue stays the one issue for the work:
   - **Medium**: run `to-spec`, but publish by rewriting this issue: `gh issue edit N --body-file <spec.md>`, with the original report kept under a closing `## Original report` heading.
   - **Large**: this issue becomes the parent, and `to-tickets` makes the tickets its sub-issues.
   - **Fog**: this issue becomes the map (below).

## Creating issues

- **Labels (GitHub):** before the first `gh issue create`, run `gh label list` and create any label this route will apply that is missing (`ready-for-agent`, `follow-up`, `wayfinder:*`, or the names in `docs/agents/triage-labels.md`).
- **Public by default:** an issue on a public repo is public. Describe, excerpt and redact; keep secrets, personal data and full logs out of it.

## Fog

`wayfinder` opens a `wayfinder:map` issue with the decision tickets as children. When the map clears, `to-spec` publishes the spec as a new parent issue that links the map, and the map is closed with a comment pointing to that parent.

## Closing

- A PR closes each issue it finishes with its own `Closes #N` line. A ticket PR also says `Part of #<parent>`; a PR that only advances an issue says `Part of #N`, and nothing closes.
- **Local Markdown tracker:** nothing closes by itself. The PR sets the ticket file's status to done (as `docs/agents/issue-tracker.md` describes), and the dispatcher does the same for the parent.
- A merged PR closes its issues. Close by hand only these, each with a comment saying why: won't-fix, duplicate, a Large or Fog parent once all its children are closed, and the decision tickets and map that `wayfinder` resolves. An issue still open after its PR merged means the PR missed its `Closes` line.
