# claude-dev-kit

AI-driven development framework for Claude Code. Tracks stories, manages backlog, writes and runs code — all through a consistent skill-based workflow.

## What it does

Instead of asking Claude to "just build X", you run structured commands that guide it through analysis, planning, implementation, and testing — one story at a time, with full state preserved between sessions.

---

## Prerequisites

- [Claude Code](https://claude.ai/code) with skills support
- The `.claude/` folder from this repo copied into your project

---

## Setup

```bash
# Copy the framework into your project
cp -r claude-dev-kit/.claude your-project/.claude
cp claude-dev-kit/CLAUDE.md your-project/CLAUDE.md
```

Then open Claude Code in your project and run:

```
init-project "Your project description here"
```

Claude scaffolds the docs folder and immediately runs `discovery` — you get your first stories and epics in one shot.

No description yet? Run `init-project` alone, then `discovery "description"` when ready.

---

## Workflow

```
init-project → discovery → do-it → (repeat: discovery → do-it → ...)
```

### 1. `init-project` — run once

Scaffolds the `docs/` folder with all required files.

```
init-project
```

Creates:
```
docs/
  stories.md       ← story registry
  tech.spec.md     ← tech stack decisions
  discovery.md     ← analysis results
.claude/
  conventions.md   ← shared rules for all skills
  skills/          ← skill definitions
```

---

### 2. `discovery` — analyze and plan

Reads your project description, assesses the tech stack, identifies what's ready to build now vs. what can wait, and proposes stories and epics. Always shows recommendations before creating anything.

```
discovery
```

Claude will output something like:

```
## Discovery analysis — 2026-04-27

### Ready to create now (full stories)
| Area                  | Phase       | Rationale                        |
|-----------------------|-------------|----------------------------------|
| Define tech stack     | discovery   | Stack decisions need recording   |
| Domain model design   | discovery   | Core entities are clear          |
| User registration API | mvp-backend | No open questions                |

### Save as epics for later
| Area                | Phase      | Why not now                      |
|---------------------|------------|----------------------------------|
| Payment integration | mvp-backend | No payment provider chosen yet  |
| Admin dashboard     | post-mvp   | Not needed for launch            |

Create these stories and epics? You can adjust before I proceed.
```

You confirm, Claude creates everything.

**Targeted mode** — focus on one area:
```
discovery "authentication"
```

**Re-run** periodically as work progresses to identify new gaps:
```
discovery
```

---

### 3. `do-it` — execute work

The main command. Without arguments, shows a menu of everything available:

```
do-it
```

```
## What would you like to work on?

🔄 In progress
  #0003 Domain Model Design (discovery)

🟡 Ready to start
  #0001 Define Tech Stack (discovery)
  #0004 User Registration API (mvp-backend)

⚪ Epics (not yet elaborated)
  #0005 Payment Integration (mvp-backend)
  #0006 Admin Dashboard (post-mvp)

Type a story number to work on it, type "epic #ID" to elaborate an epic,
or type "discovery" to find new work.
```

**Work on a specific story:**
```
do-it #0004
```

Claude reads the story's current state and picks up exactly where it left off:

| Status | What Claude does |
|--------|-----------------|
| `draft` | Reviews interview questions, asks what's unclear |
| `review` | Walks through open questions, records answers |
| `ready` | Decomposes into atomic tasks, shows plan, writes test plan |
| `tests_defined` | Transitions to `in_progress`, generates code task by task |
| `in_progress` | Continues from the first pending task |
| `tests_passing` | Asks you to review, marks done on approval |

**Work on a whole phase:**
```
do-it mvp-backend
```

**Elaborate an epic into a full story:**
```
do-it epic #0005
```

**Block / unblock:**
```
do-it block #0004 "waiting for payment provider contract"
do-it unblock #0004
```

---

## Other commands

### `status` — read-only overview

```
status                  ← project overview
status mvp-backend      ← phase detail
status #0004            ← specific story
```

Example output:
```
## Project Status — 2026-04-27

| Phase       | Total | Done | In Progress | Ready | Blocked | Draft/Review | Epics |
|-------------|-------|------|-------------|-------|---------|--------------|-------|
| discovery   | 3     | 1    | 1           | 1     | 0       | 0            | 0     |
| mvp-backend | 5     | 0    | 0           | 1     | 0       | 1            | 3     |

### 🔄 In Progress
- #0003 Domain Model Design (discovery) — last updated 2026-04-27

### ✅ Ready to start
- #0001 Define Tech Stack (discovery)
```

### `create-story` — add a story manually

```
create-story mvp-backend "JWT Authentication"
```

### Adding an epic

Epics are lightweight placeholders — just a row in `stories.md`, no folder. Use them when you know an area matters but aren't ready to spec it yet.

**Via discovery (recommended):**
```
discovery "2 factor auth"
```
Discovery analyzes the area, checks for duplicates, picks the right phase, and proposes an epic row. You confirm before anything is written.

**Manually** — add a row directly to the relevant phase table in `docs/stories.md`:
```
| 0016 | 2 Factor Auth | TOTP/SMS second factor at login | — | — | — | — | — | — |
```
`—` in all checkbox columns marks it as an epic. Use the next available ID.

When you're ready to work on an epic:
```
do-it epic #0016
```

### `update-story` — edit a story's spec

```
update-story #0004
```

Lets you change name, description, acceptance criteria, technical details, phase, or interview questions. Refuses to edit `done` stories. Requires explicit confirmation for `in_progress` stories.

---

## Story lifecycle

```
draft → review → ready → tests_defined → in_progress → tests_passing → done
                                                   ↑
                                    blocked can overlay any status
```

Each story has its own folder with five files:

```
docs/stories/0004-user-registration/
  spec.md        ← description, acceptance criteria, status
  interview.md   ← Q&A that shapes implementation choices
  tasks.md       ← atomic task breakdown (1 task = 1 PR)
  tests.md       ← test plan + live results
  progress.md    ← notes to resume in a new session
```

### Tasks are atomic

A task = one class, one endpoint, one migration, one component. Each maps to one PR.

Example task breakdown for "User Registration":

| # | Task | Type | Status |
|---|------|------|--------|
| T01 | Install IdentityServer NuGet package | infra | ⬜ |
| T02 | Add UserEntity + EF Core migration | data | ⬜ |
| T03 | Implement UserService (register + hash pwd) | backend | ⬜ |
| T04 | Add POST /api/users controller + DTOs | api | ⬜ |
| T05 | Write UserServiceTests | test | ⬜ |
| T06 | Write integration tests for POST /api/users | test | ⬜ |

### Epics are lightweight

An epic is just a row in `stories.md` — no folder, no files. Use it when you know an area matters but aren't ready to spec it yet. Elaborate when the time comes:

```
do-it epic #0005
```

---

## Tech stack

Defaults to `.NET 8+ / Azure / MS SQL Server / Angular / xUnit + Moq`.

Override via `discovery` — all decisions are recorded in `docs/tech.spec.md` and picked up automatically when tasks are generated. Specific library choices (e.g. "use MediatR", "use FluentValidation") live in the task descriptions, not in a global config. New decisions made during task decomposition are written back to `tech.spec.md` automatically.

---

## Example session

```
# Day 1 — set up and plan
init-project
discovery

# Claude proposes 3 stories + 4 epics. You confirm.

do-it
# Claude shows menu. You pick #0001 (Define Tech Stack)

do-it #0001
# Status: draft
# Claude reviews interview questions, finds 2 open → transitions to review
# "What cloud provider are you targeting?"
# You answer. Claude records it, closes the question.
# All questions closed → transitions to ready
# Claude decomposes tasks, shows list, you confirm
# Claude writes test plan → transitions to tests_defined

# Day 2 — implement
do-it #0001
# Transitions to in_progress, generates code for each task one by one
# Runs tests → all pass → transitions to tests_passing
# You review → "looks good" → done ✅

discovery
# Claude sees #0001 done, spots 2 newly unblocked areas
# Proposes 1 new story + 1 new epic → you confirm

do-it
# New story is ready → pick it up and continue
```

---

## File structure reference

```
your-project/
├── CLAUDE.md                        ← loaded in every Claude session
├── docs/
│   ├── discovery.md                 ← project analysis
│   ├── tech.spec.md                 ← confirmed tech stack
│   ├── stories.md                   ← story registry (one table per phase)
│   └── stories/
│       └── {id}-{slug}/
│           ├── spec.md
│           ├── interview.md
│           ├── tasks.md
│           ├── tests.md
│           ├── progress.md
│           ├── artifacts/           ← generated files, diagrams
│           └── test-data/           ← seed/fixture data (if applicable)
└── .claude/
    ├── conventions.md               ← shared rules (status mapping, icons, task rules)
    └── skills/
        ├── init-project/
        ├── discovery/
        ├── create-story/
        ├── do-it/
        ├── status/
        └── update-story/
```
