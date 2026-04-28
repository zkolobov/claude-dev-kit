# do-it: User Communication Templates

## Status Transition

One line, always shown when status changes:

```
Status: draft → review
```

## Review Question

One question at a time — never dump all questions at once. Wait for the answer before showing the next one.

```
Q2: What authentication library should be used?
> 
```

## Task List Confirmation

Show the table, then ask. Wait for explicit `y` before proceeding.

```
| # | Task | Type |
|---|------|------|
| T01 | Add UserEntity + EF Core migration | data |
| T02 | Implement UserService | backend |
| T03 | Add POST /api/users controller | api |
| T04 | Write integration tests | test |

Proceed with these tasks? (y/n)
```

## Tests Passing Summary

Shown when all tests pass, before asking the user to approve `done`.

```
Story #0008 — User Registration
Built: UserEntity migration, UserService, POST /api/users controller
Tests: ✅ 6 / 6 passing

Ready to mark done? (y/n)
```

---

## File Update Formats

### interview.md — closing a question

```markdown
### Q{N}: {Question}
**Status:** ✅ closed
**Answer:** {recorded answer}
```

Update `## Question Status` open/closed counters after each change.

### tests.md — recording results

```markdown
## Results
**Last run:** {date}
**Runner output source:** `dotnet test` / `jest --json` / `pytest -v`
**Summary:** ✅ 7 / 7 passing

### Failure Log

#### {TestName}
**Error:** {error message or stack trace excerpt}
**File:** {test file path}:{line}
```
