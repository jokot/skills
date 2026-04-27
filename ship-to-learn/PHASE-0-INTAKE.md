# Phase 0 — intake, spec, plan, lock

> Initial setup loop for a new ship-to-learn project. See [SKILL.md](./SKILL.md) for the entry point.

First action on `/ship-to-learn` invocation (or any trigger phrase for starting a new project):

## 1. Read `progress.json` if it exists

Three branches:

**(i) Valid and `mode != "done"` — active project.** Do not resume silently. Ask:

> "Active project detected: `<project_name>`, phase `<N>`, mode `<mode>`. What do you want? (a) resume, (b) abandon and start new (deletes `.learn/`), (c) cancel."

- (a) → resume from current phase and mode.
- (b) → require explicit confirmation ("delete .learn/ and start over — yes?"), then `rm -rf .learn/` and proceed to intake.
- (c) → exit the skill, do nothing else.

**(ii) Valid and `mode == "done"` — previous project complete.** Do not auto-start a new project, and do not touch `.learn/`. Tell the user:

> "Previous project `<project_name>` is complete. Review at `.learn/review.md`. To start a new project, move or delete `.learn/` first (e.g. `mv .learn .learn-<project_name>` to archive, or `rm -rf .learn`). Then re-run `/ship-to-learn`. (a) view review, (b) cancel."

Never auto-delete a DONE project — `review.md` is part of the artifact.

**(iii) File missing or malformed JSON** — see [SKILL.md](./SKILL.md) Interaction contract for malformed case. If missing but `.learn/` exists with `spec.md` or `plan.md` present (partial state), halt: "Partial `.learn/` state: spec or plan present without progress.json. Resolve: `rm -rf .learn/` to start fresh, or rebuild progress.json manually from spec/plan. I will not overwrite ambiguously." Only proceed to intake if `.learn/` is absent or empty.

## 2. If no `progress.json`, start intake

**Q0 — UI project check (asked first, separate turn):** "Is this a UI/frontend project (web app, dashboard, design system, any project with a significant visual layer)? y/n."

- If **yes**, verify the `ui-ux-pro-max` skill (or whichever UI/UX theming skill is configured) is installed. If missing, halt: print the install command (`npx skills@latest add <ui-ux-pro-max-repo>`), tell the user to install + restart the agent session + re-run `/ship-to-learn`. Do not proceed. Record the answer in-memory until `progress.json` is created (step 7) where it persists as `ui_project: true`.
- If **no**, skip the UI/UX skill check. Record `ui_project: false`.

**Then ask these questions one at a time.** Keep each answer short.

- Known stacks and years of experience
- Target stack and version
- Idea? (yes / no / unsure — if no, use `superpowers:brainstorming` to help; if unsure, propose 3 sized-for-learning project ideas matched to their stack)
- Time budget in hours across how many sessions
- Any scope red flags? (if the target sounds like "Twitter clone" or "build a database from scratch", push back and suggest cutting)

## 3. Create the worktree

Invoke `superpowers:using-git-worktrees` with a slug like `learn-<project-slug>` (see [COMPOSING.md](./COMPOSING.md) for sub-skill rules). The worktree may live anywhere — ship-to-learn does not enforce a parent directory; whatever `using-git-worktrees` chooses (or the user picks during its dialog) is fine. After creation, from inside the worktree:

- Run `git config user.email` — if empty, halt and ask the user to set it: `git config user.email <email>`. Retry once they confirm.
- Record the non-empty email into `progress.json.git_identity` for later capstone-commit verification.
- Record the absolute worktree path into `progress.json.worktree_path`.

## 4. Spec phase

Invoke `superpowers:brainstorming` per [COMPOSING.md](./COMPOSING.md) — **stop it after its step 8** (user reviews the spec), do not let it proceed to step 9 (which would trigger writing-plans). Overrides:

- Save design to `.learn/spec.md` (not the brainstorming default path). If the sub-skill refuses to override, let it save to its default, then move the file to `.learn/spec.md`.
- Spec must eventually include all of the following sections — brainstorming will cover the first four; step 4a below covers the rest:
  - **User profile** (from intake)
  - **Goal** (one sentence)
  - **Scope (fixed)** and **Out of scope**
  - **Time budget**

## 4a. Spec enrichment

After brainstorming returns, before `progress.json.phase` is set (the lock moment), enrich `.learn/spec.md`:

- Call `find-docs` / context7 for the target stack's current idioms and common pitfalls. Budget ≤5 calls.
- Append/enrich **Stack decisions** table with concrete choices (router, ORM, test runner, formatter) and a context7 source URL per row.
- Append **Bridges** table mapping target-stack concepts to the user's known stacks (explicit, scannable — at least 5 rows).
- Append **Created:** `<ISO date>` + `"locked"`.
- Do **not** ask the user to approve the spec yet. There are more pre-lock edits coming in step 5 (capstone reservation). Approval happens once at the end of step 5 after every spec edit is applied.

## 5. Plan phase

**Do not invoke `superpowers:writing-plans`.** Its "No Placeholders" rule (`"'TODO', 'implement later' are plan failures — never write them"`) directly conflicts with `[PRACTICE]` markers. Instead, write `.learn/plan.md` directly, following the writing-plans **format conventions** (so plans stay recognizable to tooling):

