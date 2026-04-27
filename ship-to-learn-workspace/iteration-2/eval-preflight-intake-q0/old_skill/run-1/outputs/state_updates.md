# State updates — turn 1

## Filesystem changes

**None.** No directories created, no files written.

Per Phase 0 step 1 (Interaction contract rule 1): read `progress.json` if it exists. It does not. `.learn/` itself does not exist either, and the parent dir (`/tmp/learn-rust-cli`) is a clean initialized repo with no `spec.md` / `plan.md` partial state — so the "partial state" halt branch does not trigger.

`.learn/` is **not** created on turn 1. The skill defers creation of `.learn/` and `progress.json` until step 7 of Phase 0 (after spec.md is written, plan.md is drafted, both are user-approved, optional theme is captured, and toolchain check passes). On turn 1 we are still gathering Q0 + intake answers in conversation only.

## In-memory state (held until `progress.json` is created at step 7)

| Field | Value |
|---|---|
| `prereqs.git` | true |
| `prereqs.superpowers` | true |
| `prereqs.context7_skills` | true |
| `prereqs.context7_mcp` | true |
| `prereqs.stack_toolchain` | unverified (Rust `cargo` check deferred to step 6) |
| `ui_project` | **pending — awaiting user answer to Q0** |
| Intake stage | `Q0 asked, awaiting reply` |
| Other intake answers (known stacks, target version, idea, time budget, scope flags) | not yet asked — they come one at a time **after** Q0 resolves |
| Mode | none yet (intake is not a mode; per step 7, mode jumps directly to `build` once spec + plan + toolchain are done) |

## Sub-skill invocations

None this turn. `superpowers:brainstorming` is invoked at Phase 0 step 4, after Q0 + intake answers are collected and the worktree is created.

## Events

None persisted (no `progress.json` exists yet to receive events). The first `event` (`phase_start` for phase 1) will be logged at the lock moment in step 7.

## Next turn — expected

User replies to Q0 (y/n).

- If **y**: verify a UI/UX theming skill is installed; if missing, halt with the install command and instruction to restart the agent + re-run `/ship-to-learn`. If installed, record `ui_project: true` in-memory and ask the first non-UI intake question (known stacks + years).
- If **n**: record `ui_project: false` in-memory, skip UI/UX skill check, ask the first intake question (known stacks + years).

Either way, intake questions then proceed **one at a time** per the skill's Phase 0 step 2 rule.
