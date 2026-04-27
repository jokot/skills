Hold up — phase gate can't advance yet.

All five TODOs for phase 1 are marked `done` in `progress.json`, so the TODO check (PHASE-LOOP.md gate step 1) passes. But before I run the test runner, the gate requires a **commit-clean check** (step 1a), and right now `git status --porcelain` shows uncommitted changes:

```
 M internal/shortener/shortener.go
 M internal/shortener/shortener_test.go
```

Both files are part of phase 1's file-set (per `plan.md` Task entries under `## Phase 1:`). I will not run `go test ./...` or advance until those are committed — otherwise the "phase 1 passed" signal in git history would point at a tree that doesn't match what was tested, and the capstone gate / review scope logic would later misattribute authorship.

**Do this first:**

```bash
git add internal/shortener/shortener.go internal/shortener/shortener_test.go
git commit -m "impl(phase 1): finish p1-t1..p1-t5"
```

Use whatever message reflects what's actually in the diff — one commit or split per TODO, your call. The gate just needs a clean working tree on the phase file-set.

Once committed, say "done phase 1" again and I will:

1. Re-run `git status --porcelain` to confirm clean on the phase file-set.
2. Run `go test ./...` (your `stack.test_runner`).
3. If green: ask the difficulty question (too_easy / right / too_hard / skip), record `phases[0].feedback`, log `phase_done`, advance to phase 2 build.
4. If red: paste failing tests, stay in coach, hint at the fix — I won't write it.

Staying in coach. No `progress.json` mutation this turn beyond what's noted in `state_updates.md` (nothing material — gate blocked before any state change).
