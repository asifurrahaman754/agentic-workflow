---
name: better-plan
description: Plan a new feature or change in plain, non-technical language with Mermaid diagrams showing how the system works today and how it will work after the change, plus what could break. Use this whenever the user asks to plan, design, scope, or think through a feature, wants to understand how a change affects the system, says they don't know the codebase well, or asks for a plan that is "simple", "visual", "easy to understand" or "not too technical". Input is either a Jira ticket link/key or a written feature description. Prefer this over a file-by-file technical plan unless the user explicitly asks for one.
---

# Visual Feature Plan

The reader may not know this codebase. Your job is to help them **understand the change**, not to list code edits. A good plan lets a product manager, designer, or new developer say "I see what will change, what stays the same, and what might break."

## Ground rules

- **Do not change any code** while running this skill. Only read and explore, then write the plan file. Wait for the user to approve the plan before implementing anything.
- **Plain language first.** Describe parts of the system by what they _do_ ("the login screen", "the service that sends emails", "the table that stores orders"), not by file paths or function names.
- **Technical terms are fine; code references are not.** Use normal technical terms freely (message queue, API endpoint, migration, index, worker, SignalR, etc.) without explaining them. What to avoid in the main body is references to code itself: file paths, file names, class, function, or variable names, and code snippets.
- **No file lists, code snippets, or diffs in the main body.** Those go only in the "For developers" section at the end.
- **Visuals first, text minimal.** Show it before you explain it. Use as many diagrams and tables as the plan needs to be understood, and keep the text around them short:
  - Lead each section with a diagram or table where one fits; add text only for what the visual can't show.
  - Keep text to 1–3 short sentences per section or block. Prefer a table row or a short bullet over a paragraph.
  - Don't repeat in words what a diagram or table already shows.
  - Add an extra diagram whenever it makes something clearer, for example: how the user navigates to a new page (`flowchart LR` of clicks), the states something moves through (`stateDiagram-v2`), or how a risky part could fail.
  - Cut filler, hedging, and introductions ("This section describes…"). Every sentence must tell the reader something new.

## Input

The user's input: $ARGUMENTS

(If the line above is empty or shows the literal text `$ARGUMENTS`, use whatever the user wrote after the skill command in their message.)

The input is one of two things:

- **A Jira ticket link** (e.g. `https://asthait.atlassian.net/browse/GENAI-2`) or a bare ticket key (e.g. `GENAI-2`)
- **A written description** of the feature (e.g. `i want a notification system for the workflow`)

## Guards: check these before making any plan

Run these guards **in order**, before writing any plan file. If a guard fails, reply with its message and **stop**. Do not guess, and do not write a plan. Only when every guard passes do you move on to the Workflow.

**No input given.** If there is no ticket and no description, reply:

> I need a feature to plan. Please run the command again with either a Jira ticket or a short description, for example:
> `/visual-feature-plan https://your-company.atlassian.net/browse/PROJ-123`
> `/visual-feature-plan add a notification system for the workflow`

If the input is a written description, skip guards 1–4. If it's too vague to plan anything at all (e.g. "make it better"), reply:

> That's a bit too broad for me to plan. Could you say what the feature should do and who it's for? For example: "notify approvers by email and in-app when a workflow step is waiting on them."

If the input is a Jira ticket, take the ticket key from the link (the part after `/browse/`, e.g. `GENAI-2`), fetch it with the available Jira/Atlassian tools, and check guards 1–4. **Never guess the feature from the ticket key or URL.**

### Guard 1: No Jira connection is set up in the session

No Jira/Atlassian tool is available. Reply:

> I can't read Jira tickets in this session because no Jira/Atlassian connection is set up. You can either connect the Atlassian MCP server and run the command again, or paste the ticket's description into the command instead.

### Guard 2: The ticket can't be opened

The ticket doesn't exist, or this account doesn't have permission to view it. Reply:

> I couldn't open ticket **<KEY>**. It may not exist, or this account may not have permission to view it. Please check the link or paste the ticket's description instead.

### Guard 3: The Jira sign-in has expired

The fetch fails with an authentication error. Reply:

> My Jira connection needs you to sign in again before I can read ticket **<KEY>**. Please re-authenticate the Atlassian connection, or paste the ticket's description instead.

### Guard 4: The ticket has too little detail

The ticket opens, but has only a title or not enough to plan from. (Read the summary, description, acceptance criteria, sub-tasks, linked issues, and comments before deciding.) Reply:

> Ticket **<KEY>** ("<title>") doesn't have enough detail to plan from. Please add a description or acceptance criteria to the ticket, or describe the feature in the command.

### Guard 5: The feature already exists, or can be done with a similar existing action

Applies to both tickets and descriptions. Search the codebase (read-only) for anything that already does what's being asked: matching pages and UI actions, API endpoints, background jobs, queue consumers, database tables, settings or feature flags, and similar names or wording. Check related features too (e.g. for "notifications", look for existing emails, alerts, webhooks, activity feeds, or event publishing).

