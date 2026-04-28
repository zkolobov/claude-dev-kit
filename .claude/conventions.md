# Shared Conventions

## Story vs Epic

**Epic** — a named placeholder for future work. Just a row in `stories.md`, no folder.
Use when you know the area but don't have enough detail to write a spec yet.
Epics are elaborated into full stories when you're ready to work on them.

**Story** — a fully elaborated work item with a folder, spec, interview, tasks, tests, progress.

## Status → Checkbox Mapping (stories.md)

| Status | Spec | Interview | Tests Defined | In Progress | Tests Passing | Done |
|---|---|---|---|---|---|---|
| `epic` | — | — | — | — | — | — |
| `draft` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| `review` | ☐ | ☐ | ☐ | ☐ | ☐ | ☐ |
| `ready` | ☑ | ☑ | ☐ | ☐ | ☐ | ☐ |
| `tests_defined` | ☑ | ☑ | ☑ | ☐ | ☐ | ☐ |
| `in_progress` | ☑ | ☑ | ☑ | ☑ | ☐ | ☐ |
| `tests_passing` | ☑ | ☑ | ☑ | ☑ | ☑ | ☐ |
| `done` | ☑ | ☑ | ☑ | ☑ | ☑ | ☑ |

`epic` rows use `—` in all checkbox columns (no folder exists yet).
`blocked` does not change checkboxes — it is a transient state layered on top.
`draft` and `review` share the same checkbox pattern (all ☐). To distinguish them, read `progress.md → **Status:**` — that field is the canonical source for these two statuses.
For discovery-phase stories: Tests Defined and Tests Passing always show `N/A`.

## Status Transition Rules

```
draft   → review        : open questions found
draft   → ready         : no open questions
review  → ready         : all questions answered
ready   → tests_defined : test plan written
tests_defined → in_progress : implementation started
in_progress → tests_passing : all tests pass
tests_passing → done    : code review approved
any     → blocked       : blocker arises
blocked → (restore)     : blocker resolved — restore the status that was active before blocking
                          (read progress.md to find what it was)
```

Every transition updates both `spec.md` (`**Current status:**`) and `progress.md` (`**Status:**`),
then syncs checkboxes in `docs/stories.md`.

## Story Folder Lookup

Given an ID like `0008`: use glob `docs/stories/0008-*/` to find the folder.
Do not re-derive the slug from the story name — the actual folder may differ.

## Test Commands by Stack

| Platform | Command |
|---|---|
| .NET | `dotnet test --logger trx` |
| Node.js | `jest --json` or `npm test` |
| Python | `pytest -v --tb=short` |
| Go | `go test ./...` |
| Flutter | `flutter test` |

## Test Result Markers

| Marker | Meaning |
|---|---|
| ⬜ | Not yet run |
| ✅ | Passed |
| ❌ | Failed |
| ⚠️ | Skipped / inconclusive |

---

## Task Decomposition Rules

A task is atomic when it produces **one deployable unit** and ideally maps to **one PR**.
Tasks are defined after the interview is complete — tech choices revealed in the interview
determine what tasks are needed.

### Layers and order

Decompose a story by layer, in this order (skip layers that don't apply):

| Layer | Type tag | Examples |
|---|---|---|
| Package / infra setup | `infra` | Install IdentityServer, configure DI container, add env config |
| Data model + context | `data` | Add entity, configure DbContext, write migration, seed data |
| Business logic | `backend` | Implement service, repository, domain rule (one class per task) |
| API layer | `api` | Add controller, define DTOs, wire route (one controller per task) |
| UI design | `design` | Wireframe, mockup, UX flow (before implementation) |
| UI implementation | `frontend` | Component, page, form (one component per task) |
| Tests | `test` | Unit test class OR integration test suite (always a separate task) |

### Decomposition rules

- **One task = one class / one endpoint / one migration / one component.** If you say "implement X and Y" — that's two tasks.
- **Tests are always separate** — never bundle `UserServiceTests` into the "Implement UserService" task.
- **Design before implementation** — if UI is involved, design task comes first.
- **Infra/setup first** — if a new package or configuration is needed, it's its own task.
- **If the task list exceeds 8 items** — the story is likely too large. Raise this with the user, explain what you'd split it into, and ask whether to proceed as-is or restructure.

### Task status markers

| Marker | Status |
|---|---|
| ⬜ | Pending |
| 🔄 | In progress |
| ✅ | Done |
| ⛔ | Blocked |

---

## Status Icons (for display)

| Status | Icon |
|---|---|
| `draft` | 📝 |
| `review` | 💬 |
| `ready` / `tests_defined` | 🟡 |
| `in_progress` | 🔄 |
| `tests_passing` | 🟢 |
| `done` | ☑️ |
| `blocked` | ⛔ |
| `epic` | ⚪ |
