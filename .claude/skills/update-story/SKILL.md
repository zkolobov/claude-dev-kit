---
name: update-story
description: Edit an existing story's spec, acceptance criteria, description, phase, name, or interview questions — without changing its implementation status. Use this skill whenever the user says "update story", "edit story", "change story", "rename story", "move story to phase", "add acceptance criteria", "fix the spec", "story #ID is wrong", "that description is off", "add a requirement", or wants to correct or extend any part of a story's definition. If a story's content needs to change (not its status or tasks), this is the right skill. Do NOT use for status transitions (use do-it) or blocking/unblocking (use do-it block/unblock).
---

# update-story

Edit a story's definition — its name, description, acceptance criteria, technical details, phase, or interview questions. Never touches status, tasks, or test results; those are owned by `do-it`.

Read `.claude/conventions.md` for story folder lookup (glob pattern) and status→checkbox mapping.

---

## Step 1 — Locate the story

Strip any leading `#` from the ID (users often type `#0008`), then use glob `docs/stories/{ID}-*/` to find the folder.
Also read the story's row in `docs/stories.md` to confirm current name and phase.

If the glob returns no results, check the row in `docs/stories.md`: if all checkbox columns contain `—`, it's an epic (no folder exists yet). Tell the user: "This is an epic — use `do-it epic #ID` to elaborate it into a full story first, then you can edit it." If the row exists but has normal checkboxes and the folder is still missing, something is wrong — report it to the user.

---

## Step 2 — Read current state and apply guards

1. Read `spec.md` and `interview.md`. (Read `tasks.md`, `tests.md`, or `progress.md` only if the user is asking about something in those files.)

2. Check `**Current status:**` in `spec.md`:

   - **`done`** → refuse. A completed story is a historical record; editing it undermines the audit trail. Show the refusal message below and stop — do not proceed to Step 3.
   - **`in_progress` or `tests_passing`** → warn and require confirmation. Active work is in flight; a spec change can break alignment with code already written. Show the warning message below and wait for explicit `y` before continuing. If the user says `n` or hesitates, stop and suggest they finish the current work first.
   - **Any other status** (`draft`, `review`, `ready`, `tests_defined`, `blocked`) → proceed to Step 3 directly.

### Messages

**Refusal (`done`):**
```
Story #ID is marked done and cannot be edited.

If the spec was wrong, create a follow-up story to correct the behaviour.
If this was closed by mistake, use `do-it` to reopen it (revert to tests_passing).
```

**Warning (`in_progress` / `tests_passing`):**
```
⚠️  Story #ID is currently in progress.
Changing the spec mid-implementation may conflict with work already done.

Please confirm you want to proceed before I show the proposed edit. (y/n)
```

---

## Step 3 — Understand what to change

If the user was specific ("add criterion X", "rename to Y", "move to phase Z") → go directly to Step 4.

If the user was vague ("update story #0008") → show a brief current-state summary and ask:

```
Story #0008: User Registration (mvp-backend)
Status: ready

What would you like to change?
  1. Name / description
  2. Acceptance criteria (add, edit, remove)
  3. Technical details
  4. Phase / dependencies
  5. Interview questions (add or edit)
  6. Something else
```

---

## Step 4 — Propose the change

Always show the proposed edit before applying it. Keep it concise — just enough for the user to confirm.

**Examples:**

*Rename:*
```
Rename: "User Registration" → "User Registration via Email"
Update: spec.md title, stories.md row name.
Apply? (y/n)
```

*Add acceptance criterion:*
```
Add to acceptance criteria:
  - [ ] Email must be verified before login is allowed

Apply? (y/n)
```

*Move phase:*
```
Move #0008 from mvp-backend → mvp-v2
Update: spec.md phase, stories.md row (move to mvp-v2 table).
Apply? (y/n)
```

Always show the proposed change before applying it — even for trivial edits. The point is that the user can catch mistakes before they land, not just approve the intent.

---

## Step 5 — Apply the change

Make the edits. The scope depends on what changed:

### Name change
- `spec.md` — update the `# Story {ID}: {Name}` heading
- `progress.md` — update the `# Progress: {Name}` heading to match
- `interview.md` — update the `# Interview: {Name}` heading to match
- `tests.md` — update the `# Tests: {Name}` heading to match
- `docs/stories.md` — update the story name in the row
- Do **not** rename the folder — the slug is fixed at creation time to avoid breaking references

### Description change
- `spec.md` — update `## Description` section

### Acceptance criteria
- `spec.md` — add, edit, or remove items in `## Acceptance Criteria`
- Preserve existing checkbox states (`[x]` for done, `[ ]` for pending)

### Technical details, dependencies, or external references
- `spec.md` — update `## Technical Details` or `## Dependencies`
- This is also the right place to reference external files: design mockups, Swagger specs, API contracts, etc.
  - If the file is in `artifacts/`, note the relative path: `artifacts/payment-api.yaml`
  - If it's external (Figma, Confluence, shared drive), note the URL or path
  - Claude will read this reference when generating tasks at the `ready` stage

### Phase change
- `spec.md` — update `**Phase:**`
- `docs/stories.md` — move the row from the old phase table to the new phase table
  - If the target phase section doesn't exist yet, create it with a header and table
  - Preserve the row's checkbox state exactly as-is

### Interview questions
- `interview.md` — add a new question at the end (status: ❓ open), or edit an existing question's text
- Update `## Question Status` counters if you added an open question
- Do not close questions here — that's `do-it review`'s job

---

## Step 6 — Sync files after change

1. `spec.md` — update `**Updated:**` to today's date.
2. `progress.md` — update `**Last updated:**` and add a note to `## Notes`:

```
[{date}] Spec updated: {one-line summary of what changed}
```

---

## What this skill does NOT touch

- Story status (`spec.md **Current status:**`) — use `do-it`
- Tasks (`tasks.md`) — use `do-it`
- Test plans or results (`tests.md`) — use `do-it`
- Blocking or unblocking — use `do-it block` / `do-it unblock`
- Deleting a story — there's no delete; archive by moving to an `archived` phase if needed

If the user asks for one of these, point them to the right skill rather than refusing.
