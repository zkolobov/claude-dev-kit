---
name: init-project
description: Scaffold the docs/ folder structure and empty template files for the claude-dev-kit AI development framework — stories.md, tech.spec.md, discovery.md. Use this skill whenever the user says "init-project", "scaffold project docs", "set up project docs", or wants to initialize the framework documentation for a new project. Accepts an optional project description: `init-project "My project description"`. This is NOT for initializing a CLAUDE.md file (use the init skill for that) — this creates the docs/ tracking structure.
---

# init-project

Scaffold folder structure and template files, then optionally chain into `discovery`.

**Two modes:**
- `init-project "description"` — scaffold + write description + auto-run `discovery`
- `init-project` (no args) — scaffold only, prompt user to run `discovery "description"` next

## Actions (in order)

1. Check for git repo. If `.git/` does not exist:
   - Derive a suggested repo name from the current directory name: lowercase, spaces → hyphens, strip special characters (e.g. `My Cool App` → `my-cool-app`).
   - Ask: `No git repo found. Initialize one as "<suggested-name>"? (press Enter to confirm, or type a different name, or "n" to skip)`
   - If the user confirms or provides a name → run `git init`. Record the confirmed name for the summary.
   - If the user types "n" → skip silently.

2. Create `docs/`, `docs/stories/`, and `.claude/` directories (skip any that already exist).

3. Create `.claude/conventions.md`:

Read `references/conventions-template.md` (bundled with this skill) and write its contents verbatim to `.claude/conventions.md` in the project.

4. Create `docs/stories.md`:

```markdown
# Stories

## discovery

| ID | Story | Description | Spec | Interview | Tests Defined | In Progress | Tests Passing | Done |
|----|-------|-------------|------|-----------|---------------|-------------|---------------|------|
```

5. Create `docs/tech.spec.md`:

```markdown
# Tech Spec: {Project Name}

## Chosen Stack

| Component | Technology | Reasoning |
|---|---|---|
| Backend | | |
| Cloud | | |
| Database | | |
| Frontend | | |
| Unit Tests | | |
| Integration Tests | | |

## Architectural Decisions


## Rejected Alternatives

| Alternative | Reason for rejection |
|---|---|

## Environments

- **Development:** local / Docker Compose
- **Staging:** 
- **Production:** 
```

6. Create `docs/discovery.md`:

If a description was provided, use it as the Source Prompt. Otherwise leave the placeholder.

```markdown
# Discovery: {Project Name}

## Date


## Source Prompt

> {description provided by user, or: "— run `discovery "your project description"` to populate"}

## Analysis

### Product Type


### Target Platform


### Scale and Audience


### Key Constraints


### Integrations


### Non-functional Requirements


## Default Stack Fit Assessment

| Component | Default | Fit | Reasoning |
|---|---|---|---|
| Backend | .NET 8 | | |
| Frontend | Angular | | |
| Database | MS SQL Server | | |
| Cloud | Azure | | |

## Proposed Alternatives


## Final Decision

**Confirmed by user:** —
**Chosen stack:** → see `docs/tech.spec.md`

**Rejected alternatives:**

| Alternative | Reason |
|---|---|

## Open Questions
```

7. Print summary and chain into discovery:

**If description was provided:**
```
✅ Project scaffolded.

Created:
  [git repo: <confirmed-name>]   ← omit this line if git was skipped
  docs/
  docs/stories/
  docs/stories.md
  docs/tech.spec.md
  docs/discovery.md
  .claude/
  .claude/conventions.md
  .claude/skills/

Starting discovery...
```
Then immediately invoke the `discovery` skill (no further input needed — it will read the description from `docs/discovery.md`).

**If no description was provided:**
```
✅ Project scaffolded.

Created:
  [git repo: <confirmed-name>]   ← omit this line if git was skipped
  docs/
  docs/stories/
  docs/stories.md
  docs/tech.spec.md
  docs/discovery.md
  .claude/
  .claude/conventions.md
  .claude/skills/

Next step: run `discovery "your project description"` to analyze the project and plan the first wave of work.
```

## Constraints

Do NOT analyze anything, create stories, make stack decisions, or fill in template fields beyond the Source Prompt.
