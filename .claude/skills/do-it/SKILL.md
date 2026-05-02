---
name: do-it
description: Execute development work on a story — review open questions, write test plans, generate code, run tests, and advance story status. Use this skill whenever the user says "do-it", "do it", "continue story", "work on #ID", "start story", "next step", "elaborate epic", "epic #ID", names a phase to work through, or wants to actually make progress on anything in the backlog. This is the primary execution skill — it picks up exactly where work left off. Also handles blocking/unblocking stories and elaborating epics into full stories. When in doubt whether the user wants to act vs just look, use this skill — it shows a menu first. For read-only overview, use the `status` skill instead.
---

# do-it

The execution worker. Reads current story state and performs the next concrete action.

Read `.claude/conventions.md` for status→checkbox mapping, transition rules, test commands, and status icons.

---

## Mode 1: Specific Story — `do-it #0008`

### Step 1 — Locate the story

Strip any leading `#` from the ID (users often type `#0008`), then use glob `docs/stories/{ID}-*/` to find the folder (e.g. `docs/stories/0008-*/`).
Do not re-derive the slug — the actual folder name may differ from what you'd generate.
Also read `docs/stories.md` to confirm phase and story name.

### Step 2 — Read story state

Read all five files:
- `spec.md` — current status, acceptance criteria, technical details
- `interview.md` — open/closed questions
- `tasks.md` — task breakdown and completion status
- `tests.md` — test plan and results
- `progress.md` — blockers, notes to resume

### Step 3 — Execute next action based on status

| Status | Action |
|---|---|
| `draft` | Review `interview.md`. If open questions → transition to `review`. If none → transition to `ready`. |
| `review` | If story has UI, see UI Check below first. Then present each open question. Record answers in `interview.md`, close questions, update counters. When all closed → `ready`. |
| `blocked` | Display blocker from `progress.md`. Suggest tasks that could unblock it. Do not advance status. |
| `ready` | Check dependencies (see Dependency Check below) → Read `interview.md` + `tech.spec.md` → decompose tasks per `conventions.md` (specific library/approach per task; new decisions → `tech.spec.md ## Architectural Decisions`) → confirm task list with user → write test plan in `tests.md` → `tests_defined`. |
| `tests_defined` | Transition to `in_progress` (update `spec.md`, `progress.md`, `stories.md`) → implement tasks in order, skip `test` type → after last impl task run tests (Step 5). |
| `in_progress` | Read `tasks.md`. If there are pending implementation tasks (`infra`/`data`/`backend`/`api`/`design`/`frontend`), continue from the first one. If only `test` tasks remain → run tests (Step 5). |
| `tests_passing` | Summarize completed work. Ask user to review. When approved → transition to `done` → then run git wrap-up (commit + PR, see "Git: Wrap-up after story completes" below) → then offer suggested actions. |
| `done` | Report story complete, show summary. Then offer suggested actions (see "Suggested Actions after Story Work" below). |

### UI Check (during `review`)

If the story involves UI (pages, forms, screens, components) and `interview.md` has no question about design yet, add one before presenting the other open questions:

> "Is there a design or mockup for this feature? If yes, where is it — file path, Figma link, or dropped into `artifacts/`?"

Record the answer in `interview.md` as a closed question, and if a path or URL was given, add it to `spec.md → ## Technical Details`:

```markdown
**Design reference:** artifacts/checkout-flow.fig  (or: https://figma.com/...)
```

This reference will be read during task decomposition at the `ready` stage.

---

### Dependency Check (before `ready → tests_defined`)

Read `spec.md → ## Dependencies → Depends on:`. If it lists story IDs, look them up in `docs/stories.md` and check their checkbox pattern. If any dependency is not `done` (all ☑), warn the user:

```
⚠️  Story #0008 depends on #0005 (Database Schema), which is not done yet.
Proceeding may cause integration issues. Continue anyway? (y/n)
```

If the user confirms, proceed. If no dependencies are listed or all are done, proceed silently.

### Step 4 — After each implementation task completes

1. Mark the task `✅ done` in `tasks.md`, add the PR link if available.
2. Update `## Summary` counters in `tasks.md`.
3. When all non-test tasks are done, proceed to run tests (Step 5).

