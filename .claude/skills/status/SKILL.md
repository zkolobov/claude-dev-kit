---
name: status
description: Show the current state of the project, a phase, or a specific story — read-only, no changes made. Use this skill whenever the user says "status", "what's the status", "show progress", "where are we", "what's blocked", "show me story #ID", or wants an overview without doing any work. Also use it when the user asks what's going on, what's done, what's left, or just seems to want a snapshot before deciding what to do next. Use for morning check-ins, project reviews, or any time the user wants to see state without touching it. Prefer this over do-it when there's no intent to act.
---

# status

Read-only project status overview. Never modifies any files.

Read `.claude/conventions.md` for status→checkbox mapping, story folder lookup, and status icons.

---

## Mode 1: Project overview — `status`

Read `docs/stories.md`. Build a phase summary table:

```markdown
## Project Status — {today's date}

| Phase | Total | Done | In Progress | Ready | Blocked | Review | Draft | Epics |
|-------|-------|------|-------------|-------|---------|--------|-------|-------|
| discovery | 5 | 3 | 0 | 1 | 0 | 1 | 0 | 0 |
| mvp-backend | 8 | 0 | 1 | 2 | 1 | 0 | 2 | 2 |
| mvp-frontend | 3 | 0 | 0 | 0 | 0 | 0 | 0 | 3 |
```

**Epics** are rows with `—` in all checkbox columns — they have no folder and aren't yet elaborated.
Count them separately; don't include them in Done/In Progress/Ready/Blocked totals.
**Review vs Draft**: both show all ☐ in stories.md — read `progress.md → **Status:**` to tell them apart.

Then list active items:

```markdown
### 🔄 In Progress
- #0006 Infrastructure Setup (mvp-backend) — last updated {date}

### ⛔ Blocked
- #0009 JWT Authentication (mvp-backend) — blocked by: {reason}

### 💬 In review (open questions to answer)
- #0003 Security Model (discovery)

### 📝 Draft (not yet reviewed)
- #0004 Domain Model (discovery)

### 🟡 Ready to start
- #0007 Database Schema (mvp-backend)
- #0008 User Registration (mvp-backend)

### ⚪ Epics (not yet started)
- #0012 Payment Integration (mvp-backend)
- #0015 Admin Dashboard (post-mvp)
```

To determine each story's status: infer from the checkbox pattern in `stories.md`
using the mapping in `.claude/conventions.md`. For stories with all ☐ (`draft` or `review`),
read `progress.md → **Status:**` — that is the canonical source. Do the same for `blocked` stories.
Rows with `—` in all columns → `epic` (no folder, no progress.md to read).

---

## Mode 2: Phase detail — `status mvp-backend`

Show the full story list for the phase with status and last-updated date:

```markdown
## mvp-backend — Status

| ID | Story | Status | Last Updated |
|----|-------|--------|--------------|
| 0006 | Infrastructure Setup | 🔄 in_progress | 2026-04-25 |
| 0007 | Database Schema | 🟡 ready | 2026-04-20 |
| 0008 | User Registration | 📝 draft | 2026-04-18 |
| 0009 | JWT Authentication | ⛔ blocked | 2026-04-22 |

**Blocked:** #0009 — waiting for: {blocker description}
**Suggested next:** #0006 (in_progress — continue it) or #0007 (ready — start it next)
```

Status icons (see `.claude/conventions.md` for full table):
- ⚪ `epic` (no folder, all `—`)
- 📝 `draft`
- 💬 `review`
- 🟡 `ready` / `tests_defined`
- 🔄 `in_progress`
- 🟢 `tests_passing`
- ☑️ `done`
- ⛔ `blocked`

For epics in Mode 2, show `⚪ epic` as the status and `—` as Last Updated (no folder to read).

---

## Mode 3: Specific story — `status #0008`

Strip any leading `#` from the ID before looking up. Check `docs/stories.md` for the row. If it has `—` in all checkbox columns, it's an epic — no folder exists yet. Show only what's in the row:

```markdown
## Epic #0012: Payment Integration
**Phase:** mvp-backend
**Status:** ⚪ epic (not yet elaborated)
**Description:** {description from stories.md row}

Use `do-it epic #0012` to elaborate it into a full story.
```

Otherwise, read all five story files (spec, interview, tasks, tests, progress). Show a full summary. If `tasks.md` has no rows yet (story is still in `draft`/`review`), omit the Tasks section entirely.

```markdown
## Story #0008: User Registration

**Status:** `in_progress`
**Phase:** mvp-backend
**Last updated:** 2026-04-20

### Acceptance Criteria
- [x] User can register with email + password
- [ ] Duplicate email returns 409
- [ ] Password is hashed before storage

### Interview
- Open: 0
- Closed: 3

### Tasks
| # | Task | Type | Status | PR |
|---|------|------|--------|----|
| T01 | Install IdentityServer package | infra | ✅ | #12 |
| T02 | Configure DbContext | data | 🔄 | — |
| T03 | Implement AuthService | backend | ⬜ | — |
| T04 | Write integration tests | test | ⬜ | — |

**Progress:** 1 / 4 done

### Tests

{If N/A — e.g. discovery phase or pure UI:}
N/A — {reason from tests.md Applicability section}

{If tests have not been run yet (Last run: "—"):}
**Plan:** 3 unit tests, 4 integration tests defined — not yet run.

{If tests have been run, show the full results table from tests.md:}
**Last run:** 2026-04-20 | **Summary:** ✅ 5 / 6 passing

| # | Test Name | Description | Result |
|---|-----------|-------------|--------|
| 1 | Should_Register_Valid_User | Happy path registration | ✅ |
| 2 | Should_Reject_Duplicate_Email | 409 on existing email | ❌ |

### Blockers
None

### Notes
{excerpt from progress.md Notes section}

---

### Actions

{Show only the lines relevant to the story's current status:}

**draft / review** — open questions to answer:
- `do-it #0008` — answer open questions, move to ready
- `update-story #0008` — edit spec or acceptance criteria

**ready / tests_defined** — ready to implement:
- `do-it #0008` — implement this story (write tests, write code, run suite)
- `update-story #0008` — adjust spec before starting

**in_progress** — implementation underway:
- `do-it #0008` — continue implementation
- `update-story #0008` — update spec mid-flight (you'll be asked to confirm)

**tests_passing** — review and close:
- `do-it #0008` — review and mark done
- `update-story #0008` — final spec correction before closing

**done:**
- `create-story {phase} "{name}"` — add a follow-up story

**blocked:**
- `do-it unblock #0008` — clear the blocker and restore previous status
- `update-story #0008` — update spec while waiting
```

For epics, append the action line to the epic summary block:

```markdown
**Next step:** `do-it epic #0012` — elaborate into a full story.
```

---

## Behavior Rules

- Never write to any file.
- In Mode 1 and 2 (overview): show state only — no action menu. The user is scanning, not deciding.
- In Mode 3 (specific story): always end with the **Actions** menu. The user has drilled into a story and likely wants to do something next — showing the exact commands saves them from remembering syntax.
- Read `progress.md` for stories where checkbox inference is ambiguous (e.g. all ☐ → could be draft or review).
