# State updates this turn

Per PHASE-LOOP coach rule 8 (lock-impl) the response is a **pause-and-warn**, not a state transition. The user has not signaled progress, has not said "give up on p2-t3", and has not provided a substantiated impl-bug claim. Therefore most state stays put.

## `progress.json`

| Field | Before | After | Reason |
|---|---|---|---|
| `phase` | `2` | `2` | unchanged |
| `mode` | `"coach"` | `"coach"` | unchanged — rule 8 explicitly does not switch modes |
| `phases[1].todos[<p2-t3>].status` | `in_progress` (or `todo`) | unchanged | user did **not** mark it done; the diff is not a progress signal under rule 10(a/b/c) — no done-claim, the only commit-candidate is on a locked impl region (and the user hasn't even committed), and we are not running tests proactively |
| `phases[1].todos[<p2-t3>].user_writes` | `"test"` | `"test"` | unchanged |
| `phases[1].todos[<p2-t3>].practice` | `true` | `true` | unchanged — no give-up |
| `phases[1].todos[<p2-t3>].gave_up` | `false` | `false` | unchanged — no give-up |
| `turns_since_last_progress` | `n` | `n + 1` | rule 10 increment: no progress signal this turn (no "done" claim, no new commit on phase files, no test transition we observed) |
| `phases[1].last_seen_commit` | unchanged | unchanged | the user pasted an uncommitted diff; nothing to update |

## Events appended to `progress.json.events`

```json
{
  "ts": "<now>",
  "event": "lock_impl_violation_warned",
  "phase": 2,
  "todo_id": "p2-t3",
  "file": "internal/shortener/validate.go",
  "note": "User edited LLM-locked impl to make test stub pass; coach paused per PHASE-LOOP rule 8 and asked for revert or substantiated impl-bug claim."
}
```

## `notes/phase-2-<project-slug>.md` — append

Per SKILL.md interaction-contract step 5 ("Append to `notes/` after substantive coach exchanges"):

```
## p2-t3 — lock-impl warning

- User question: implicit ("I changed validate.go so my tests would pass — now they work!")
- Concept: Test-first discipline; tests as spec, not as a knob to twist until green.
- Explanation:
  - For `user_writes: test` TODOs the impl is LLM-authored and locked.
  - Weakening the impl to satisfy a stub test deletes the behavior under test, not verifies it.
  - The test stub fails by design until the user writes real assertions against the original behavior.
  - Legit escape: argue the impl is wrong with evidence; on confirm, give-up path produces fix+solve commits with `gave_up: true`.
  - Idiomatic Go path forward: table-driven tests over `Validate` covering http, https, non-http(s) scheme, empty, garbage.
- Doc URL: (cite Go testing docs via find-docs/context7 next turn when the user starts writing the test — not pre-emptively this turn)
```

## What does NOT change

- `plan.md` — untouched (post-phase-0 invariant).
- `spec.md` — untouched.
- No commits authored by the LLM this turn. The user's diff is uncommitted; if they revert per the response, nothing was ever committed against the locked region. If they refuse and commit anyway, next turn re-apply rule 8 (still no give-up unless they substantiate).

## Branches for next turn

- User reverts and starts on the test → reset `turns_since_last_progress` to 0 on first progress signal (done-claim or new commit on `validate_test.go`).
- User claims the impl is genuinely wrong with a real reason → enter give-up-with-fix path (rule 8): two commits `fix(phase 2, p2-t3): impl` then `solve(phase 2, p2-t3): test`; set `gave_up: true`, `practice: false`, `status: done`.
- User just pushes back ("come on, let me edit the impl") → hold the line per SKILL.md "On pushback"; do not relax the rule.
