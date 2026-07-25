# Project instructions for Claude

## Git commits — do not commit without explicit instruction

The user tests changes locally and commits manually as their normal workflow.

- **Never run `git commit` (or `git add` in preparation for one) unless the user explicitly asks for a commit in that turn.** Finishing a task, fixing a bug, or completing a review round is not implicit permission to commit — leave changes staged/unstaged in the working tree for the user to review and commit themselves.
- This applies even during multi-step work like implementation plans or subagent-driven development where frequent commits are the documented default — confirm with the user first if it's not already clear they want that mode for the current task.
- When the user does explicitly ask for a commit, also push it unless they say otherwise ("always push a commit when requested").
