# State updates — turn applied

## `progress.json` mutations

- `turns_since_last_progress`: incremented by 1 (no progress signal this turn — user's claim "now they work!" is a false-progress claim that the lock-impl rule explicitly rejects, so it does NOT reset the counter).
- `phases[1].todos[*]` (the p2-t3 entry): **no change** to `status`. Still `in_progress` (or `pending`, whichever it was). Specifically, do NOT mark `status: done` despite the user's claim — they did not complete the TODO; they edited the wrong file.
- `phases[1].todos[*]` (p2-t3): `gave_up` remains `false`, `practice` remains `true`. The give-up escape is offered to the user but not yet triggered — that requires the user to say "give up on p2-t3" explicitly.
- `mode`: stays `coach`. No transition.
- `phase`: stays `2`. No transition.
- `updated_at`: refreshed to current ISO-8601 timestamp (`<ISO-8601>` — runtime supplies actual value via `date -Iseconds`).

## Events appended

No state-machine events appended. The lock-impl pause is a coaching response, not a mode/phase transition. (The schema's `events[]` is reserved for `phase_start`, `mode_switch`, `phase_done`, `done`, etc. — not turn-by-turn coaching.)

## Notes appended (`notes/phase-2-url-shortener.md`)

Per coach rule 5 (append after substantive coaching exchange), append:

```
## Turn — lock-impl violation, p2-t3

**User question/action:** Modified LLM-authored impl `internal/shortener/validate.go` (gutted scheme check to `return nil`) to make their failing test stub pass.

**Concept:** Lock-impl rule for `user_writes: test` TODOs. Test-first practice requires the impl to be the fixed reference; editing the impl to satisfy a placeholder test inverts the loop.

**Coach explanation:**
- For `user_writes: test`, the LLM-authored impl is the spec for the test, not the other way around.
- The user's "passing" test now asserts nothing meaningful — `return nil` accepts every input.
- Correct path: revert validate.go, replace the `t.Fatal` stub with table-driven cases covering plan.md Task 2.4 behaviors (valid http/https accepted, ftp/empty rejected).
- Bridge: `t.Run` subtests are the Go analog of `describe.each` (Jest) / `@pytest.mark.parametrize` (Python).
- Escape hatch noted: if user genuinely thinks the impl is wrong, the give-up path on p2-t3 produces `fix(phase 2, p2-t3): impl` + `solve(phase 2, p2-t3): test` and excludes the TODO from review.

**Doc URL:** https://pkg.go.dev/testing#T.Run
```

## Files NOT touched

- `internal/shortener/validate.go` — coach mode forbids LLM code writes; user must `git restore` themselves.
- `internal/shortener/validate_test.go` — same; user writes the test.
- `.learn/spec.md`, `.learn/plan.md` — locked since phase 1.

## Next-turn expectations

The skill is reactive. On the next user turn:
- If user runs `git restore validate.go` and asks a test-syntax question → coach normally, reset `turns_since_last_progress` only on a real progress signal (commit, "done with p2-t3" claim, etc.).
- If user says "give up on p2-t3" → trigger lock-impl give-up path: write corrected impl (already correct, so this is a no-op edit committed as `fix(phase 2, p2-t3): impl` for audit) then write the test, two commits, mark `gave_up: true`, `practice: false`, `status: done`.
- If user pushes back ("just let me change it") → hold the line per anti-patterns section; offer skill exit if they want autocomplete.