- **Already fully available.** Reply, then stop:

  > This already exists, so I haven't made a new plan.
  > **What it is:** <plain-language description>
  > **Where to find it:** <page + route + area of the page, or the part of the system>
  > **How to use it:** <short steps>
  > If you need something it doesn't do, tell me what's missing and I'll plan just that part.

- **Can be done with a similar existing action** (something already covers most of it, or could be reused or extended). Reply, then stop and wait for the user's choice:

  > Something close to this already exists.
  > **What exists:** <plain-language description, and where it is>
  > **What it already covers:** <…>
  > **What it doesn't cover:** <…>
  > How would you like to go ahead?
  >
  > 1. Use the existing feature as it is (no plan needed)
  > 2. Plan a smaller change that extends the existing feature
  > 3. Plan a new, separate feature anyway

  If the user picks 2 or 3, continue to the Workflow and mention the existing feature in the plan.

- **Nothing similar found.** Continue to the Workflow. Note in the plan's **Already checked** line what you searched for, so the reader knows it was verified.

### When all guards pass

If the input came from a ticket, start the plan with a **Source** line: ticket key, title, and link. Treat the ticket as the source of truth for scope.

## Workflow

1. **Explore the codebase (read-only).** Find the parts involved: entry points, data stored, external services, and anything else that depends on them. Pay special attention to things that _other_ parts rely on, since these are where breakage happens. (Reuse what you found during guard 5.)
2. **Ask clarifying questions, then wait.** Before writing anything, ask the questions whose answers would change the plan. Do not write the plan until the user answers (see "Clarifying questions" below).
3. **Write the plan** as a Markdown file at `docs/feature-plans/<feature-name>.md` (kebab-case name; create the folder if needed), using the structure and diagram rules below. Plain Markdown only: no HTML tags.
4. **In the terminal, give only a 3–5 sentence summary** plus the file path, then ask whether they want changes to the plan or want to proceed.

## Clarifying questions

Ask these **after** exploring the codebase, so the questions are specific to this project ("Approvals already have a 'delegate' option. Should delegates be notified too?") rather than generic.

**What to ask about.** Go through this edge-case checklist and ask only where the answer isn't already clear from the ticket, the description, or the code, and where it would change the plan:

| Area                               | Example questions                                                                                     |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------- |
| Scope                              | What's in the first version and what can wait?                                                        |
| Users and permissions              | Which roles get this? What happens for users without access, guests, or admins?                       |
| Triggers and timing                | What exactly starts it? Instant, delayed, scheduled, batched?                                         |
| Empty and first-time states        | What do new users or users with no data see?                                                          |
| Errors and failures                | What should happen if a step, external service, or email fails? Retry, show an error, or stay silent? |
| Duplicates and concurrency         | What if it happens twice, or two people act on the same thing at once?                                |
| Existing data                      | Does it apply to old records, or only new ones? Is a backfill needed?                                 |
| Deletion and undo                  | What happens when the related item is deleted, cancelled, or reverted?                                |
| Volume and limits                  | Expected amount of usage? Any rate limits, quotas, or cleanup needed?                                 |
| Settings and opt-out               | Can users or admins turn it on/off or configure it? What are the defaults?                            |
| Multi-tenant, time zones, language | Do these matter here?                                                                                 |
| Devices                            | Does it need to work on mobile or small screens?                                                      |

**How to ask.**

- One message, numbered questions, grouped by area. Ask as many as needed, but skip anything you can confidently decide from the code.
- For each question, give a **suggested answer** based on what you found in the codebase, so the user can simply reply "use your suggestions" or answer only the ones they disagree with.
- If there are no real questions, say so in one line and go straight to writing the plan.

**After the answers.**

- Use the answers in the plan, and record each edge case in the "Edge cases" section with who decided it.
- If the user skips a question or says "use your suggestions", use your suggested answer and mark it as an assumption.
- Anything still genuinely open goes under "Decisions you need to make".

## Plan structure

Use these sections, in this order:

