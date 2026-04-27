# State — `.learn/` directory + `progress.json` schema

> Persistent state layout for ship-to-learn. See [SKILL.md](./SKILL.md) for the entry point.

All project state lives in `.learn/` at the repo root of the worktree. Single source of truth.

```
.learn/
  spec.md              # locked after phase 0 (from superpowers:brainstorming)
  plan.md              # locked after phase 0 (from superpowers:writing-plans)
  progress.json        # live — you update it on every state change
  notes/
    phase-<N>-<project-slug>.md  # append-only coach notes (one file per phase, uses progress.json.slug)
  theme.md             # optional — present only when ui_project == true; written by ui-ux-pro-max at step 5a, locked at same moment as spec.md/plan.md
  review.md            # written once at capstone completion
```

## Invariants

- `spec.md` is **fully read-only** after creation. No amendments ever.
- `theme.md` (if present — `ui_project == true`): **fully read-only** after creation. Same lock moment as spec.md (`progress.json.phase = 1`). Theme changes mid-project require a new `/ship-to-learn` session.
- `plan.md` is **append-only after creation**, with exactly two allowed amendments — nothing else:
  1. `## Re-plan at phase N` — appended only when two consecutive phases overran the target by ≥150%.
  2. `## Capstone` — appended once at capstone start to record the capstone feature spec.
- **Lock moment:** both files are considered created and locked the instant `progress.json.phase` is set to `1` (first `build`-phase entry). Until that point, ship-to-learn may still amend both files to meet the required-sections rule (e.g. inject a bridges table post-brainstorming).
- Scope change requests (new feature, expanded goal) are refused in either file. Instruct the user to start a new `/ship-to-learn` session. Scope drift is the single biggest threat to this skill's value.
- `progress.json` is the ground truth for current phase, mode, and TODO state. Read it at the start of every turn. Update and save it on every state change. All timestamp fields (`created_at`, `updated_at`, `started_at`, `completed_at`, event `ts`) are ISO-8601 strings obtained via `bash date -Iseconds` (or equivalent). Phase durations compute from these.
- `notes/` is append-only. You may add files, never rewrite.
- `review.md` is written exactly once.

## `progress.json` schema

```json
{
  "$schema_version": 1,
  "project_name": "url-shortener",
  "slug": "url-shortener",
  "created_at": "ISO-8601",
  "updated_at": "ISO-8601",
  "worktree_path": "/abs/path/to/worktree",
  "git_identity": "user@example.com",
  "stack": {
    "language": "go",
    "version": "1.22",
    "test_runner": "go test ./...",
    "formatter": "gofmt"
  },
  "time_budget_hours": 6,
  "phase_time_target_min": 90,
  "ui_project": false,
  "projected_phases": 4,
  "phase": 2,
  "total_phases": 4,
  "mode": "coach",
  "turns_since_last_progress": 0,
  "prereqs": {
    "git": true,
    "superpowers": true,
    "context7_skills": true,
    "context7_mcp": true,
    "stack_toolchain": true
  },
  "phases": [
    {
      "n": 1,
      "title": "Scaffold + domain",
      "status": "done",
      "started_at": "ISO-8601",
      "completed_at": "ISO-8601",
      "duration_min": 75,
      "feedback": "right",
      "last_seen_commit": "<sha or null — used by coach rule 10 progress-signal detection>",
      "todos": [
        {
          "id": "p1-t1",
          "user_writes": "impl",
          "label_diff": "easy",
          "label_kind": "bridge",
          "test_file": "internal/shortener/shortener_test.go",
          "test_name": "TestEncodeBase62_zero",
          "impl_file": "internal/shortener/shortener.go",
          "impl_func": "encodeBase62",
          "bridge_note": "Node Buffer.toString(base64url) analog, but deterministic",
          "doc_url": "https://pkg.go.dev/strings#Builder",
          "status": "done",
          "practice": true,
          "attempts": 1,
          "gave_up": false
        }
      ]
    }
  ],
  "capstone": {
    "required": true,
    "completed": false,
    "feature": "GET /stats/:code hit counter",
    "start_commit": "<git SHA at solo entry, e.g. abc1234>",
    "turns_since_last_progress": 0,
    "last_seen_commit": "<sha or null>",
    "scaffold_commits": ["<sha of each LLM commit during solo, e.g. Task C.1 stub>"]
  },
  "events": [
    {"ts": "ISO-8601", "kind": "phase_start", "phase": 1},
    {"ts": "ISO-8601", "kind": "mode_switch", "from": "build", "to": "coach"},
    {"ts": "ISO-8601", "kind": "phase_done", "phase": 1, "feedback": "right"}
  ]
}
```

## Enum fields

- `mode`: `build` | `coach` | `solo` | `review` | `done`
- `phase.status`: `pending` | `in_progress` | `done`
- `todo.status`: `pending` | `in_progress` | `done`
- `todo.user_writes`: `impl` | `test` | `both`
- `todo.label_diff`: `easy` | `medium` | `hard`
- `todo.label_kind`: `bridge` | `idiom` | `logic`
- `phase.feedback`: `too_easy` | `right` | `too_hard` | null
