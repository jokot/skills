# State updates for Turn 1

On this turn, ship-to-learn is in Phase 0 pre-intake. The following state operations would be performed, in order:

## 1. Read `progress.json` (Interaction contract rule 1)

- Attempt: read `/tmp/learn-rust-cli/.learn/progress.json`.
- Result: file does not exist (the whole `.learn/` directory is absent).
- Branch: Phase 0 step 1 (iii), sub-case "missing and `.learn/` absent or empty" → proceed to intake (step 2).

## 2. Partial-state halt check

- Check: does `.learn/` exist with `spec.md` or `plan.md` but no `progress.json`?
- Result: `.learn/` does not exist at all. No partial-state halt needed.

## 3. Hard prereq detection (run before anything else)

Per the "Hard prerequisites" table and "Detecting prereqs per agent" rows:

| Prereq | Method | Result |
|---|---|---|
| `git` | `bash: git --version` → exit 0 | present |
| `superpowers:brainstorming` | skill listing includes it | present |
| `superpowers:using-git-worktrees` | skill listing includes it | present |
| `find-docs` (context7 skills) | skill listing includes it | present |
| context7 MCP | tool name `mcp__plugin_context7_context7__query-docs` in tool list | present |

Stack toolchain (`cargo`) is runtime-tier — deferred until step 6 (pre-build). `ui-ux-pro-max` is hard-conditional on Q0 answer — deferred to after Q0 response.

No prereq failures → no halt. Proceed to Phase 0 step 2 (intake).

## 4. No file writes this turn

Explicitly NOT doing any of the following on turn 1:

- Do **not** create `.learn/` directory yet. `.learn/` is created after the worktree exists, not in the host repo (Phase 0 step 3 creates the worktree first).
- Do **not** create `progress.json` yet. Per Phase 0 step 7, `progress.json` is only written once intake is complete, spec is enriched, plan is drafted and approved, UI theming (if any) is done, and toolchain check passes. At that moment `phase: 1` is set (the lock moment).
- Do **not** invoke `superpowers:using-git-worktrees` yet. That happens at Phase 0 step 3, after all intake questions (Q0 + the five bullet questions) are answered.
- Do **not** invoke `superpowers:brainstorming` yet. That happens at Phase 0 step 4, after the worktree exists.
- Do **not** call `find-docs` / context7 yet. Those calls are budgeted for Phase 0 step 4a (spec enrichment) and later.

## 5. In-memory state (not yet persisted)

The only state captured this turn is held in-memory for the duration of intake, to be folded into `progress.json` at the step 7 lock moment:

- `prereqs.git = true`
- `prereqs.superpowers = true`
- `prereqs.context7_skills = true`
- `prereqs.context7_mcp = true`
- `prereqs.stack_toolchain` — unknown, check deferred to step 6
- Target stack (tentative, from user's free-text intent): `rust` — will be confirmed and versioned during intake
- Project shape (tentative): CLI tool — will be refined during intake

None of this is written to disk on turn 1.

## 6. Ask Q0 (Phase 0 step 2)

Per the skill's explicit rule: **"Q0 — UI project check (asked first, separate turn)"**. Response to user is the Q0 prompt only. The remaining five intake questions (known stacks, target version, idea, time budget, scope red flags) are deferred to subsequent turns — they are asked one at a time per the skill, not batched with Q0.

## 7. Next-turn expectations

- If user answers Q0 = "n" → record `ui_project = false` (in-memory), skip ui-ux-pro-max check, ask the next intake question (known stacks + years).
- If user answers Q0 = "y" → verify `ui-ux-pro-max` skill is installed; if missing, halt with the install command and restart instruction. If present, record `ui_project = true` (in-memory), then ask the next intake question.

No worktree, no `.learn/`, no `progress.json`, no sub-skill invocations on this turn.
