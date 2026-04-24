# State updates — turn: capstone confirm → stub path ask

## progress.json — intended changes this turn

No mutating write yet. User confirmed readiness, but the skill is still in the **capstone-setup handshake** (clean-tree check + scope confirmation + stub path ask). None of those resolve this turn, so `mode` stays `coach` until the user replies with (a) clean tree, (b) scope confirmed, (c) stub path.

In particular, do NOT write any of these yet:

- `mode: "solo"` — deferred until after stub file is created and committed (skill step 5).
- `capstone.start_commit` — deferred until `git status --porcelain` confirms clean + we capture `git rev-parse HEAD` (step 2).
- `capstone.last_seen_commit` — initialised at the same moment as `start_commit`.
- `capstone.turns_since_last_progress: 0` — initialised at solo entry.
- `capstone.scaffold_commits: [...]` — populated after we commit the `scaffold(capstone, task C.1)` file, which requires the user's chosen path.
- `plan.md` `## Capstone` amendment — deferred until user confirms the scope paragraph above. (Per invariants, this is one of the two permitted plan.md amendments; written once at solo entry.)

## Event log

Append to `progress.json.events` this turn (single event — the user's explicit capstone-start signal):

```json
{ "ts": "<now ISO-8601>", "kind": "capstone_confirm_received", "note": "awaiting clean-tree + scope + stub path before solo entry" }
```

(If the schema's `kind` enum is strict and doesn't include `capstone_confirm_received`, fall back to a generic `note` event or simply leave events untouched this turn — the state machine does not depend on it. The important thing is that no mode/phase mutation happens yet.)

## Next-turn branches

Depending on user reply:

- **Clean + scope confirmed + path given** → run capstone steps 2–5 in one pass: capture start_commit, append `## Capstone` to plan.md, write the stub file at user-chosen path, commit `scaffold(capstone, task C.1): failing stub at <path>`, capture SHA into `capstone.scaffold_commits`, set `mode: "solo"`, init `capstone.turns_since_last_progress = 0`, `last_seen_commit = start_commit`, log `mode_switch coach→solo` event.
- **Tree dirty** → stay in coach, remind to commit/stash, do not capture start_commit. Re-ask.
- **Scope pushback (e.g. "drop concurrency")** → revise the behaviors list in the response, re-confirm, do not yet write `## Capstone`.
- **Path ambiguous or user defers the choice** → ask once more with a concrete default suggestion; do not pick unilaterally (the user-designs-the-shape rule applies — path choice is part of their design ownership).
- **User reverses ("actually not ready")** → stay in coach, `mode` unchanged, no events appended beyond a neutral note.

## Invariant checks held this turn

- No code block was emitted (coach/pre-solo mode — only the `git status --porcelain` bash invocation shown for the user to run, and the unimplemented-stub one-line pattern quoted as a convention, not authored code).
- `spec.md` not touched.
- `plan.md` not yet touched (the `## Capstone` append is deferred to the confirmation turn, which is correct per skill step 3).
- No stub file created yet — skill step 4 explicitly requires asking the user's chosen path first. This is exactly the branch the scenario is evaluating.