```markdown
# Feature plan: <Feature name>

**Created:** <date> · **Status:** Draft for review
**Source:** <Jira key + title + link, or "Description provided by user">
**Already checked:** <what existing features were searched for and what was found, e.g. "No existing notification system; the workflow already publishes events to RabbitMQ, which this plan reuses.">

## How it works today

Short explanation + Mermaid diagram of the current flow, limited to the parts relevant to this feature.

## How it will work after

Same diagram style, with new and changed parts highlighted. Keep the layout similar to the "today" diagram so the difference is easy to spot. Add a one-line legend under it.

## What changes for users

What people will see or do differently. If nothing visible changes, say so.

## Where in the UI this changes

(Include only if the plan touches the frontend.)
For every change the user can see, say exactly where it is: the page (with its URL/route, e.g. `/settings/billing`), the area of the page (header, sidebar, a specific tab, modal, form, table), and what appears, changes, or disappears there.
| Page (route) | Where on the page | What changes | New or changed |
|---|---|---|---|
If a whole new page, modal, or menu item is added, say how the user gets to it (e.g. "Settings → Billing → new 'Invoices' tab").

## Parts of the system involved

| Part (plain name) | What it does today | What changes | Size of change |
| ----------------- | ------------------ | ------------ | -------------- |

## Database changes, and why they're needed

(Include only if the plan touches the backend and needs a database change. If no database change is needed, write one line saying so and why the existing structure is enough.)
For each change (new table, new column, changed column, new index, data migration):
| Change | What it stores, in plain words | Why it's needed | Effect on existing data |
|---|---|---|---|

- **Why it's needed** must explain why the existing tables can't do the job (e.g. "one order can now have many invoices, so a single column on Orders can't hold them").
- Say whether existing data needs to be moved or filled in, and whether anything that already reads these tables could be affected.
- If there are relationships between tables, add a Mermaid `erDiagram` showing only the new/changed tables and the tables they connect to.

## What could break, and how we'll protect it

For every existing feature or part that depends on something being changed: the risk in plain words, and the safeguard (tests, keeping old behaviour working, gradual rollout, etc.). Be honest; this is the most important section for people who don't know the codebase.

## Edge cases

| Case | What happens | Decided by |
| ---- | ------------ | ---------- |

One row per edge case from the clarifying questions (and any others found while planning). "Decided by" is **User** (answered a question) or **Assumption** (suggested answer used; easy to change).

## Decisions you need to make

Only questions still open after the clarifying questions, with 2–3 options each and a short recommendation. If none are left, write "None".

## Build order

Numbered steps in plain language, each leaving the system in a working state.

## For developers

Technical detail: files to change, new files, data/schema changes, messages/events, and tests to add.

## Full user flow

The complete journey once the feature is built, from where the user starts to the final result, so anyone can follow it end to end:

1. A numbered, step-by-step walkthrough in plain words: what the user does, what they see on screen (page + area), and what happens behind the scenes at each step.
2. Include the main alternative paths: errors, empty states, cancelling, missing permissions, and what the user sees in each case.
3. A Mermaid `sequenceDiagram` of the main (happy) path, with the user as the first participant and the system parts (frontend, backend, database, queue, etc.) as the others.
```

## Diagram rules

- Use fenced Mermaid blocks:

  ````markdown
  ```mermaid
  flowchart TD
      A[Frontend - Next.js] --> B[Backend - .NET]
      B --> C[(PostgreSQL)]
      B --> D[RabbitMQ]
  ```
  ````

- Keep each diagram under ~15 boxes; split into two diagrams if bigger.
- Label boxes with plain names plus the technology where helpful ("Checkout page - Next.js"), never class or file names.
- Use shapes consistently: `[ ]` for apps/services, `[( )]` for databases, `{{ }}` for queues or external services, `([ ])` for people/users.
- Label arrows with what flows between them: `B -->|sends order details| D`.
- Use `flowchart TD` or `flowchart LR` for overall structure, and `sequenceDiagram` for step-by-step interactions (e.g. "what happens when a user clicks Pay").
- In the "after" diagram, highlight new and changed parts with these classes and add a legend line below the diagram ("🟩 new · 🟨 changed"):

  ````markdown
  ```mermaid
  flowchart TD
      A[Frontend - Next.js] --> B[Backend - .NET]
      B --> C[(PostgreSQL)]
      B -->|publishes 'order placed'| D{{RabbitMQ}}
      D --> E[Email worker]

      classDef new fill:#d1f7d6,stroke:#1a7f37,color:#0b3d1a
      classDef changed fill:#fff1c2,stroke:#9a6700,color:#3d2a00
      class E new
      class B changed
  ```
  ````

- Example for the full user flow:

  ````markdown
  ```mermaid
  sequenceDiagram
      actor U as User
      participant F as Frontend - Next.js
      participant B as Backend - .NET
      participant DB as PostgreSQL
      participant Q as RabbitMQ
      U->>F: Clicks "Download invoice" on /orders/123
      F->>B: Requests invoice for order 123
      B->>DB: Looks up order and invoice
      DB-->>B: Invoice details
      B->>Q: Publishes 'invoice requested'
      B-->>F: Invoice link
      F-->>U: PDF opens in new tab
  ```
  ````

- Example for database changes:

  ````markdown
  ```mermaid
  erDiagram
      ORDERS ||--o{ INVOICES : "has many"
      INVOICES {
          uuid id
          uuid order_id
          decimal amount
          timestamp issued_at
      }
  ```
  ````

- Avoid characters that break Mermaid inside labels (unquoted parentheses, quotes, `#`, `;`). Wrap a label in double quotes if it needs special characters: `A["Orders (v2)"]`.
