# Issue tracker: GitHub

Issues and specifications for this repository live in GitHub Issues at `gweiermann/problem-tree-app`. Use the authenticated GitHub integration when available; otherwise use the `gh` CLI.

## Conventions

- **Create an issue**: create it with a concise title and a Markdown body.
- **Read an issue**: fetch its body, comments, labels, assignees, and state.
- **List issues**: query the repository with the appropriate state and label filters.
- **Comment on an issue**: add a top-level issue comment.
- **Apply or remove labels**: preserve unrelated labels.
- **Close an issue**: record the resolution in a comment before closing it.

When using `gh`, infer the repository from `git remote -v` when inside a clone, or pass `--repo gweiermann/problem-tree-app`.

## Pull requests as a triage surface

**PRs as a request surface: no.**

## When a skill says "publish to the issue tracker"

Create a GitHub issue in `gweiermann/problem-tree-app`.

## When a skill says "fetch the relevant ticket"

Fetch the named issue with its comments, labels, assignees, and state.

## Wayfinding operations

The **map** is a single issue with **child** issues as decision tickets.

- **Map**: an issue labelled `wayfinder:map`, holding Destination, Notes, Decisions so far, Not yet specified, and Out of scope.
- **Child ticket**: link the issue to the map as a GitHub sub-issue. Where sub-issues are unavailable, add it to a task list in the map and put `Part of #<map>` at the top of the child's body.
- **Ticket labels**: use exactly one of `wayfinder:research`, `wayfinder:prototype`, `wayfinder:grilling`, or `wayfinder:task`.
- **Blocking**: use GitHub's native issue dependencies. Where they are unavailable, put `Blocked by: #<number>, ...` at the top of the child body.
- **Frontier**: the map's open, unblocked, unassigned children, in map order.
- **Claim**: assign the ticket to the developer driving the map before doing ticket work.
- **Resolve**: post the answer as a resolution comment, close the ticket, and append a one-line gist with a named link to the map's Decisions-so-far.
- **Deferred ideas**: apply `defer` when an idea is valid but outside the current iteration. Deferred work is not part of the active frontier.
