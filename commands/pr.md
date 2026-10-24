---
description: Write or refresh the PR title and description for the current branch with the pr-writer agent
argument-hint: [branch] [JIRA-KEY] [notes]
---

Use the `pr-writer` subagent to write the PR title and description for the branch I give, or the currently checked-out branch if I don't give one, following `.claude/pr-template.md` and the pull request rules in `.claude/git-conventions.md`.

Extra input from me (optional):

$ARGUMENTS

Also pass any context you have from this conversation: whether AI produced the changes, which tools and agents were used, which parts they generated, and any test, lint, build or coverage results. Only pass facts you actually know. Don't guess.

When the agent finishes, give me:
- the PR title and the target branch;
- the path of the description file;
- the items I still need to fill in or tick myself;
- any notes, such as unpushed commits.
