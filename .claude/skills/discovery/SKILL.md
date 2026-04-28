---
name: discovery
description: Analyze a software project, assess tech stack fit, identify what's ready to build now vs. what's future work, and create stories or epics accordingly. Use this skill whenever the user says "discovery", "analyze project", "assess tech stack", "what should we build next", "re-run discovery", "what are we missing", "plan the next phase", or wants to identify gaps and plan the next wave of work. Run after init-project on a fresh project, and periodically as work progresses to spot what's missing. Never auto-runs — always shows recommendations first and asks for confirmation.
---

# discovery

Analyze the project, identify what's ready to build now vs. what can wait, and create the right artifacts (full stories or lightweight epics) based on that distinction.

Read `.claude/conventions.md` for the Epic vs Story distinction, status mapping, and icons.

---

## When to run

- **Fresh project** — after `init-project`, before any stories exist.
- **Periodic re-run** — after a phase completes or when the backlog runs dry. Identifies what's done, what's missing, and what's next.
- **Targeted** — `discovery "auth"` focuses on one area.

---

## Targeted mode — `discovery "payments"`

If an argument is given, note it as the **focus area** before starting Phase 1. Phases 1–3 run as normal (you still need the full picture to do gap analysis). In Phase 4, filter the gap analysis to only the focus area — skip unrelated areas, propose only stories and epics relevant to what was asked. Phase 5 updates only the sections of `docs/discovery.md` touched by this run.

---

## Phase 1 — Read existing state

Before doing any analysis, read what already exists:

1. Read `docs/discovery.md` (if it exists) — prior analysis, stack decisions, open questions.
2. Read `docs/tech.spec.md` (if it exists) — confirmed stack.
3. Read `docs/stories.md` — all existing stories and epics across all phases.

Categorize what you find:
- What phases exist?
- What stories are `done` or `in_progress`?
- What areas have epics but no stories?
- What important areas have nothing at all?

On a fresh project these files don't exist yet — skip the categorization and proceed to Phase 2.

---

## Phase 2 — Analyze the project

Read the project description (from `docs/discovery.md` → `## Source Prompt`, or ask the user if missing).

Extract:

| Parameter | Description |
|---|---|
| **Product type** | Web, mobile, API, CLI, desktop, embedded, ... |
| **Target platform** | Browser, iOS/Android, Windows/Mac, cloud-only, ... |
| **Scale** | MVP / startup / enterprise / internal tool |
| **Audience** | B2C, B2B, internal, IoT, ... |
| **Key constraints** | Budget, deadlines, team size, infrastructure |
| **Integrations** | Third-party APIs, payments, social auth, hardware |
| **Non-functional requirements** | Performance, security, availability, compliance |

---

## Phase 3 — Stack assessment

Skip this phase if `docs/tech.spec.md` already has content in the `## Chosen Stack` table (at least one row with a non-empty Technology column). The stack is a living document — it grows as decisions are made, so there's no "confirmed" flag needed.

Score each default component: **high / medium / low fit**.

