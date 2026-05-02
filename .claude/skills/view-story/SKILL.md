---
name: view-story
description: Display a formatted summary of a single story — its status, description, open interview questions, task progress, acceptance criteria, and dependencies — all in one readable view. Use this skill whenever the user says "view story", "show story", "what's story #N", "show me #N", "open story N", or references a story by ID (e.g. "#0005", "story 5"). Also use when the user asks "what does story N contain?" or wants to inspect or review a story before working on it.
---

# view-story

Render a complete, readable summary of a story from its five source files.

## Mode 1: Specific story — `view-story #0005`

### Step 1 — Locate the story

Strip any leading `#` from the ID. Zero-pad to 4 digits if needed (`5` → `0005`).

Use glob `docs/stories/{ID}-*/` to find the folder. The slug in the folder name may differ from what you'd generate — always use the actual folder.

If no folder matches, check `docs/stories.md` to confirm whether the story exists (it may be an epic with no folder). If it's an epic, say so and show its row from stories.md. If the ID doesn't exist at all, say so clearly.

### Step 2 — Read all five files in parallel

Read these files simultaneously:
- `spec.md` — status, phase, description, acceptance criteria, technical details, dependencies
- `interview.md` — open/closed question counts and question list
- `tasks.md` — task list with statuses
- `progress.md` — blockers, notes
- `tests.md` — applicability and test plan status

### Step 3 — Render the summary

Output the story view using this structure. Adapt sections based on what's actually in the files — skip empty sections rather than showing blank placeholders.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 {status icon}  #{ID} — {Story Name}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Phase: {phase}   Status: {status}   Updated: {date}
{If blocked: ⛔ Blocked by: {reason}}

## Description
{description — truncate at ~200 words if very long, indicate truncation}

## Acceptance Criteria
{render each criterion with its checkbox state: ✅ or ☐}

## Interview
{Open: N}  {Closed: M}
{If open questions exist, list them:}
  ❓ Q{n}: {question text}
{If all closed:}
  ✅ All questions resolved

## Tasks
{If tasks defined:}
  Total: N  |  ✅ Done: N  |  🔄 In progress: N  |  ⬜ Pending: N
  {list each task: status icon · task name · type}
{If no tasks yet:}
  Not yet decomposed (story must reach `ready` status first)

## Tests
{Applicability line from tests.md}
{If N/A: state reason}
{If applicable: whether plan is written or pending}

## Dependencies
  Depends on: {list or "—"}
  Blocks:     {list or "—"}

## Notes
{blockers from progress.md if any}
{last note from progress.md, truncated if long}
── Actions ─────────────────────────────
  do-it #{ID}          — work on this story
  update-story #{ID}   — edit spec or description
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

After rendering the summary, call the `AskUserQuestion` tool to offer next actions:

- If the story is **not** `done`: ask "What would you like to do next?" with options `/do-it #ID` (label "Work on story", description "Continue or start implementation") and `/update-story #ID` (label "Edit story", description "Update spec or description").
- If the story is `done`: ask with only `/update-story #ID` (label "Edit story", description "Update spec or description").

Use the actual story ID in the option labels/descriptions.

**Status icons** (from `.claude/conventions.md`):
- 📝 draft  |  💬 review  |  🟡 ready / tests_defined  |  🔄 in_progress  |  🟢 tests_passing  |  ☑️ done  |  ⛔ blocked

---

## Mode 2: No argument — `view-story`

Read `docs/stories.md` and show a compact list grouped by phase, with status icon and one-line description for each story. Epics shown with ⚪.

```
## Stories

### discovery
  📝 #0001 Define Tech Stack — document all tech choices and rejected alternatives
  📝 #0002 Domain Model Design — SQL schema, relationships, indexes
  ...

### mvp
  📝 #0004 Project Setup & Navigation Scaffold — Expo init, tabs, NativeWind, SQLite
  ...

### post-mvp
  ⚪ #0011 Reports Screen (epic)
  📝 #0014 User Account Creation
  ...

Type `view-story #ID` to see the full detail for any story.
```

To determine the true status of all-☐ stories (draft vs review), read `progress.md` for each. For large backlogs this may be slow — it's OK to show the checkbox pattern as the status without reading every progress file.

---

## Behavior notes

- Always read files in parallel (Step 2) — don't read one then the next.
- If a file is missing (e.g. story was partially created), note which file is absent and show what you have.
- Keep the output tight — this is a glance view, not a dump of raw file contents.
