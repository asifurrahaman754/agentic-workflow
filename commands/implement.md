---
description: Implement a plan file, a planned Jira ticket, or a free-text description with the plan-executor agent
argument-hint: <plan-file.md | JIRA-KEY | description of what to build>
---

Use the `plan-executor` subagent to implement the following input:

$ARGUMENTS

If the input is a path to a `.md` file, use it as the plan. If it looks like a Jira key (e.g. `PROJ-123`), the agent should find the matching plan in `docs/feature-plans/` or `docs/bug-plans/`. Otherwise, treat it as a free-text description of what to build.

When the agent finishes, relay its report to me, including the files changed, the tests and coverage, the verification results, the definition-of-done checklist, any deviations from the plan, and open questions.
