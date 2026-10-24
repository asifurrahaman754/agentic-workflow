---
description: Read the unresolved review comments on a Bitbucket PR and apply the valid ones with the pr-comment-resolver agent
argument-hint: <bitbucket-pr-url | pr-number> [extra instructions]
---

Use the `pr-comment-resolver` subagent to resolve the review comments on the following pull request:

$ARGUMENTS

If the input is a Bitbucket PR URL, use it as is. If it is a number, the agent should take the workspace and repo from the `origin` remote. Pass on any extra instructions I gave.

When the agent finishes, relay its report to me: the comments it applied and what it changed, the comments it rejected and why, the ones that need my decision, and the build and test results.
