# Mark changed files as reviewed

## Problem

Long reviews do not show which changed files the reviewer has already inspected. A simple
checkbox is insufficient because the agent may edit a reviewed file while the review is still in
progress.

## Behavior

Each file in the Changes navigator has one of three review states:

| State | Display | Meaning |
| --- | --- | --- |
| Not reviewed | No review marker | The current comparison has not been accepted |
| Reviewed | Green `✓` and subdued details | The displayed comparison matches the accepted one |
| Reviewed but changed | Orange `!` | The file changed after its comparison was accepted |

`R` toggles the current file's review state from either the navigator or the diff pane:

- On a not-reviewed file, it accepts the displayed comparison.
- On a reviewed file, it clears the review.
- On a reviewed-but-changed file, it accepts the new comparison.

`R` is withheld when Reviewr cannot certify the exact displayed comparison, including unreadable
files, over-budget files, lossy text or symlink paths, and dirty submodules. A later refresh can
make the action available once the comparison is certifiable.

Lowercase `r` remains refresh. The configurable action name is `toggle-reviewed`; normal
keybinding replacement and collision validation apply.

## Scope and lifetime

Review state exists only in the Changes view and only for the running Reviewr session. Reviewr
does not write it to Git, the worktree, or configuration.

Review state is isolated by the comparison the user is viewing, including:

- Uncommitted, Branch, Last turn, and Commits scopes;
- the selected branch base;
- the selected commit range; and
- the Last turn baseline.

A file that leaves a changeset loses its review state. If it later returns, it starts not reviewed.

## Refresh and identity

A review records the exact comparison shown to the reviewer, not line counts alone. The identity
includes the comparison endpoints, paths, change kind, modes, content, and binary status.

Refreshes reconcile review state only after a complete current snapshot lands. A stale or failed
refresh keeps the last consistent file list and review state. If a file changes without leaving the
changeset, its state becomes reviewed but changed.

Marking or reconciling a file adds no writes. Reviewr's existing refresh machinery may use its
private refs, session index copies, and snapshot objects, but never mutates the worktree, real
index, or branches.
