# State updates — scope-change-refuse turn

## Files touched

- `.learn/spec.md` — **not modified** (read-only, locked since phase 1).
- `.learn/plan.md` — **not modified** (append-only; scope-change amendments are not among the two permitted amendments).
- `.learn/progress.json` — **no state-change updates** from this turn (no phase advance, no TODO status change, no mode switch, no give-up). The only field that may increment is the transient coach counter; see below.
- `.learn/notes/phase-3-url-shortener.md` — **not appended**. Rule 5 ("append after substantive coach exchanges") covers concept-coaching exchanges; a scope-refusal is policy enforcement, not concept instruction, so no note is warranted. (If the author disagrees, a one-line entry of form `scope refusal: user requested OAuth; declined per scope-lock rule; offered new-session path` would be the minimum acceptable append.)

## progress.json field-level changes

| Field | Before | After | Reason |
|---|---|---|---|
| `phase` | 3 | 3 | No gate triggered. |
| `mode` | `coach` | `coach` | No transition; refusal is a coach no-op. |
| `turns_since_last_progress` | N | N+1 | No progress signal this turn (no "done with <id>", no new commit on phase files, no failing→passing flip). Per rule 10 increment rule. |
| `updated_at` | prev | `<now ISO-8601>` | Standard per-turn bump since progress.json was read+written (even as a no-op increment). |
| `events[]` | ... | unchanged (no new event) | No state-change kind applies (no `phase_start`, no `mode_switch`, no `phase_done`, no `done`). Scope-refusal isn't an enumerated event kind. |

If `turns_since_last_progress` reaches ≥10 after this increment, the proactive check-in from rule 10 fires on the *next* turn, not this one.

## Rule citations (SKILL.md) driving the refusal

- **State invariants — `spec.md` is fully read-only after creation. No amendments ever.** (line ~129)
- **State invariants — `plan.md` append-only with exactly two permitted amendments** (`## Re-plan at phase N`, `## Capstone`). Scope additions are not among them. (line ~131)
- **State invariants — Scope change requests (new feature, expanded goal) are refused in either file. Instruct the user to start a new `/ship-to-learn` session. Scope drift is the single biggest threat to this skill's value.** (line ~135)
- **Trigger phrases table — "change the scope" / "add feature X" → Refuse. Explain scope lock. Offer: start new session.** (line ~800)
- **Anti-patterns — "Expanding scope mid-project because user asked. Refuse. Start a new session."** (line ~832)
- **Pushback section — if user disagrees, hold the line or direct them to exit the skill for a regular coding session.** (line ~838)

## Mode/guard check performed this turn

1. Read `progress.json` → `{ phase: 3, mode: 'coach' }`. Parse OK.
2. Detected mode = `coach`. Allowed actions: explanation, bridge, doc pointer, line-referenced critique. **Not allowed:** scope edits, code blocks >1 line, plan amendments outside the two permitted kinds.
3. Self-checked response for LLM-authored code blocks >1 line → none present. Response is hint/policy text only.
4. No code writes, no spec/plan edits, no TODO status changes.

## What would change the outcome

- If user had instead said "done with p3-t2" → mark that TODO `status: done`, reset counter to 0, stay in coach.
- If user says "give up on p3-t<X>" → one-shot build for that TODO only (rule 7), not a scope expansion.
- If user says "stop" / "exit skill" → save `progress.json`, print state summary, exit. They can then add OAuth freely in a normal session.
- If user insists on OAuth-in-this-project → continue refusing per pushback rules; do not convert the session into autocomplete.

## Follow-ups queued (in-memory, not persisted)

- If the user accepts option 2 and starts a new session later, the new `/ship-to-learn` invocation will see `mode: coach` (still active here) and prompt the resume/abandon/cancel triplet per Phase 0 step 1(i). OAuth project only begins cleanly after current project reaches `mode: done` or user explicitly abandons `.learn/`.
