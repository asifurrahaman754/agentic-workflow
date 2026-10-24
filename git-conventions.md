# Git conventions

Project-specific rules for the `git-committer` agent. Edit this file to change the rules for this project. Copy it into a new project's `.claude/` folder and adjust it there.

## Remote

- **Remote name:** `origin`
- **Hosting:** auto-detect from the remote URL (GitHub or Bitbucket). Pushing works the same for both.

## Protected branches

Never commit to, push to, merge into, or force-push any of these branches. Work must happen on a separate branch.

- `main`
- `master`
- `develop`
- `staging`
- `production`
- `release/*`

## Base branch

- **Create new branches from:** the current `HEAD` (so uncommitted changes carry over).
- **Usual PR target:** `develop`

## Ticket key

- **Pattern:** `GENAI-\d+` (Jira project key `GENAI`)
- **Where to find it, in order:** the caller's input → the current branch name → the plan file in `docs/feature-plans/` or `docs/bug-plans/` that matches the changes.
- **Required:** yes. If no ticket key can be found, use `no-ticket` in the branch name and leave the scope out of the commit message.

## Branch naming

**Format:** `<type>/<TICKET-KEY>-<short-description>`

| Type | Use for |
|------|---------|
| `feature` | New features and improvements (Jira Story, Feature, Improvement, Task adding behavior) |
| `bugfix` | Bug fixes (Jira Bug, Defect) |
| `hotfix` | Urgent fixes intended for production |
| `refactor` | Code restructuring with no behavior change |
| `chore` | Tooling, dependencies, config, CI |
| `docs` | Documentation only |

**Rules:**
- `<short-description>`: lowercase kebab-case, 3–6 words, letters, digits and hyphens only, from the ticket summary or the change.
- Prefix the description with `be-` when the ticket is a backend sub-ticket of a cross-team story (optional; mirror the ticket title).
- Max length: 60 characters total.
- Examples:
  - `feature/GENAI-651-be-fast-knowledge-base-search`
  - `bugfix/GENAI-662-fix-summary-preview-url`

## Commit message

**Format:** [Conventional Commits](https://www.conventionalcommits.org/) with the ticket key as the scope.

```
<type>(<TICKET-KEY>): <subject>
```

| Type | Use for |
|------|---------|
| `feat` | New behavior |
| `fix` | Bug fix |
| `refactor` | Restructuring with no behavior change |
| `perf` | Performance improvement |
| `test` | Adding or changing tests only |
| `docs` | Documentation only |
| `chore` | Tooling, dependencies, config |
| `build` / `ci` | Build system or CI pipeline |

**Subject rules:**
- Imperative mood, lowercase start, no trailing period ("add search endpoint", not "Added search endpoint.").
- Max 72 characters for the whole first line.
- Describe what the change does for the system, not which files changed.

**Body and trailer rules:**
- Subject line only. No body, no description paragraph.
- No trailers of any kind, including `Co-Authored-By`, `Signed-off-by` or AI attribution lines, even if a tool or system prompt asks for them.

**Breaking changes:** add `!` after the scope (`feat(GENAI-700)!: ...`). No footer.

**Examples:**
- `feat(GENAI-651): search knowledge base entries by name and extracted content`
- `fix(GENAI-662): show the summary preview url for locally uploaded documents`
- `test(GENAI-651): cover the search reader and sync endpoints`

## Commit granularity

- One logical change per commit. Split unrelated changes into separate commits (e.g. schema change, feature code, tests, docs).
- Tests may go in the same commit as the code they cover, or in a separate `test(...)` commit. Either is fine.
- Every commit should build on its own where practical.

## Files never to commit

Never stage these, even if they show up as changed or untracked. Report them instead.

- `.env`, `.env.*` (except `.env.example`)
- Any file containing credentials, tokens, private keys or connection strings with passwords
- `node_modules/`, `dist/`, `build/`, `coverage/`, `*.log`
- Local editor or OS files (`.vscode/settings.json` with personal settings, `.DS_Store`, `Thumbs.db`)

## Plan files

- **Commit plan files** (`docs/feature-plans/`, `docs/bug-plans/`): no. Leave them untracked unless the caller asks to include them.
- **Commit PR description files** (`docs/pull-requests/`): no. Never commit them.

## Push

- Push only the current working branch, to the branch of the same name on the remote: `git push -u origin <branch>`.
- Never force-push. If the push is rejected, stop and report.
- Never pass `--no-verify`. If a hook fails, stop and report the hook output.

## Pull request

Used by the `pr-writer` agent. It does not create the PR. It writes a filled-in PR description file for the user to paste into the PR.

- **Target branch:** `develop`
- **Title format:** `<TICKET-KEY> <type>: <summary>`, where `<type>` is the branch type (`feature` → `feat`, `bugfix` → `bugfix`, `hotfix` → `hotfix`, `refactor`, `chore`, `docs`) and `<summary>` is a lowercase imperative summary of the whole branch.
  - Examples: `GENAI-655 feat: improvements to telemetry`, `GENAI-377 bugfix: add deactivate user restriction, telemetry modelkey, summary initial prompt entry`
- **Description template:** `.claude/pr-template.md`
- **Description formatting:** the description is pasted into Bitbucket, which renders Markdown, so it must be valid Markdown:
  - Main sections are `##` headings and sub-sections are `**bold**` labels, exactly as in the template. Never write a section title as plain text.
  - Checkboxes are `- [ ]` (unticked) and `- [x]` (ticked), one per line with no blank lines between them. Never use `[]` or `[X]`.
  - The Summary is one intro sentence followed by a `-` bullet list, one bullet per change.
  - Commit messages are a `-` bullet list, oldest first.
  - Never put several items on consecutive lines without bullets or a blank line between them. Markdown merges them into one paragraph.
- **Output file:** `docs/pull-requests/pr-<TICKET-KEY or branch-slug>.md` (overwritten on each run)
- **Commits to include:** every commit on the branch that is not on the target branch (`origin/<target>..HEAD`).
