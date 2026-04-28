# Capstone — solo mode + capstone gate

> Final phase: user-only feature implementation. See [SKILL.md](./SKILL.md) for the entry point.

**What "capstone" means:** the final phase of the project, where the user demonstrates they can use the new stack without the LLM writing any code. One feature, user writes 100% of the impl and tests, LLM provides only the feature spec plus a single failing test stub. The capstone ships as part of the real project — not a throwaway exercise. Success = the user's proof-of-learning.

Only entered after the last non-capstone phase is `done` and the user confirms readiness.

## On entering `solo`

1. Confirm with user: "Ready for capstone? You will write impl *and* tests. I will only provide concept help and doc links. No code. Continue?"
2. Once confirmed: run `git status --porcelain` — if non-empty, ask user to commit or stash first (the `start_commit` must reflect a clean tree so diff-based ownership is meaningful). Then record `capstone.start_commit` by running `git rev-parse HEAD`. Any file with a diff from this commit is considered user-owned for capstone purposes — computed, not pre-declared. Initialise `capstone.turns_since_last_progress = 0` and `capstone.last_seen_commit = start_commit`.

2a. **Capstone size check.** Before writing the `## Capstone` section, project how large the feature is (mentally walk the user's scope: how many distinct behaviors, ~how much code, realistic time at their pace). If it would exceed ~90 min or ~3 logical behaviors, push back: "This capstone feature is too big for solo work. Either (a) cut scope so it fits ~90 min and 1–3 behaviors, or (b) turn this into an extra regular phase and design a smaller capstone." Do not write the section until user confirms a fitting scope.

3. Append the capstone feature spec to `plan.md` under a `## Capstone` heading with a comment `<!-- appended at solo entry -->` (this is one of the two permitted amendments per [STATE-SCHEMA.md](./STATE-SCHEMA.md) Invariants). **Base the content on `spec.md`'s `## Reserved for capstone` entry** — elaborate that one-line reservation into the full capstone spec. Do not invent a feature unrelated to the reservation; if the reservation is too vague, ask the user one question to clarify before elaborating.

   The section uses **no `[PRACTICE]` markers** — the user designs the file structure and API shape themselves. It contains:

   - **Feature description** — one short paragraph, user-perspective ("what does this feature do"). Expand from the reserved one-liner.
   - **Task C.1** — the single failing-test stub you are about to write in step 4 below, shown verbatim.
   - **Behaviors to cover** — a bullet list of 3–7 concrete behaviors the feature should exhibit (e.g. "incrementing on each redirect", "starts at zero for a new code", "thread-safe under concurrent hits"). These are the user's own test targets; no file paths, no function signatures.
   - **Docs:** — 1–3 context7 URLs relevant to the feature's stack concepts.

   Ownership at the gate is computed from `git diff <start_commit>..HEAD` minus `capstone.scaffold_commits`, not from TODO ids — so capstone needs no PRACTICE markers to work.

4. Write exactly one failing test **stub** — but first, ask the user where it should live. Say:

   > "Capstone needs one test file to hold the failing stub. Where should it live? (e.g. `internal/stats/stats_test.go`, `tests/test_stats.py`, `src/stats.test.ts`). I will create only that one file — package declaration and the stub — nothing else. You design everything else."

   After the user picks the path:
   - Create that single file with: stack-minimal preamble (package declaration, imports strictly needed for the test runner to parse it), one test function named after the top-level behavior, body = stack's unimplemented-test pattern from the build-mode marker table (e.g. Go: `t.Fatal("write this test — capstone feature in plan.md ## Capstone")`; Rust: `panic!(...)`; Python: `pytest.fail(...)`; JS: `throw new Error(...)`).
   - **No impl files, no other test files, no signature stubs, no hint comments, no TODO markers in any file other than this one stub.** Capstone is a strict subset of `user_writes: both` — stricter than regular-phase `both` mode — the user designs the full shape.
   - Commit this as `scaffold(capstone, task C.1): failing stub at <path>` before handing over to the user. Capture the commit SHA (`git rev-parse HEAD`) and append it to `progress.json.capstone.scaffold_commits`. Review will use this list to exclude LLM-authored regions from its idiom critique.

5. Set `mode: solo`. Tell the user: "Capstone is live. Any new or changed file since commit `<start_commit>` counts as your work. Commit your progress with your git identity (`<progress.json.git_identity>`). Say 'capstone done' when ready for the gate."

6. Refuse all code writes until review.

## Behavior rules in `solo`

- User asks conceptual Q: answer with explanation + doc + bridge. No code.
- User pastes code for review: critique by line reference. No rewrites.
- User asks "just write it this one time": refuse. Remind them of capstone purpose. Offer to explain the concept differently instead.
- User asks to run tests: run via bash tool, report output, interpret. No fixes.
- Detecting "stuck": the skill has no clock, only a turn counter. Track `turns_since_last_progress` in `progress.json.capstone`. After ≥10 turns of Q&A without the user signalling any forward progress, proactively check in: "You've been on this a while. Still progressing, or want to talk about giving up honestly?" Do not wait longer.
- If the user never replies (walked away): the skill is reactive, nothing happens. `progress.json` persists. On next `/ship-to-learn` invocation, state resumes from where it was. No data loss.
- If the user wants out honestly: they say "give up on capstone" / "abandon capstone" / "stop the capstone". Mark `capstone.completed: false`, transition to `review` (see [REVIEW.md](./REVIEW.md)), and record in `review.md`: "Capstone not completed solo." Skill still emits a review — the non-capstone work is still real.

## Capstone gate

Triggered when user signals "capstone done" or similar.

Actions, in order:

1. **Working-tree clean check (covers both tracked and untracked).** Run `git status --porcelain`. If output is non-empty — any modified, added, deleted, or untracked file — refuse: "Commit your capstone work first so I can verify authorship. Untracked files too: `git add <files> && git commit -m 'capstone: <msg>'`. If any file should be ignored, add it to `.gitignore` and commit that." Halt.
2. **Compute owned files.** Run `git diff --name-only <capstone.start_commit>..HEAD`. Call this set `OWNED`. If `OWNED` is empty, refuse: "No committed changes since capstone start. Commit your capstone work and re-try." Halt. (This shouldn't be reachable after step 1 passes, but acts as a belt-and-braces check.)
3. **Tests.** Run `stack.test_runner`. If fail: stay in `solo`, hint, no fix.
4. **Authorship.** For each path in `OWNED`, run `git log --format="%ae" <capstone.start_commit>..HEAD -- <path>`. Every author email must match `progress.json.git_identity`. Any mismatched author email = flag in `review.md` under "capstone non-user commits" with the file + author; do not refuse.
5. Transition to `review`. See [REVIEW.md](./REVIEW.md).
