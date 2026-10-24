---
description: Stage, commit and push current changes on a correctly named branch, then write the PR description
argument-hint: [JIRA-KEY] [feature|bug] [notes, e.g. "single commit" or "include plan file"]
---

Run these two agents in order.

**1. Commit and push.** Use the `git-committer` subagent to stage, commit and push the current changes, following the rules in `.claude/git-conventions.md`. Pass it this optional input from me:

$ARGUMENTS

**2. Write the PR description.** Only if step 1 pushed successfully, use the `pr-writer` subagent to write the PR title and description for the pushed branch. Pass it:
- the branch name and ticket key from step 1;
- any context you have from this conversation: whether AI produced the changes, which tools and agents were used (e.g. feature-planner, plan-executor, git-committer), and which parts they generated;
- the test, lint and build results and the coverage numbers reported by plan-executor, if it ran.

Only pass facts you actually know. Don't guess. If step 1 failed, skip step 2 and tell me why.

When both finish, give me one combined report:
- the branch, the commits, and whether the push succeeded;
- any files left out;
- the PR title, the target branch, and the path of the PR description file;
- the items I still need to fill in or tick myself;
- any issues that need my attention.