### Git: Branch at story start

At the very beginning of Step 3 (before any status-based action), if `.git/` exists and the current branch is not already a feature branch for this story (check `git branch --show-current`):

1. Derive a branch name from the story slug: `feature/{story-slug}` (use the folder slug, e.g. `feature/0008-user-registration`).
2. Ask: `Create branch "feature/0008-user-registration"? (press Enter to confirm, or type a different name)`
3. On confirmation (or custom name), run `git checkout -b <branch-name>`.
4. If the user declines, continue on the current branch silently.

---

### Step 5 — Run and parse tests

Run the appropriate test command (see `.claude/conventions.md`).

Parse the output:
1. Match test names to rows in `tests.md` Plan table.
2. Set each `Result` cell: `✅` / `❌` / `⚠️`.
3. Update `Summary` line with pass count.
4. Populate `Failure Log` for `❌` entries.
5. Mark test tasks `✅ done` in `tasks.md`.
6. If all `✅` → transition to `tests_passing`.
7. If any `❌` → stay in `in_progress`. Show the Failure Log. Attempt to fix failing tests by correcting the implementation (not the tests). After each fix, re-run the suite. If after two fix attempts tests still fail, stop and ask the user for guidance — describe exactly what's failing and why the fix isn't straightforward.

### Git: Wrap-up after story completes

When a story transitions to `done` (user approved, all files updated), if `.git/` exists, run the following sequence:

1. **Stage all changes** — run `git add -A` and show a brief summary of what was staged (file count).

2. **Commit message** — suggest a message based on the story name and what was implemented, e.g.:
   `feat(#0008): implement user registration with email verification`
   Ask: `Commit with this message? (press Enter to confirm, or type a different message)`
   Then run `git commit -m "<confirmed message>"`.

3. **Push** — ask: `Push to origin? (y/n)`
   If yes, run `git push -u origin <current-branch>`.

4. **PR** — ask: `Create a pull request? (y/n)`
   If yes, run `gh pr create --title "<story name>" --body "Closes #<story-id>\n\n<one-line summary of what was implemented>"` and print the PR URL.

If `.git/` does not exist, skip this section entirely.

---

### Suggested Actions after Story Work

After completing a story (`done`) or reaching `tests_passing`, read `docs/stories.md` and find the next most actionable story (prefer `in_progress` → `tests_defined` → `ready` → `review` → `draft`). Then call the `AskUserQuestion` tool:

- If another story exists: ask "What would you like to do next?" with options `/do-it #XXXX` (label "Next story", description "Continue with the next most actionable story") and `/discovery` (label "Discovery", description "Run discovery to find new work").
- If no other stories exist: ask with only `/discovery` (label "Discovery", description "Run discovery to find the next wave of work").

Replace `#XXXX` with that story's actual ID.

---

### Step 6 — After every significant step: sync state

1. **`tasks.md`** — update task status and Summary counters.
2. **`progress.md`** — update `**Status:**`, `**Last updated:**`, and `## Notes` with enough context to resume in a brand-new session.
3. **`spec.md`** — update `**Current status:**` and `**Updated:**`.
4. **`docs/stories.md`** — update this story's checkbox row using the mapping in `.claude/conventions.md`.

---

## Mode 2: Phase — `do-it mvp-backend`

Read `docs/stories.md`. Find all stories in the given phase. Apply this priority:

1. `in_progress` → continue the first one (Mode 1).
2. `tests_defined` → list candidates (test plan ready, implementation next), let user choose, then Mode 1.
3. `ready` → list candidates, let user choose, then Mode 1.
4. `blocked` → show blockers, suggest what could unblock them.
5. `review` → list candidates (open questions need answers), let user choose, then Mode 1.
6. `draft` → list candidates (not yet reviewed), let user choose, then Mode 1.

---

## Mode 3: No argument — `do-it`

Read `docs/stories.md` across all phases. Build a full picture of what's available, then let the user choose.

Show this menu:

