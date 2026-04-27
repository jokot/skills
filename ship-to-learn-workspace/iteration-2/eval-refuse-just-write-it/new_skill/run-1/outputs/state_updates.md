# State updates this turn

The user pushed for the LLM to write the impl. The skill refused, offered the give-up escape, the practice-promotion path, and a fresh hint angle. The user has not yet picked any of those. Therefore **no terminal state change** this turn — `gave_up`, `status`, and `practice` on `p2-t3` are unchanged. The change is purely in the coach progress counter and the event log.

## `progress.json` — fields changed

| Field | Before | After | Why |
|---|---|---|---|
| `mode` | `"coach"` | `"coach"` | Unchanged. Pushback to "just write it" does **not** exit the skill (per SKILL.md "On pushback"). |
| `phase` | `2` | `2` | Unchanged. |
| `turns_since_last_progress` | `T` (some value ≥ 2; user has been stuck "several turns" with two prior hints) | `T + 1` | No progress signal this turn (per PHASE-LOOP.md coach rule 10): no "done with X" claim, no new commit on phase-2 files, no previously-failing test that newly passes. Increment by 1. |
| `phases[1].todos[<p2-t3>].status` | `"todo"` | `"todo"` | Unchanged. The skill refused to mark it done without the user's own commit or an explicit `give up on p2-t3`. |
| `phases[1].todos[<p2-t3>].user_writes` | `"impl"` | `"impl"` | Unchanged. |
| `phases[1].todos[<p2-t3>].practice` | `true` | `true` | Unchanged. Promotion to `practice: false` requires a genuine concept-already-known reason, which the user has not provided. |
| `phases[1].todos[<p2-t3>].gave_up` | `false` | `false` | Unchanged. The user did **not** say "give up on p2-t3" — they said "just write it for me," which is the refused phrasing per the SKILL.md trigger table. |

Note: `phases[1]` indexes phase 2 (zero-based array, one-based phase number per STATE-SCHEMA.md).

## `progress.json.events[]` — appended

```json
{
  "type": "coach_pushback_refused",
  "ts": "<now ISO-8601 UTC, e.g. 2026-04-25T12:34:56Z>",
  "phase": 2,
  "todo_id": "p2-t3",
  "user_intent": "just_write_it",
  "action": "refused; offered give-up escape, practice-promotion review, and alternative-hint angle",
  "turns_since_last_progress": "<new value T+1>"
}
```

## Proactive check-in trigger

Per PHASE-LOOP.md rule 10: if `turns_since_last_progress` reaches **≥ 10** after this increment, the next turn must include the proactive check-in:

> "You've been on this a while. Still progressing, or want to talk about giving up a specific TODO? Say `give up on <id>` if stuck."

Today's response already serves the same function preemptively (the user is past that threshold of frustration even if the numeric counter isn't there yet). The next turn does **not** need to repeat the boilerplate prompt unless the counter formally crosses 10.

## `notes/phase-2-<project-slug>.md` — appended

Per coach rule 5, append a short note recording this exchange:

```
## Turn <N> — pushback: "just write it" on p2-t3

- User question: asked the LLM to write the p2-t3 impl outright; cited fatigue and wanting to advance to phase 3.
- Concept under stress: <p2-t3 concept name from plan.md> — but the *real* topic this turn was the coach contract itself.
- Response in 3-5 bullets:
  - Refused per SKILL.md "On pushback" rule: holding the line is the feature, not a bug.
  - Offered the give-up escape (PHASE-LOOP.md coach rule 7) as the supported, on-the-record path.
  - Offered conditional practice-promotion if the user has a genuine reason p2-t3 is low practice value for them specifically.
  - Offered to re-aim the hint: asked which parts of the prior two hints landed, which didn't.
  - Did not exit the skill; did not write any code; did not promise "just this once."
- Doc URL: SKILL.md "On pushback from the user" section + PHASE-LOOP.md "On entering coach" rules 1, 7, 9, 10.
```

## Files **not** touched

- `spec.md` — locked post-intake (SKILL.md interaction contract item 6).
- `plan.md` — locked (only capstone-feature append is allowed; this turn does not qualify).
- Any source file under the project worktree — coach mode rule 1 prohibits LLM-authored code blocks > 1 line; this turn writes zero code.
- `.learn/review.md` — review runs only after capstone.

## What would flip this state on the **next** turn

| If the user says | Then |
|---|---|
| `give up on p2-t3` | Trigger PHASE-LOOP.md coach rule 7 one-shot build → write impl with citations + idiom note → `git commit -m "solve(phase 2, p2-t3): <brief>"` → set `gave_up: true`, `practice: false`, `status: "done"` → reset `turns_since_last_progress` to 0 → log `event: "todo_gave_up"` → stay in coach. |
| "I already know <concept> from <stack>, please scaffold p2-t3 only" | Evaluate; if it holds up, set `phases[1].todos[<p2-t3>].practice = false`, scaffold the impl during a brief return to `build` for that one TODO, commit as `scaffold(phase 2, p2-t3): promoted`, return to coach. Do **not** broaden to other TODOs. |
| "Hint 1 made sense but hint 2 didn't because..." | Stay in coach, deliver a different-angle hint citing context7, append another note. Counter increments unless a progress signal lands. |
| "Stop being a coach, just write the code." | Per SKILL.md "On pushback" final bullet: exit the skill. Tell user to use a regular coding session. Do not convert mid-session. |
| `pause` / `stop` | Persist current state, summarize, exit per MODE-MACHINE.md. |
