# State updates — turn: user confirms capstone

This turn the skill is **mid-transition** between coach and solo. The user said yes, but per SKILL.md "On entering `solo`" the skill must (a) verify clean tree, (b) size-check, (c) ask the user where the single failing-test stub lives, **before** any of the substantive state writes (start_commit capture, plan.md amendment, scaffold commit, mode flip). So this turn's persisted updates are minimal — guard checks come first, real writes happen next turn after the user replies with the path.

## Files written this turn

None.

- `.learn/plan.md` — **not** amended yet. The `## Capstone` append happens after the user picks a stub path and the size-check passes (next turn).
- `.learn/spec.md` — untouched (read-only post-lock; only the existing `## Reserved for capstone` section is referenced).
- No new files created. The single LLM-authored capstone file (the failing-test stub) is deferred until the user provides the path.

## `progress.json` updates this turn

Strictly speaking, the conservative read of SKILL.md is that no `progress.json` mutation is required this turn — the skill is awaiting a path-decision response from the user, and `mode` is still `coach`. However, two small bookkeeping updates are reasonable and consistent with rule "Update `progress.json` on every state change":

```jsonc
{
  "updated_at": "<now ISO-8601>",
  "events": [
    // ...prior events...
    {
      "ts": "<now ISO-8601>",
      "kind": "capstone_confirm",   // user said yes; not yet a mode_switch
      "note": "user confirmed capstone; awaiting stub-path decision and clean-tree check"
    }
  ]
}
```

Notably **NOT** updated this turn (deferred to the next turn after the user replies):

- `mode` — stays `"coach"` until after the stub-path is chosen, the stub file is written, and the scaffold commit lands. Only then does it flip to `"solo"`. Flipping prematurely would put the skill in solo without the start_commit anchor, breaking the gate's diff-based ownership rule.
- `capstone.start_commit` — captured from `git rev-parse HEAD` only **after** `git status --porcelain` returns clean. Not captured this turn.
- `capstone.last_seen_commit` — initialized to `start_commit` at the same moment.
- `capstone.turns_since_last_progress` — initialized to `0` at solo entry; not yet.
- `capstone.scaffold_commits` — appended with the SHA of the `scaffold(capstone, task C.1): failing stub at <path>` commit, after the user picks a path and the commit lands. Not yet.
- `capstone.completed` — stays `false`; will only flip when the capstone gate passes.

## Mode

- Current: `coach` (last non-capstone phase gate already passed; capstone-readiness prompt fired previous turn).
- Next: `solo`, but only after (a) clean-tree confirmation, (b) user picks stub path, (c) stub file is created and committed, (d) `start_commit` recorded.

## Outstanding asks to user (this turn)

1. Confirm `git status --porcelain` is empty (clean tree). If not, commit/stash and report back.
2. Pick the path for the single failing-test stub (e.g. `internal/stats/stats_test.go`).

## Why no plan.md amendment this turn

SKILL.md lists `## Capstone` as one of two permitted amendments to plan.md, appended "once at capstone start to record the capstone feature spec." The "On entering `solo`" sequence places the plan amendment **after** the clean-tree / start_commit capture (steps 2 to 3) and **before** the stub creation (step 4). Since this turn pauses at step 2 awaiting user input (clean-tree status), the amendment is correctly deferred. Writing it now would mean amending plan.md while the tree state is unverified — a contract risk if the user has uncommitted changes that include speculative capstone work that should be folded into `start_commit`.

## Notes file

No append to `.learn/notes/phase-4-<slug>.md` this turn — the rule applies "after every substantive coaching exchange." This turn is a state-transition handshake, not coaching. (A note may be appended next turn if the user asks any conceptual question while picking the stub path.)
