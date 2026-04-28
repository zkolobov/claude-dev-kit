# claude-dev-kit

AI-driven development framework.

## Typical workflow

```
init-project → discovery → do-it → (periodic) discovery → do-it → ...
```

1. **`init-project`** — once, to scaffold the `docs/` folder.
2. **`discovery`** — analyzes the project, creates a few full stories for immediate work and lightweight epics for the rest. Never stuffs the backlog — just enough to get started.
3. **`do-it`** — picks up the next story. Shows a menu: in_progress → ready → epics. User chooses what to work on. If the backlog is empty, suggests running `discovery` again.
4. **Repeat** — run `discovery` periodically to identify new gaps as completed work reveals what's next.

## Commands

| Command | What it does |
|---|---|
| `init-project` | Scaffold `docs/` structure and empty templates. Run once per project. |
| `discovery` | Analyze project, propose stories/epics, confirm with user before creating anything. |
| `create-story <phase> "<name>"` | Create a new story with all files. |
| `do-it [#ID \| phase]` | Execute next step. Without args, shows a menu of available work. |
| `do-it epic #ID` | Elaborate an epic into a full story, then start working on it. |
| `status [#ID \| phase]` | Show project status overview (read-only). |

## File Layout

```
docs/
  discovery.md          # Discovery analysis
  tech.spec.md          # Confirmed tech stack
  stories.md            # Story registry (tables per phase)
  stories/
    {id}-{slug}/
      spec.md           # Specification and acceptance criteria
      interview.md      # Q&A — open/closed questions
      tasks.md          # Task breakdown (1 task = 1 PR) — defined after interview
      tests.md          # Test plan + live results
      progress.md       # Status, blockers, notes to resume
      artifacts/        # Input/output files
      test-data/        # Seed/fixture data (if applicable)
.claude/conventions.md  # Shared status mapping, transitions, icons
.claude/skills/         # Skill definitions
```

## Story ID Format

4-digit zero-padded global counter: `0001`, `0002`, ...
Folder name: `docs/stories/{id}-{slug}/`

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

`epic` — placeholder row only, no folder. Elaborated into a full story via `do-it epic #ID`.
`blocked` — transient overlay, does not change checkboxes.
Discovery-phase stories: Tests Defined and Tests Passing always show `N/A`.

## Default Tech Stack

.NET 8+ / Azure / MS SQL Server / Angular / xUnit + Moq

Stack is overridable via `discovery`. All decisions recorded in `docs/tech.spec.md`.