| Component | Default |
|---|---|
| Backend | .NET 8+ (C#) |
| Cloud | Azure |
| Database | MS SQL Server |
| Frontend | Angular |
| Unit Tests | xUnit + Moq |
| Integration Tests | xUnit + WebApplicationFactory |

If any component scores **low**, propose alternatives using this block:

```markdown
## ⚠️ Recommendation: replace {component}

### Reason
{Why the default doesn't fit — be specific.}

### Options

#### Option A: {Technology} (recommended)
**Pros:** ...
**Cons:** ...

#### Option B: {Technology}
**Pros:** ...
**Cons:** ...
```

**Stop and wait for the user to confirm the stack before proceeding.**

Once confirmed, record in `docs/tech.spec.md` and `docs/discovery.md`.

---

## Phase 4 — Gap analysis

Look at the full picture: what the project needs vs. what exists in `docs/stories.md`.

Group coverage areas by readiness:

**Ready now** — you have enough context to write a spec immediately:
- The "what" is clear (user described it, or it's an obvious building block)
- Dependencies are known or already done
- Tech choices are settled

**Not yet ready** — interesting area but not enough detail to spec it:
- Requirements unclear or not yet discussed
- Depends on something not finished
- Future phase / "would be nice to have"

For discovery-specific stories, always treat them as ready-now on a fresh project.

---

## Phase 5 — Show recommendations and ask

Present your analysis to the user before creating anything:

```markdown
## Discovery analysis — {today's date}

### Current state
{Summary of what's done, what's in progress, what's pending — if re-running.
On a fresh project: "No stories exist yet."}

### Ready to create now (full stories)

| Area | Phase | Rationale |
|------|-------|-----------|
| Define tech stack | discovery | Stack just confirmed, decisions need recording |
| Domain model design | discovery | Core entities are clear from the description |
| User registration API | mvp-backend | Straightforward, no open questions |

### Save as epics for later

| Area | Phase | Why not now |
|------|-------|-------------|
| Payment integration | mvp-backend | No payment provider chosen yet |
| Admin dashboard | post-mvp | Not needed for launch |
| Mobile app | future | Out of scope for now |

### Open questions
{Any gaps or decisions still needed — leave blank if none}

---

Create these stories and epics? You can adjust the list before I proceed.
```

Wait for the user to confirm, adjust, or cancel. If they want to adjust — modify the proposed list (add, remove, or reclassify items), show the updated version, and ask again before proceeding.

---

## Phase 6 — Create artifacts

After user confirmation, create the agreed items in this order:

1. **Full stories** — invoke `create-story` for each one.
   Use the `discovery` phase for analysis stories, or the appropriate dev phase.

2. **Epics** — add a single row to the relevant phase table in `docs/stories.md`.
   Epics have **no folder** — just a row with `—` in all checkbox columns.

   Epic row format:
   ```
   | {ID} | {Epic Name} | {1-2 sentence description} | — | — | — | — | — | — |
   ```

   Assign epic IDs the same way as story IDs (read current highest, increment).

3. Update `docs/discovery.md` — write into the specific sections that exist in the template:
   - `## Date` → today's date
   - `## Analysis` subsections → Phase 2 output (product type, platform, scale, constraints, etc.)
   - `## Default Stack Fit Assessment` → Phase 3 fit scores per component (skip if Phase 3 was skipped)
   - `## Proposed Alternatives` → any alternatives surfaced in Phase 3
   - `## Final Decision` → confirmed stack, rejected alternatives, open questions that remain
   - `## Open Questions` → anything still unresolved after this run

4. After all artifacts are created, append suggested actions. Include one `/do-it #ID` action per created full story (not epics), plus `/do-it` to see the full backlog menu. Order: discovery-phase stories first, then by phase. Example for 3 created stories:

```
<suggested-actions>
<action>/do-it #0001</action>
<action>/do-it #0002</action>
<action>/do-it #0003</action>
<action>/do-it</action>
</suggested-actions>
```

Use the actual IDs assigned during this run. If only epics were created (no full stories), just suggest `/do-it`.

---

## Discovery-specific stories

When creating discovery-phase stories, use the `discovery` phase and mark `Tests Defined` / `Tests Passing` as `N/A` in `stories.md` (these are spec/design stories, not code).

Standard discovery stories to consider on a fresh project:

| Slug | Purpose | Create when |
|------|---------|-------------|
| `define-tech-stack` | Justify and document technology choices | Always |
| `basic-design-requirements` | UX principles, branding, accessibility | If product has UI |
| `domain-model-design` | Entities, relationships, bounded contexts | Always |
| `api-contract-definition` | REST/GraphQL contracts, versioning | If project has an API |
| `infrastructure-setup` | Cloud resources, CI/CD, environments | If cloud/infra needed |
| `security-model` | Auth strategy, roles, data protection | If auth/security required |

On a re-run, skip any that already exist.

---

## Behavior rules

- Always read existing state first — never assume a clean slate.
- Never create stories or epics without explicit user confirmation.
- Prefer fewer full stories over a stuffed backlog. When in doubt, make it an epic.
- Use answers obvious from the prompt — don't ask redundant questions.
- Mark assumptions as `[assumption]` when you're not sure.
- On a re-run, surface what's finished and acknowledge progress before proposing new work.

## Alternative test stacks by platform

| Platform | Unit | Integration |
|---|---|---|
| .NET | xUnit + Moq | WebApplicationFactory + TestContainers |
| Node.js | Jest | Supertest + testcontainers-node |
| Python | pytest + unittest.mock | pytest + httpx |
| React Native | Jest | React Native Testing Library |
| Flutter | flutter_test | integration_test |
| Go | testing + testify | net/http/httptest |
