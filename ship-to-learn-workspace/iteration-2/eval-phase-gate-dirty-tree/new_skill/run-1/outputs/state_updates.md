# State updates — turn: "done phase 1" with dirty tree

## progress.json mutations

**None.**

The phase gate (PHASE-LOOP.md, "Phase gate", step 1a — commit-clean check) blocked before any state-changing action. Per the rule:

> If any file in the phase file-set shows uncommitted changes, refuse to advance: "Commit your work on phase N first." Do not proceed.

Resulting state (unchanged from input scenario):
- `phase`: `1`
- `mode`: `"coach"`
- `phases[0].status`: `"in_progress"`
- `phases[0].todos[*].status`: all `"done"` (p1-t1..p1-t5)
- `stack.test_runner`: `"go test ./..."`

No `feedback` recorded, no `completed_at` written, no `mode` change, no `phase` increment.

## Events log

**None appended.** The `phase_done` event fires only after step 4 of the gate (tests pass + feedback recorded). A dirty-tree refusal at step 1a is a soft block — user retries the same turn after committing — and the schema does not define a `phase_gate_blocked` event for this path. Logging one would be a schema extension, not current behavior.

## turns_since_last_progress

**Effective value after this turn: `0`.**

The user's "done phase 1" is a forward progress signal under PHASE-LOOP.md coach rule 10 ("Progress signals (any one resets the counter): (a) User says 'done with <id>' / 'fixed <thing>' / 'progress on X' / similar forward claim."). The counter resets to 0 regardless of whether the gate ultimately advances — the signal is a claim of progress, not its verification. The gate-block does not undo that.

## Files touched

**None on disk.** No reads of `plan.md`, `spec.md`, or `notes/*` were required. The gate-block decision is fully determined by the two inputs in scope:
- `progress.json` (TODO statuses — provided in scenario)
- `git status --porcelain` (provided in scenario)

The response's reference to "phase 1's file-set per plan.md" describes what *would* be parsed on the user's retry with a clean tree. No parse happened this turn.

## Notes append

**None.** PHASE-LOOP.md coach rule 5 prescribes a notes append "after every substantive coaching exchange" — meaning a Q&A about a concept, with a doc URL, bridge to a known stack, etc. A procedural gate-block refusal does not match: there is no concept name, no `find-docs` call, no bridge. Skipping the notes append is intentional, not an oversight.

## Next-turn expectations

When the user retries `"done phase 1"` after committing:

1. Re-run `git status --porcelain` on the phase file-set → expect clean (gate step 1a passes).
2. Run `go test ./...` → capture output (gate step 2).
3. On pass:
   - Ask difficulty question (gate step 4). Record `phases[0].feedback` ∈ {`"too_easy"`, `"right"`, `"too_hard"`, `null`}.
   - Set `phases[0].status = "done"`, `phases[0].completed_at = <ISO-8601-UTC>` (e.g. `"2026-04-25T..."`).
   - Append event `{type: "phase_done", at: <ISO-8601-UTC>, phase: 1}`.
   - Apply re-plan rule (compare last two phases' `duration_min` vs `phase_time_target_min`; only one phase complete here so no re-plan).
   - Increment `phase` to `2`. Set `mode = "build"`. Begin phase 2 scaffolding per PHASE-LOOP.md "On entering build for a new phase".
4. On fail: stay `mode = "coach"`, paste failing tests, hint at fix, do not write the fix.
