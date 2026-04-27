# State updates — eval-capstone-stub-path-ask, turn 1

Turn entry condition: `phase == total_phases (4)`, `mode == 'coach'`, all phases `done`, `capstone.required == true`, `capstone.completed == false`. Previous turn asked the solo-entry confirmation prompt; user replied "yes ready for capstone".

## What this turn DID change

Nothing on disk yet. CAPSTONE.md step 2 requires a clean-tree check (`git status --porcelain`) before recording `start_commit`, and step 4 requires the user to pick the stub path before any file is created. Both are blocking on the user's next reply, so this turn is a setup + question turn — no commits, no `progress.json` writes, no `plan.md` append. Holding the line on "no code, no scaffolding until preconditions met" is the right move per CAPSTONE.md.

If I had blindly recorded `start_commit` against a dirty tree, every uncommitted change from phase 4 would be silently absorbed into "user-owned for capstone purposes" at the gate, corrupting the authorship check. Hence the gate-before-write ordering.

## What this turn WILL change on the user's next reply

Assuming the user replies with a clean tree and a stub path (call it `<PATH>`):

### `progress.json` mutations

```jsonc
{
  // unchanged: $schema_version, project_name, slug, created_at, worktree_path,
  //            git_identity, stack, time_budget_hours, phase_time_target_min,
  //            ui_project, projected_phases, phase, total_phases, prereqs, phases

  "updated_at": "<ISO-8601 from `date -Iseconds`>",

  "mode": "solo",                          // was "coach"
  "turns_since_last_progress": 0,          // top-level counter — reset on mode switch

  "capstone": {
    "required": true,                      // unchanged
    "completed": false,                    // unchanged
    "feature": "Redirect-hit counter per short code (see plan.md ## Capstone)",  // NEW
    "start_commit": "<git rev-parse HEAD output>",                                // NEW
    "turns_since_last_progress": 0,                                               // NEW
    "last_seen_commit": "<same SHA as start_commit>",                             // NEW
    "scaffold_commits": ["<SHA of the Task C.1 stub commit>"]                     // NEW — appended after stub commit
  },

  "events": [
    // ...prior events unchanged...
    { "ts": "<ISO-8601>", "kind": "capstone_start",  "start_commit": "<SHA>" },
    { "ts": "<ISO-8601>", "kind": "scaffold_commit", "phase": "capstone", "task": "C.1", "sha": "<SHA>", "path": "<PATH>" },
    { "ts": "<ISO-8601>", "kind": "mode_switch",    "from": "coach", "to": "solo" }
  ]
}
```

Note: `capstone.feature` is a short pointer to plan.md rather than the full spec, because plan.md is the single source of truth for the feature description (STATE-SCHEMA.md invariants).

### `plan.md` mutation (one of two permitted post-lock amendments)

Append at end of file:

```markdown
## Capstone
<!-- appended at solo entry -->

### Feature
A redirect-hit counter. Every time a short code is resolved and redirected,
its hit count increments. The current count for any short code is readable
on demand. The counter persists across redirects within a single process
lifetime; durability beyond that is out of scope.

### Task C.1
A single failing test stub at <PATH>. Body uses Go's unimplemented-test
pattern: t.Fatal("write this test — capstone feature in plan.md ## Capstone").
The user designs every other file, signature, and test.

### Behaviors to cover
- New short code reports zero hits before any redirect.
- Each successful redirect increments that code's count by exactly one.
- Counts are independent per short code.
- Reading a non-existent short code's count is handled deliberately and tested.
- Concurrent redirects to the same code do not lose increments.

### Docs
- https://pkg.go.dev/sync
- https://pkg.go.dev/sync/atomic
- https://pkg.go.dev/net/http/httptest
```

(Doc URLs subject to `find-docs` resolution against the locked Go version before the actual write.)

### Filesystem mutation

Create exactly one file at `<PATH>` with content:

```go
package <pkg-from-PATH>

import "testing"

func TestRedirectHitCounter(t *testing.T) {
    t.Fatal("write this test — capstone feature in plan.md ## Capstone")
}
```

Then:

```bash
git add <PATH>
git commit -m "scaffold(capstone, task C.1): failing stub at <PATH>"
git rev-parse HEAD   # → append to capstone.scaffold_commits
```

### `notes/phase-capstone-<slug>.md`

Append a coach note recording: solo entry confirmation, start_commit SHA, stub path, and a one-liner that "the LLM will not write code from here until review."

## Files NOT touched

- `spec.md` — fully read-only post-lock (STATE-SCHEMA.md invariant). The `## Reserved for capstone` line was the seed for plan.md's `## Capstone` section; spec itself is unchanged.
- `theme.md` — N/A (no `ui_project` indicator in scenario).
- `review.md` — written exactly once at capstone gate pass; not now.
- Any non-test source files — capstone is stricter than `user_writes: both`. No impl file, no signature stub, no hint comments outside the one test stub.

## Invariants preserved this turn

- `spec.md` untouched.
- `plan.md` will receive only the `## Capstone` amendment (1 of 2 permitted post-lock amendments; the re-plan amendment slot is unused and remains available).
- No `[PRACTICE]` markers introduced — capstone deliberately uses ownership-by-diff (`git diff <start_commit>..HEAD` minus `scaffold_commits`) rather than per-TODO tagging.
- Mode transition is logged as an event before any code-write authority shifts.
- Coach rule still active until the moment `mode` flips to `solo`; no code written this turn.
