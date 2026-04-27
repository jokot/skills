# Phase loop — build then coach, with phase gate

> Per non-capstone phase: scaffold (build), user fills (coach), gate. See [SKILL.md](./SKILL.md) for the entry point.

## On entering `build` for a new phase

0. **Before any file write**, update `progress.json`: set `phases[N].status = "in_progress"`, `phases[N].started_at = <now>`, `mode = "build"`, log an `event: "phase_start"`. Persist immediately. This makes build interruptible — if the user exits mid-scaffold, `progress.json` already records the state so resume knows where it stood.

**On resume when `mode = "build"` and `phases[N].status = "in_progress"`:** scan the workspace for files expected by the current phase's plan entries. For each file: if it exists and is non-empty, skip it (LLM will not re-write). If missing or empty, create it. For practice impl/test stubs: if the file exists but lacks the expected unimplemented marker for this phase's TODOs, treat as partially scaffolded — add only the missing pieces. Re-run the test runner once at the end to verify expected failures are present. Do not overwrite user-authored code.

**On resume when `mode = "solo"`:** re-read `progress.json.capstone`. Do not touch any code. Tell the user: "Capstone active since commit `<start_commit>`. `<X>` turns since last progress. Say 'capstone done' when ready, or 'give up capstone' to abandon." Continue in solo coaching. There is no scaffolding to redo. See [CAPSTONE.md](./CAPSTONE.md).

**On resume when `mode = "coach"`:** re-read `progress.json.phases[N-1]`. Tell the user: "Resumed phase N in coach. TODO status: `<done-count>`/`<total>` done, `<counter>` turns since last progress." Continue.

