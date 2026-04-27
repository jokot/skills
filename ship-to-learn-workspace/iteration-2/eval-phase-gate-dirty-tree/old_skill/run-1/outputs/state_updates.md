# State updates — eval-phase-gate-dirty-tree (old_skill, monolithic pre-split)

User signal: "done phase 1"
Gate result: **refused at step 1a (commit-clean check)** — phase file-set has uncommitted modifications.
Mode: stays in `coach`. No phase advance.

## Reasoning trace

1. Read `progress.json`. `phase: 1`, `mode: 'coach'`, all five `phases[0].todos[*].status == 'done'`. Step 1 of phase gate (TODO completeness) passes.
2. Step 1a — commit-clean check:
   - Phase file-set parsed from `plan.md` Phase 1 tasks (synthesized for this scenario): `internal/shortener/shortener.go`, `internal/shortener/shortener_test.go`.
   - `git status --porcelain` reports both files as ` M` (modified, unstaged).
   - Intersection with phase file-set is non-empty → **refuse**, halt before running tests.
3. Step 2 (`go test ./...`) **not executed** — gate halts in order at the first failed precondition.
4. Step 3, 4 (difficulty question, advance) — not reached.

## `progress.json` mutations (this turn)

| Field | Before | After | Why |
|---|---|---|---|
| `phase` | `1` | `1` | No advance. |
| `mode` | `"coach"` | `"coach"` | Gate refused; stay in coach. |
| `phases[0].status` | `"in_progress"` | `"in_progress"` | Phase not done until gate passes. |
| `phases[0].todos[*].status` | all `"done"` | all `"done"` | User claims accepted at face value (rule 6); gate is verifier. |
| `phases[0].completed_at` | (unset) | (unset) | Set only on gate pass. |
| `phases[0].duration_min` | (unset) | (unset) | Same. |
| `phases[0].feedback` | `null` | `null` | Difficulty question not asked. |
| `turns_since_last_progress` | `<prev>` | `<prev> + 1` | No progress signal this turn — user's "done phase 1" is a gate signal, not a per-TODO progress signal, and the gate refused. Counter increments per coach rule 10. |
| `updated_at` | `<prev ISO>` | `<now ISO>` (e.g. `2026-04-25T14:32:10+07:00`) | Standard touch on any state read/write. |

## `events[]` append

```json
{"ts": "2026-04-25T14:32:10+07:00", "kind": "phase_gate_blocked", "phase": 1, "reason": "dirty_tree", "files": ["internal/shortener/shortener.go", "internal/shortener/shortener_test.go"]}
```

Note: the pre-split SKILL.md schema lists example event kinds (`phase_start`, `mode_switch`, `phase_done`) but does not enumerate them as a closed set. `phase_gate_blocked` is consistent with the pattern — captures the failed gate attempt for resume + later review without claiming the phase is done.

## Files NOT touched

- `.learn/spec.md` — locked.
- `.learn/plan.md` — locked except for the two permitted amendments; neither applies here.
- `.learn/notes/phase-1-<slug>.md` — no substantive coach exchange this turn (it's a gate refusal, not a Q&A). Skipping the append is consistent with rule 5 ("after every substantive coaching exchange"). Defensible either way; choosing skip to keep notes signal-rich.
- No code files written. Coach mode holds.

## Next-turn expectations

- If user commits and re-signals "done phase 1": re-enter gate at step 1, re-check commit-clean, run `go test ./...`, ask difficulty, advance to phase 2 build on green.
- If user pushes back ("just advance, the changes are trivial"): refuse per anti-patterns ("hold the line"). Restate the commit-clean rationale (authorship + diff-based ownership for later review).
- If user says "give up on p1-tX" instead: out of scope here — all TODOs already `done`. Treat as confused signal, ask for clarification.