```
## What would you like to work on?

🔄 In progress
  #0006 Infrastructure Setup (mvp-backend)
  #0011 Login Page (mvp-frontend)

🟡 Tests defined (ready to implement)
  #0010 Password Reset (mvp-backend)

🟡 Ready to start
  #0007 Database Schema (mvp-backend)
  #0008 User Registration (mvp-backend)

💬 In review (open questions to answer)
  #0003 Security Model (discovery)

📝 Draft (not yet reviewed)
  #0004 Domain Model (discovery)

⚪ Epics (not yet elaborated)
  #0012 Payment Integration (mvp-backend)
  #0015 Admin Dashboard (post-mvp)

Type a story number to work on it, type "epic #ID" to elaborate an epic into a full story,
or type "discovery" to run discovery and find new work.
```

Rules:
- Show only non-empty sections. Skip sections with nothing in them.
- To distinguish `draft` from `review`: read `progress.md → **Status:**` for each all-☐ story.
- If `In progress` has exactly one story, you may continue it directly (ask "Continue #ID — {Name}? [y/n]" to confirm).
- If everything is empty (no in_progress, no ready, no epics), say: "The backlog is empty. Run `discovery` to find the next wave of work." Then stop.
- Epics section lists only `epic` rows (rows with `—` in all checkbox columns).

After displaying the menu, call the `AskUserQuestion` tool. Include `/discovery` always, and one `/do-it #ID` for the most actionable story (prefer `in_progress` → `tests_defined` → `ready` → `review` → `draft` order):

- If a story exists: ask "What would you like to work on?" with options `/do-it #XXXX` (label "Top story", description "Jump to the most actionable story") and `/discovery` (label "Discovery", description "Find new work to add to the backlog").
- If the backlog is empty: ask with only `/discovery` (label "Discovery", description "Find the next wave of work").

Replace `#XXXX` with the actual story ID.

When the user picks a story → Mode 1.
When the user picks "epic #ID" → elaborate the epic (see below).

---

## Elaborate Epic — `do-it epic #0012`

Turn an existing epic row into a full story (create the folder and all five files).

### Step 1 — Find the epic

Read `docs/stories.md`. Confirm the row has `—` in all checkbox columns (it's still an epic, not yet elaborated).

### Step 2 — Confirm with the user

Show what you found:

```
Epic #0012: Payment Integration (mvp-backend)
Description: {description from stories.md}

This will create the full story folder with spec, interview, tasks, tests, and progress files.
The epic row will be updated to a `draft` story row.

Proceed? (y/n)
```

### Step 3 — Invoke `create-story`

Call `create-story` with the phase and name from the epic row, using the same ID (do not assign a new one).

After `create-story` completes, update the epic row in `docs/stories.md`:
- Replace `—` placeholders with the standard `draft` checkbox pattern: `| ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |`

### Step 4 — Continue as Mode 1

The story is now in `draft` status. Proceed with `do-it #ID` immediately.

---

## Block / Unblock — `do-it block #0008 "reason"` / `do-it unblock #0008`

To block: locate the story folder (glob), read `spec.md` to get the current status, record it in `progress.md` → `## Notes` (e.g. `[{date}] Blocked. Previous status: ready`), then:
- `progress.md` → set `**blocked_by:** {reason}`
- `spec.md` → set `**Current status:** \`blocked\`` and `**Blocked by:** {reason}`
Sync checkboxes in `stories.md` (checkboxes don't change — blocked is a transient overlay).

To unblock: clear both `**blocked_by:**` in `progress.md` and `**Blocked by:**` in `spec.md` (restore to `—`), restore the previous status (read from `## Notes` in `progress.md` — it was recorded when blocking happened), sync everywhere.

---

## Code Generation Rules

- Before starting implementation, check `spec.md → ## Technical Details` for referenced files (Swagger specs, mockups, API contracts, design docs in `artifacts/`). Read them — they define the contract your code must satisfy.
- Generate complete, compilable code — not pseudocode or stubs.
- Follow the tech stack in `docs/tech.spec.md`.
- Respect existing patterns in the codebase.
- Write or update test files before or alongside implementation files.
- Record all created file paths in `progress.md` Artifacts section.

---

## User Communication

Read `references/communication.md` for exact message formats: status transitions, review questions, task list confirmation, tests_passing summary, interview.md update format, and tests.md results format.
