---
description: Plan a Jira ticket (feature or bug) or a free-text description with the feature-planner agent
argument-hint: <JIRA-KEY | feature or bug description>
---

Use the `feature-planner` subagent to create an implementation plan for the following input:

$ARGUMENTS

If the input looks like a Jira key (e.g. `PROJ-123`), the agent should fetch it from Jira and use the issue type to decide whether it is a feature or a bug. Otherwise, treat it as a free-text description and infer the type from the wording. When the agent finishes, relay its summary to me, including the plan file path, the detected type, the status, the root cause (for bugs), and any breaking changes, security issues, migrations, data repairs, and open questions.