- Header: `# <Feature> Implementation Plan`, `**Goal:**`, `**Architecture:**`, `**Tech Stack:**`.
- `## File Structure` section listing every file the plan creates/modifies with a one-line responsibility.
- Bite-sized tasks grouped per phase, 2–5 minutes each, TDD cycle (write failing test → run → impl → run → commit).
- **Phase count = regular (non-capstone) phases only.** Formula: `total_phases = ceil(time_budget_hours * 60 / phase_time_target_min)` where `phase_time_target_min` defaults to 90. **The capstone is always an additional final phase on top**, not counted in `total_phases`. It is designed later at solo entry and appended to `plan.md` via the `## Capstone` amendment — do not include a capstone phase in this initial plan write.
- If `total_phases > 10`, warn the user. Offer either proceeding as-is or splitting into sub-projects (per `superpowers:brainstorming`'s scope-check convention). User chooses.
- Each phase written here: 5 practice TODOs distributed **2 easy + 2 medium + 1 hard**. Mark each with the `[PRACTICE]` marker below.
- TODO `user_writes` progression across phases: see "TODO progression" below.
- **Do not include any "Execution Handoff" section** (writing-plans convention). Ship-to-learn owns execution.
- `progress.json.capstone.required = true` means a capstone phase will run after phase `total_phases` completes. Solo-entry logic handles its creation; initial plan.md ends at `## Phase <total_phases>:`.
- **Reserve a feature for capstone.** After drafting phases 1..`total_phases`, inspect `spec.md` scope. If every scope item is covered by the phases (nothing left for capstone), prompt the user: "Your scope is fully planned by phase `<total_phases>`. Capstone needs one additional small feature (~90 min, 1–3 behaviors). Propose one — examples for your project: `<2–3 suggestions derived from spec>`. I'll add it to spec.md's scope under `## Reserved for capstone` and keep it out of the regular phases." Append the user-approved feature to `spec.md` as a `## Reserved for capstone` section (allowed — still pre-lock). Do not lock progress.json until this reservation exists.
- **Plan.md footer.** End `plan.md` with a horizontal rule and note:

  ```
  ---

  **Note:** After phase <total_phases> completes, ship-to-learn prompts for the **capstone phase** — one additional feature you ship solo (you write impl + tests, LLM coaches only). The reserved feature is recorded in `.learn/spec.md` under `## Reserved for capstone`. Designed in detail and appended here as `## Capstone` at solo entry.
  ```

- **Spec approval — final, once.** Now that spec.md is fully enriched (step 4a stack decisions + bridges + Created, plus step 5 reservation) and plan.md is drafted, present **both files** to the user for approval in a single review. If they reject any section, ask which, edit inline in the relevant file (still pre-lock), loop until approved. Do NOT re-invoke `superpowers:brainstorming` — that would reset the dialog. Only after both files are approved do you proceed to step 5a.

## 5a. UI/UX theming (conditional — only if `ui_project == true`)

Invoke the configured UI/UX theming skill (e.g. `ui-ux-pro-max`) per [COMPOSING.md](./COMPOSING.md). Let it drive its internal dialog (theme tokens, palette, component inventory, typography) to completion. When it returns:

- Write its output to `.learn/theme.md` (move or create as needed).
- Amend `plan.md`'s File Structure section to include new theme-related files (token file, component directory, etc.).
- **Append theme-setup steps to Task 1.1 only** (Phase 1's phase-wide LLM scaffolding task). They become part of Task 1.1's single `scaffold(phase 1, task 1.1):` commit. Do **not** create new Tasks or renumber practice TODOs (1.2–1.6 stay). Task 1.1 gets bigger; everything else stays.
- Re-present the updated `plan.md` + `theme.md` for a second quick approval. Loop until approved.
- If `ui_project == false`, skip this step entirely.

## 6. Toolchain check

Verify the target-stack toolchain is installed by running the stack's version command via bash (e.g. `go version`, `cargo --version`, `python3 --version`, `node --version`). If the command fails, halt: print the failing command, tell the user to install the toolchain, and **do not proceed to build**. This must happen before any file scaffolding because phase 1 `build` runs the test runner using this toolchain.

**Version reconciliation.** Compare the detected version to the version the user named during intake (and that you recorded in `.learn/spec.md` stack decisions). If they differ (e.g. user said "Go 1.22", detected "go1.21.5"), ask once: "Installed is `<detected>`; your spec says `<claimed>`. Use installed, or install `<claimed>` first?" Update `progress.json.stack.version` and `spec.md` stack decisions to match the version that will actually be used. Do this before the lock moment.

## 7. Create `progress.json`

Reflect the above. Set `phase: 1` (this is the lock moment for spec.md and plan.md per the [STATE-SCHEMA.md](./STATE-SCHEMA.md) Invariants section). `mode: "intake"` is not a mode — jump straight to `mode: "build"` once plan is approved and toolchain check passes.

## 8. Transition to `build` for phase 1

See [PHASE-LOOP.md](./PHASE-LOOP.md) for the build/coach/gate loop.

---

## `plan.md` structure — required template

Every `plan.md` ship-to-learn writes must follow this layout exactly. Consistency lets resume + review + gate all parse reliably.

```markdown
# <Project Name> Implementation Plan

**Goal:** <one sentence>

**Architecture:** <2–3 sentences>

**Tech Stack:** <language>, <router/framework>, <DB>, <test runner>

---

## File Structure

- Create: `path/to/file.ext` — <one-line responsibility>
- Create: `path/to/other.ext` — ...
- Test: `path/to/test.ext` — <what it covers>
- ...

---

## Phase 1: <title>

**Time target:** 90 min
**Difficulty mix:** 2 easy + 2 medium + 1 hard
**`user_writes` mix:** 5 × impl

### Task 1.1: <LLM-authored task, e.g. "Scaffold module X">

**Files:**
- Create: `path/to/file.ext`

- [ ] **Step 1: Create file with skeleton**

\`\`\`<lang>
// full working code
\`\`\`

- [ ] **Step 2: Commit**

Run: `git add <files> && git commit -m "scaffold(phase 1): module X"`

### Task 1.2: <Practice task — user fills>

**Files:**
- Modify: `path/to/file.ext:<line>` — function `<name>`
- Test: `path/to/file_test.ext`

- [ ] **Step 1: Write failing test (LLM)**

\`\`\`<lang>
// full test code, LLM-authored
\`\`\`

- [ ] **Step 2: Run test, verify failure**

Run: `<test-runner>`
Expected: FAIL with "unimplemented" panic.

- [ ] **Step 2a: Commit (LLM) — `scaffold(phase 1, task 1.2): failing test for <func>`**

Run: `git add <test-file> <impl-file> && git commit -m "scaffold(phase 1, task 1.2): failing test for <func>"`
(This commit includes the failing test and the unimplemented-marker stub in the impl file. Author boundary before user work begins.)

- [ ] **Step 3: Implement <func> [PRACTICE — p1-t1 | easy|bridge | user_writes=impl]**

  **Goal:** <one-sentence behavior>
  **Target file:** `path/to/file.ext` — function `<name>`
  **Bridge from known stack:** <analogy>
  **Idiom note:** <relevant target-stack convention>
  **Docs:** <url from find-docs / context7>

  No code block. Fill in impl to make the referenced failing test pass.

- [ ] **Step 4: Run test, verify pass**

Run: `<test-runner>`
Expected: PASS.

- [ ] **Step 5: Commit (user) — `impl: <func>`**

Run: `git add <files> && git commit -m "impl: <func>"`

### Task 1.3..1.N: ...

---

## Phase 2: ...
## Phase 3: ...
## Phase 4: ... (last non-capstone)

---

<!-- Capstone section appended by ship-to-learn at solo entry -->
<!-- Re-plan sections appended only on pacing trigger -->
```

Rules:

- **One Task per practice TODO.** For phase N: Task N.1 is phase-wide LLM scaffolding (writing supporting files shared across TODOs); Tasks N.2 through N.(k+1) are one per practice TODO in order, k = TODO count for that phase (5 for regular phases). Capstone has a single Task C.1 containing only the failing-test stub; everything else is user-authored and implicit in the `## Capstone` section.
- Every `[PRACTICE]` step appears as a TDD sub-step surrounded by LLM-authored steps that emit the failing test (for `user_writes: impl`), the working impl (for `user_writes: test`), or neither (for `user_writes: both` — only a failing-test stub exists).
- LLM-authored steps always include full code blocks.
- Practice steps replace the code block with the spec shown above. The `todo-id` must match `progress.json[phases[i].todos[j].id]`.
- Commit steps always name the author: `scaffold(...)` / `solve(...)` / `fix(...)` (LLM) vs `impl: ...` / `test: ...` (user). Author names help the capstone gate + review distinguish regions later.
- A `Files:` block appears on every task so that resume-on-build and phase-gate commit-check can parse expected files deterministically.
- If `plan.md` exceeds ~1500 lines, optionally split: keep `plan.md` as the index and move per-phase content into `.learn/plans/phase-N.md` files. Reference them from `plan.md` headings. For v1 projects this is rare — do not split unless you hit the size.

## `[PRACTICE]` marker — exact syntax

Inside a task's step list, the user-authored step uses this exact syntax:

```markdown
- [ ] **Step N: <title> [PRACTICE — <todo-id> | <label_diff>|<label_kind> | user_writes=<impl|test|both>]**

  **Goal:** <one-sentence behavior>
  **Target file:** `<path>` — function/section `<name>`
  **Bridge from known stack:** <analogy>
  **Idiom note:** <relevant target-stack convention>
  **Docs:** <url from find-docs / context7>

  <for user_writes: test ONLY, also include:>
  **Behaviors to cover:**
  - <concrete behavior 1, e.g. "zero input returns empty">
  - <concrete behavior 2, e.g. "collision case returns error">
  - <concrete behavior 3, e.g. "max-length input handled">
  (at least 3)

  No code block. <For user_writes=impl: fill the impl to pass the referenced failing test. For user_writes=test: write tests covering Behaviors above. For user_writes=both: design both.>
```

## TODO `user_writes` progression (default)

| Phase | Mix |
|---|---|
| 1 | 5 × impl |
| 2 | 3 × impl + 2 × test |
| 3 | 2 × impl + 2 × test + 1 × both |
| 4+ | 1 × impl + 2 × test + 2 × both |
| Capstone | all TODOs: `both` (impl + tests, zero LLM code); 1–3 TODOs total, aim for ~90 min; if more needed, the capstone spec is too big — push back in intake |

The user may override in intake ("I already know testing; skip test-writing"). Record override in `progress.json` and adjust.
