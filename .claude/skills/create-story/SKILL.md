---
name: create-story
description: Create a new development story with all required files (spec, interview, tests, progress) and register it in stories.md. Use this skill whenever the user says "create-story", "add story", "new story", or provides a phase and story description and wants to start tracking it. Also invoked automatically by the discovery skill when elaborating an epic or creating a new story. Always use this skill when adding work items to the project — never create story files manually.
---

# create-story

Create a single story with all its files and register it in `docs/stories.md`.

Two modes:
- **New story** — assign a new ID, add a new row to `stories.md`.
- **Elaborate epic** — called by `do-it` when an epic already has a row. The caller passes the existing ID; skip ID assignment and do NOT add a new row (the existing epic row will be updated by the caller).

## Actions (in order)

### 1 — Assign ID

**New story:** Read `docs/stories.md`. Find the highest existing ID across all phases.
New ID = highest + 1, zero-padded to 4 digits. If no stories exist yet → `0001`.

**Elaborate epic:** The ID is provided by the caller. Use it as-is; skip this step.

### 2 — Generate Slug

Kebab-case English, max 5 words, derived from the description.
Example: "User Registration via email" → `user-registration`

### 3 — Create Folder

```
docs/stories/{id}-{slug}/
docs/stories/{id}-{slug}/artifacts/
```

Create `test-data/` subfolder only if the story involves data-driven testing or file processing
(CSV imports, seed data, mock API responses). Don't create it otherwise.

### 4 — Create `spec.md`

```markdown
# Story {ID}: {Name}

## Status
**Current status:** `draft`
**Phase:** `{phase}`
**Blocked by:** —
**Created:** {today's date}
**Updated:** {today's date}

## Description
{Full task description derived from the input}

## Acceptance Criteria
- [ ] {criterion 1}
- [ ] {criterion 2}

## Technical Details
{Architectural decisions, schema references, API contracts — fill what is known, leave blank if unknown}

## Dependencies
- Depends on: {story IDs or "—"}
- Blocks: {story IDs or "—"}
```

### 5 — Create `interview.md`

```markdown
# Interview: {Story Name}

## Question Status
- Open: {N}
- Closed: {N}

## Questions

### Q1: {question}
**Status:** ❓ open
**Answer:** —

### Q2: {question}
**Status:** ❓ open
**Answer:** —
```

Rules:
- Generate questions that are genuinely unclear from the prompt — tech choices, integration details, scope boundaries.
- If an answer is unambiguously clear from the prompt or project context, close the question immediately: set `**Status:** ✅ closed` and fill in `**Answer:**`.
- Include at least one question. If all are closeable from context, the story starts as `ready` (not `draft`) — set `**Current status:** \`ready\`` in `spec.md` and `**Status:** \`ready\`` in `progress.md`, and use the `ready` checkbox pattern in `stories.md`.

### 6 — Create `tasks.md`

Empty skeleton — populated by `do-it` after the interview is complete (at `ready` stage),
when tech choices are known and concrete tasks can be defined.

```markdown
# Tasks: {Story Name}

## Summary
**Total:** — | **Done:** 0 | **In Progress:** 0 | **Pending:** —

## Tasks

> Defined by `do-it` at the `ready` stage, after interview reveals tech choices.
> See `.claude/conventions.md` for task types and decomposition rules.

| # | Task | Type | Status | PR |
|---|------|------|--------|----|
```

### 7 — Create `progress.md`

`progress.md` is for context and resumability — not for tracking task completion.
Task progress lives in `tasks.md`. Keep progress.md lean.

```markdown
# Progress: {Story Name}

## Meta
**Status:** `draft`
**Last updated:** {today's date}
**Completed at:** —

## Blockers
**blocked_by:** —

## Artifacts

## Notes
Story created. Awaiting interview review and question resolution.
```

### 8 — Create `tests.md`

**If tests apply** (backend logic, services, API endpoints, data processing):

```markdown
# Tests: {Story Name}

## Applicability
- [ ] Unit tests
- [ ] Integration tests
- [ ] N/A — reason: ...

---

## Plan
> Written during `tests_defined` stage. Must be complete before any implementation starts.

### Unit Tests

#### {ClassName}Tests
| # | Test Name | Description | Result |
|---|-----------|-------------|--------|

### Integration Tests

#### Endpoint: {METHOD} {/path}
| # | Test Name | Description | Result |
|---|-----------|-------------|--------|

---

## Results
> Updated by the worker after running the test suite.

**Last run:** —
**Runner output source:** —
**Summary:** ⬜ 0 / 0 passing

### Failure Log
```

**If tests do NOT apply** (discovery phase, pure UI story, infra/DevOps task, migration without logic):

```markdown
# Tests: {Story Name}

## Applicability
- [ ] Unit tests
- [ ] Integration tests
- [x] N/A — reason: {e.g. "discovery phase story" / "pure UI, no business logic"}
```

### 9 — Register in `stories.md`

**Elaborate epic mode:** Skip this step. The epic row already exists and will be updated by the caller (`do-it`).

**New story mode:** Add a row to the phase table. If the phase section doesn't exist yet, create it with a header and table.

For backend / frontend stories:
```
| {ID} | {Story Name} | {1-2 sentence description} | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
```

For discovery-phase stories (tests N/A):
```
| {ID} | {Story Name} | {1-2 sentence description} | ☐ | ☐ | N/A | ☐ | N/A | ☐ |
```

Column order: `ID | Story | Description | Spec | Interview | Tests Defined | In Progress | Tests Passing | Done`

### 10 — Print Summary

```
✅ Story {ID} created: {Name}
   Phase: {phase}
   Folder: docs/stories/{id}-{slug}/
   Status: draft
   Open questions: {N}
```