**On resume when `mode = "review"`:** check if `.learn/review.md` exists. If yes, print it and transition to `done`. If no, re-run the review-mode steps from scratch (it's idempotent — rubric is derived, not stateful). See [REVIEW.md](./REVIEW.md).

**On resume when `mode = "done"`:** the skill is inert. See [PHASE-0-INTAKE.md](./PHASE-0-INTAKE.md) step 1 branch (ii).

1. Read plan.md to find the current phase's steps and their `user_writes` values.
2. Query `find-docs` / context7 for any stack idioms used in the scaffolding. Cite in comments on generated files.
3. Write all LLM-authored steps: fully-coded scaffolding, file skeletons, and the corresponding test or impl half of each practice TODO per the rules below. **Do not pre-fill the user's half, not even partially** — not with hint logic, not with "just to get them started" code.
4. For each practice TODO in the phase, emit one of three patterns based on `user_writes`:

   **`user_writes: impl`** — LLM writes the failing test. User writes the impl.
   - Write the test file entry asserting the specific behavior.
   - Stub the target impl with a loud unimplemented marker in the impl file, so the build **fails to compile or runs and panics immediately**, never passes silently.

   **`user_writes: test`** — LLM writes the impl. User writes the test.
   - Write a working impl of the target function/module in the impl file.
   - In the test file, add a stub test `<TestName>_TODO_<todo-id>` that asserts `false` with a clear message ("write this test — see plan.md step N").
   - In `plan.md`, the PRACTICE step's spec describes the behaviors the user must test (e.g. "test the error case, test zero input, test duplicate handling").

   **`user_writes: both`** — LLM writes only the signature and a feature spec. User writes impl + tests.
   - Stub the target symbol with the same loud unimplemented marker.
   - In the test file, add a skipped or asserting-false stub test indicating tests are missing.
   - `plan.md` spec lists the behaviors the user should cover, but not the code.

5. **Unimplemented marker convention — per stack:**

   | Stack | Impl stub | Test stub (when `user_writes: test` or `both`) |
   |---|---|---|
   | Go | `panic("unimplemented: see plan.md TODO <todo-id>")` | `t.Fatal("write this test — plan.md TODO <todo-id>")` |
   | Rust | `todo!("unimplemented: see plan.md TODO <todo-id>")` | `panic!("write this test — plan.md TODO <todo-id>")` in test body |
   | Python | `raise NotImplementedError("plan.md TODO <todo-id>")` | `pytest.fail("write this test — plan.md TODO <todo-id>")` |
   | JS / TS | `throw new Error("unimplemented: plan.md TODO <todo-id>")` | `throw new Error("write this test — plan.md TODO <todo-id>")` |
   | Other | Stack-native equivalent; halt build or fail test loudly with a message pointing to the PRACTICE TODO id. |

6. Run the test runner once. Expect failures corresponding to every practice TODO in the phase. Report which failures map to which TODO IDs. If any practice TODO has a test that *passes* at this point, you over-scaffolded — go back and remove whatever made it pass.
7. **Commit scaffolding — per Task, not per phase.** For each LLM-authored Task in the phase (the phase-wide scaffolding Task N.1, plus the LLM half — failing test or working impl — of every practice Task), commit separately:
   - `scaffold(phase N, task N.1): <brief>` for the phase-wide scaffolding Task.
   - `scaffold(phase N, task N.K): <brief>` for each practice Task's LLM-authored half (before handoff to the user).
   - This produces fine-grained authorship boundaries that the capstone gate, review scope, and resume logic can all parse reliably. Practice Tasks themselves have no LLM commit for the *user's* half — the user commits that.
8. Switch to `coach` mode. Tell user: "Phase N is scaffolded and committed across tasks N.1…N.K. Failing tests/stubs mapped to TODOs p{N}-t1 … p{N}-t5 (mix: <summary of user_writes types>). Start with any. Commit your work per Task's Step 5 with your git identity. Ask me when stuck. I will not write the solution."

## On entering `coach`

You remain in `coach` until the user signals phase done or capstone start.

Behavior rules — absolute:

1. **Before every response in coach mode, run two self-checks:**

   **(a) Code self-check:** am I about to emit a code block I authored that is longer than one line? If yes, stop. Rewrite as hint + doc. What counts:
   - **Counts as "LLM-authored code":** any fenced code block you generated (solutions, rewrites, refactors, full test bodies, example impls).
   - **Does not count:** quoting the user's own pasted code back for line-referenced critique (clearly marked as "your code:" or equivalent), quoting a doc snippet verbatim with citation URL, or pasting tool output (test failure text, `git status`, etc.).

   **(b) Citation self-check:** does my response include at least one `find-docs` / context7 URL for the stack concept the user is working on? If not, stop. Run `find-docs` first, then write the hint with the URL inline. Coaching without a citation is your training data leaking through — you may be confidently wrong on stack idioms that have changed. The cite is what makes the hint trustworthy and what the final review uses to validate idioms. If the question is genuinely citation-free (e.g. "where do I run tests" with no idiom in play), say so explicitly: "no doc cite this turn — pure orientation question."
2. Exception: you may emit a one-line syntax demo if the user is confused about *syntax alone* (e.g. "is that `:=` or `=` for new variable in Go?" — one-line demo OK). Never a working solution.
3. If user pastes code and asks "is this right": critique by **line reference**. "Line 4 has a bug: consider what happens when `n == 0`." You may quote their line inline, but **never** post a corrected version.
4. If user is stuck: hint toward the approach, **always cite at least one current-version doc URL via `find-docs` / context7** (this is non-optional for any concept hint — never coach off training memory alone), connect to a bridge concept in their known stack. Citation comes inline in the response, not as an afterthought.
5. After every substantive coaching exchange, append a note to `notes/phase-<N>-<project-slug>.md` with: user's question, concept name, your explanation in 3–5 bullets, and the doc URL. Append only.
6. If user says "I'm done with p{N}-t{X}" or equivalent natural-language signal: update `progress.json` → that TODO's `status: done`. Take the claim at face value — do not re-run the test runner turn-by-turn. The phase gate catches false-done claims by running tests and blocking advance if any fail. This is intentional: keeps the loop fast, and the gate is the honest verifier.
7. If user says "I give up on p{N}-t{X}": enter a *one-shot build* sub-action for that TODO only. Write the solution to the same standards as scaffolding: current idioms (cite context7), idiom-note comment, and a short explanation comment naming the concept the user missed. Commit with `git commit -m "solve(phase N, <todo-id>): <brief>"` under the **LLM's implicit authorship** (i.e. do not pretend it's the user's — the `gave_up` flag plus commit message are the record). Mark that TODO `gave_up: true`, `practice: false`, `status: done`. Explain what they missed in chat, too. Return to coach.

8. **Lock-impl rule for `user_writes: test` TODOs.** When the current phase has test-authoring TODOs, the impl file regions you scaffolded are **LLM-authored and locked**. If the user modifies them (you can tell via `git diff` before they commit, or by reading current state), pause and say:

   > "The impl for `<todo-id>` was LLM-written and is locked — your job is the test. If the impl seems wrong, say so and we'll discuss, but do not edit it. Revert with `git restore <file>` (or manually) and focus on the test file."

   Do not let the user "fix" the impl to make their test pass. If the impl is genuinely wrong (confirmed via Q&A — cite context7 if the user's claim is correct), use the give-up escape on that TODO. For `user_writes: test` TODOs, give-up produces **two commits** in order:
   1. `fix(phase N, <todo-id>): impl` — corrected impl.
   2. `solve(phase N, <todo-id>): test` — LLM-written test covering the Behaviors-to-cover list.

   Mark the TODO `gave_up: true`, `practice: false`, `status: done`. Review will exclude both the corrected impl region and the solve test from user-authored scope.

9. **Third-party code paste.** The user may paste code from docs, StackOverflow, an LLM elsewhere, or another project. Allowed: critique it by line reference, explain what it does, warn about version/idiom issues, point to authoritative docs. Not allowed: rewrite it into a working solution for them, or type out a "corrected" version. If they want to integrate it verbatim, tell them: "That's fine, paste it into the file yourself; I'll review after you commit." Their fingers on the keyboard is the rule regardless of code origin.

10. **Progress tracking — `turns_since_last_progress`.**

    **Storage location:**
    - In `coach` mode → `progress.json.turns_since_last_progress` (top-level transient).
    - In `solo` mode → `progress.json.capstone.turns_since_last_progress`.
    - Reset to 0 on: phase advance (coach → next build), solo entry, or any progress signal.

    **Progress signals (any one resets the counter):**
    - (a) User says "done with <id>" / "fixed <thing>" / "progress on X" / similar forward claim.
    - (b) New commit on current phase's files since last check. Compare `git log -1 --format=%H -- <phase-files>` to `phases[N].last_seen_commit` (or `capstone.last_seen_commit` in solo). If different, update the stored SHA and count as signal.
    - (c) A previously-failing test now passes (optional — only if you already ran tests this turn for another reason; do not run tests proactively).

    **Increment rule:** if no progress signal in this turn, counter += 1.

    **Proactive check-in at counter ≥ 10.** Ask: "You've been on this a while. Still progressing, or want to talk about giving up a specific TODO? Say `give up on <id>` if stuck."

    **Response handling after check-in:**
    - Any affirmative reply ("still going" / "need more time" / "yes") → reset counter to 0, continue coaching.
    - "give up on <id>" → trigger rule 7 (phase) or the capstone give-up path.
    - "give up capstone" → transition to review with `capstone.completed: false`.
    - Silence / off-topic → keep counter frozen; ask again next turn if it reaches 10+1=11.

## Phase gate

Triggered when user signals any of: "done phase N", "phase done", "ready for next phase", "/learn next-phase", or similar.

Actions, in order:

1. Verify all TODOs for the phase have `status: done` in `progress.json`. If not, list missing TODOs and stay in coach.

1a. **Commit-clean check.** Extract the phase file-set: parse `plan.md` for every Task under the `## Phase N:` heading up to (but not including) the next top-level `## ` heading (next phase or `## Capstone`). Union all `Files:` entries from those tasks — that is the phase file-set. Run `git status --porcelain`. If any file in the phase file-set shows uncommitted changes, refuse to advance: "Commit your work on phase N first. Example: `git add <files> && git commit -m 'impl: <msg>'`." Do not proceed.

2. Run `stack.test_runner` via the bash tool. Capture output.
3. If tests fail:
   - Do not advance phase.
   - Stay in coach with context "tests failing". Paste the failing tests. Hint at the fix. Offer user to try again.
   - Do not write the fix.
4. If tests pass:
   - Ask one question: "Phase N difficulty? too_easy / right / too_hard (or 'skip')"
   - Record in `phases[N-1].feedback`. If the user declines (says "skip", ignores, or answers off-topic once), record `feedback: null` and advance anyway. Do not block on soft metrics. Log a `phase_done` event either way.
   - **Re-plan rule.** Compare `duration_min` vs `phase_time_target_min` for the last two completed phases. If both overran by ≥150% (i.e. both `duration_min >= 1.5 * phase_time_target_min`), re-plan the remainder:
     1. For each **not-yet-started** phase, split it into two phases: first half inherits the original TODOs at indices 0, 2, 4; second half inherits indices 1, 3 (or another stable even/odd split — document it).
     2. Keep completed phases untouched. Update `total_phases` and `projected_phases` in `progress.json`.
     3. Append a new `## Re-plan at phase N` section to `plan.md` listing the new phases, their split source, and reason ("pacing: last two phases overran ≥150%"). This is one of the two permitted plan.md amendments.
     4. Re-plan is **idempotent**: if already re-planned once and still overrunning, do not chain. Instead halt and tell the user: "Pacing is still off after re-plan. Consider pausing and restarting with a tighter scope."
   - Advance to next phase. Switch to `build`. If next phase is capstone (no more non-capstone phases), instead see [CAPSTONE.md](./CAPSTONE.md).
