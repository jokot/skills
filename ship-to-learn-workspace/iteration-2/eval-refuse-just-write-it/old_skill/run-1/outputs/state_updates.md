# State updates — this turn

## Summary

User sent a "just write it for me" pushback in `coach` mode on `p2-t3`. Per monolithic SKILL.md:

- **Coach rule 1** (self-check, no LLM-authored code blocks > 1 line): held — response contains zero code.
- **Coach rule 7** (give-up escape is the only LLM-writes-code path inside coach): not yet triggered — user said "just write it", not "give up on p2-t3". The give-up path requires the explicit signal; "just write it" alone does not authorize a solve.
- **Pushback rules** (lines 838–843): user is "frustrated and pushing harder" — hold the line. Did not promote `p2-t3` to `practice: false`, because the user did not present a "genuinely good reason" (e.g. "I already learned this concept in another language"); they presented urgency ("I need to move on to phase 3"), which the skill explicitly does not honor.
- **Mode switching** (lines 261–289): no user-controlled transition to `build` exists. "switch to build so you can write code for me" is explicitly never user-controlled. Stay in `coach`.
- **Trigger phrases** ("just write it" / "do it for me" -> "Refuse in coach/solo. Remind them of purpose. In build, fine."): applied verbatim.

## `progress.json` mutations this turn

Read-modify-write of `.learn/progress.json`:

| Field | Before | After | Why |
|---|---|---|---|
| `phase` | `2` | `2` | No phase change. Phase gate not triggered (user didn't signal "done phase 2", and `p2-t3` is not `status: done`). |
| `mode` | `"coach"` | `"coach"` | No mode transition. User-triggered switch to `build` is not a permitted transition (mode-switching table). Implicit no-op stays in coach. |
| `turns_since_last_progress` | `<prev>` (>= a few; user has been stuck "several turns") | `<prev> + 1` | Coach rule 10 increment rule: no progress signal this turn. User did not say "done", "fixed", "progress on X"; no new commit on phase files (user is asking for code, not committing); no test transition observed. Counter increments. |
| `phases[1].todos[2].status` (id `p2-t3`) | `"in_progress"` (assumed; user has been working on it) | `"in_progress"` | Unchanged. Not marked `done` (user did not claim done), not marked `gave_up` (user did not say "give up on p2-t3" — they said "just write it", which the skill refuses). |
| `phases[1].todos[2].gave_up` | `false` | `false` | Unchanged. Give-up flag flips ONLY on explicit "give up on p2-t3" signal per rule 7 + trigger-phrase table. |
| `phases[1].todos[2].practice` | `true` | `true` | Unchanged. Pushback rule says promote-to-LLM-authored only on a "genuinely good reason" (already knows the concept in another language). User did not give one — they gave urgency. Not promoted. |
| `phases[1].todos[2].attempts` | `<prev>` | `<prev>` | Unchanged. Attempts counter is per-attempt, not per-turn; user did not submit a new attempt. |
| `updated_at` | `<prior ISO-8601>` | `<now ISO-8601>` (from `bash date -Iseconds`) | Always update on state change; counter increment qualifies. |

**Counter check-in threshold (coach rule 10):** if `turns_since_last_progress` reaches `>= 10` after this increment, the proactive check-in triggers next turn ("You've been on this a while..."). The current response already pre-empts that by offering the give-up path explicitly, but the counter still drives the formal check-in if the user keeps pushing without signalling progress or give-up. We do NOT reset the counter — "user pushed back" is not a progress signal.

## `events[]` append

One event appended to `progress.json.events`:

```json
{
  "ts": "<now ISO-8601 from bash date -Iseconds>",
  "kind": "pushback_refused",
  "phase": 2,
  "todo_id": "p2-t3",
  "user_message_summary": "asked LLM to write impl to move to phase 3",
  "action": "refused; offered give-up escape + alternative coaching angles; held coach mode"
}
```

(`pushback_refused` is not in the explicit enum table at lines 211–216, but `events[]` is open-ended for audit; the three examples there are illustrative, not exhaustive. If the harness enforces a closed enum, fall back to omitting this event — the counter increment alone is a sufficient state record.)

## `notes/phase-2-<project-slug>.md` append

Per coach rule 5 (append after substantive coaching exchanges):

```markdown
## Turn <N> — pushback on p2-t3

**User:** "I've had enough. Just write the implementation for p2-t3 for me, I need to move on to phase 3."

**Concept at stake:** <whatever p2-t3 teaches — derived from plan.md's p2-t3 PRACTICE step Bridge/Idiom note, e.g. "error wrapping with %w in Go" or equivalent>

**Coach response in 3–5 bullets:**
- Refused write per coach rule 1 + pushback rule "hold the line on frustration".
- Reminded user of skill's purpose: their fingers on keyboard, tests as spec.
- Offered formal give-up escape (`give up on p2-t3` -> one-shot solve, `gave_up: true`, excluded from review).
- Offered four alternative coaching angles (different bridge, line-by-line critique, tighter doc cite, analogous example in known stack).
- Made exit-the-skill explicit as a clean third option, to avoid silent autocomplete drift.

**Doc URL:** <whichever context7 URL was previously cited for p2-t3 in plan.md; not re-queried this turn since no new concept was introduced>
```

## Files NOT modified

- `.learn/spec.md` — locked (Invariants).
- `.learn/plan.md` — locked except for the two permitted amendments (`## Re-plan`, `## Capstone`); neither applies.
- `.learn/theme.md` — N/A (scenario doesn't indicate `ui_project`); locked if present.
- Source files (`<project worktree code>`) — coach mode forbids LLM writes; no edits.
- `.learn/review.md` — only written once at capstone end; not now.

## What would change state next turn

| User's next message | State delta |
|---|---|
| `"give up on p2-t3"` | Trigger coach rule 7: write `p2-t3` impl, commit `solve(phase 2, p2-t3): <brief>`, set `todos[2].status = "done"`, `gave_up = true`, `practice = false`, reset `turns_since_last_progress = 0`, append `event: "give_up"`, return to coach. |
| `"different bridge please"` (or any of the offered angles) | Stay in coach; no `progress.json` mutation beyond the next-turn counter increment; append a fresh notes entry with the new explanation. |
| `"stop"` / `"exit ship-to-learn"` | Save `progress.json` as-is, print resume hint, exit (Exiting the skill rule 2). |
| Pushes harder ("no really, just write it") | Refuse again. Counter increments. At `>= 10` trigger formal check-in per rule 10. Never write the code. |
| `"I already know error wrapping from Rust, just scaffold this one"` | Pushback rule "genuinely good reason" branch: promote `p2-t3` to `practice: false`, LLM writes the impl as scaffolding (not give-up; no `gave_up` flag). Limited to this one TODO, not "all TODOs". |
