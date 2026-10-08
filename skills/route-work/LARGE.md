# Large: run the ticket graph

Reached from [`SKILL.md`](SKILL.md) once `to-tickets` has published the tickets as sub-issues of the spec issue.

## The frontier

The **frontier** is every open ticket whose blockers are all closed (GitHub: `issue_dependencies_summary.blocked_by == 0`). Start every frontier ticket at once, each as a background general-purpose `Agent` in its own worktree cut from the default branch, all launched in one message. Each builds with `tdd`, opens its own PR with `Closes #<ticket>` and `Part of #<parent>`, and leaves merging to the dispatcher.

Two tickets that rewrite the same function or file block are dependent even if neither says so: add the edge. Shared files touched only by small additions (a config field, a model import) stay parallel; the later PR rebases.

## The dispatcher loop

On each completion: verify, `code-review`, merge (when the project allows), and confirm the ticket closed → recompute the frontier → start what became available → repeat. Each ticket gets its own context; this session only dispatches. When the project names a phase log ([`SKILL.md`](SKILL.md#at-every-phase-boundary)), the dispatcher also records there each ticket's start, its phase changes, its PR and its close.

When every child is closed, run the full suite once on the default branch, then close the parent with a comment listing the merged PRs. That is the one close the dispatcher makes by hand: `to-tickets` leaves parents alone.
