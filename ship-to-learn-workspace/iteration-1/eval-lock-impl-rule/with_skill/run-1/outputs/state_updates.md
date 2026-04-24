# State updates — turn for p2-t3 lock-impl violation

## `progress.json` mutations

**None this turn.** The user has not claimed a TODO done, not requested give-up, not signalled phase done. Lock-impl rule 8 triggered; we're pausing coaching to get the impl reverted. No state transition.

Specifically:
- `phase`: stays `2`
- `mode`: stays `"coach"`
- `phases[1].todos[<p2-t3 index>].status`: stays `"pending"` (or `"in_progress"` if already set — unchanged either way)
- `phases[1].todos[<p2-t3 index>].gave_up`: stays `false`
- `phases[1].todos[<p2-t3 index>].practice`: stays `true`
- `turns_since_last_progress`: **increment by 1** (no progress signal this turn — user's "tests pass" claim is invalidated because it was achieved by deleting the impl under lock; does not count as progress per rule 10(a) since the claim is not honest forward motion on the test-writing task). New value: previous + 1.
- `phases[1].last_seen_commit`: **unchanged**. The user's diff is uncommitted (they pasted it, haven't run `git commit`). Even if they had committed, rule 10(b) compares commits on phase files — but this commit would be a lock-violation edit to `validate.go`, not forward progress on the test file, so we would not treat it as a progress signal. Leave `last_seen_commit` as-is.
- No new `events` entry. Lock-impl pause is not an enumerated event kind. (If we later extend the schema, `lock_impl_pause` would be the natural name — for now, it lives only in the coach notes.)

## `notes/phase-2-<slug>.md` append

Append a new entry (coach rule 5 — substantive exchange):

```markdown
## <ISO-8601 timestamp> — p2-t3 lock-impl trigger

**User action:** pasted diff showing `validate.go` (LLM-authored, locked) edited to delete scheme check, returning `nil` unconditionally. Stated goal: "make my tests pass."

**Concept:** LLM-authored impl regions are locked for `user_writes: test` TODOs. The test is the spec; the impl is the subject. Editing the subject to satisfy the spec inverts TDD.

**Response in 3–5 bullets:**
- Refused the edit; instructed `git restore internal/shortener/validate.go`.
- Flagged that the diff didn't just tweak behavior — it removed validation entirely, so every input now passes.
- Asked for the test body + failure output before hinting, to verify the user understands what `Validate` is supposed to do.
- Offered the legitimate escape: if user can cite a doc showing the impl is genuinely wrong, invoke give-up on p2-t3 (two commits: `fix(phase 2, p2-t3): impl` + `solve(phase 2, p2-t3): test`, mark `gave_up: true`, `practice: false`, `status: done`). Otherwise hold the line.
- Stayed in coach mode.

**Doc URL:** n/a this turn — no specific idiom cited yet; will cite when user pastes test + we discuss the scheme-check semantics (likely `net/url` Parse vs. manual prefix check: https://pkg.go.dev/net/url#Parse — to be verified via find-docs when relevant).
```

## Next-turn expectations

Three branches:

1. **User reverts + pastes test + failure output.** Resume normal coach-mode critique of the test. Cite `net/url` or `testing` docs via find-docs. Reset `turns_since_last_progress` to 0 only if they also signal forward progress (e.g. "ok here's my new test attempt").
2. **User claims the impl is wrong with a doc citation.** Verify via find-docs / context7. If correct → invoke give-up path for p2-t3 (two-commit fix+solve sequence per rule 8); set `gave_up: true`, `practice: false`, `status: done`; update `turns_since_last_progress = 0`; log `todo_gave_up` or equivalent event if schema supports.
3. **User pushes back ("just let me edit the impl" / "why are you being strict").** Hold the line per the Pushback section. Do not relent. If they escalate to "stop being a coach, write the code," exit the skill per the closing of that section; do not convert mid-session into autocomplete.

## Files NOT touched

- `spec.md` — locked post-phase-0. Untouched.
- `plan.md` — no amendment permitted here (not a re-plan trigger, not capstone entry). Untouched.
- `validate.go`, `validate_test.go` — user's territory; coach mode does not write code.
