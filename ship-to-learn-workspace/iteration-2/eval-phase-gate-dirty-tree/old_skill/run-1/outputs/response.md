# Phase 1 gate — blocked: uncommitted changes

I can't advance to phase 2 yet. Phase gate step 1a (commit-clean check) failed.

All five practice TODOs (`p1-t1` … `p1-t5`) are marked `done` in `progress.json`, so the TODO check passes. But before I run the test runner, I have to verify the phase file-set is committed. `git status --porcelain` shows:

```
 M internal/shortener/shortener.go
 M internal/shortener/shortener_test.go
```

Both files are part of phase 1's file-set (they're the impl + test files for the practice TODOs in this phase). Working tree is dirty — the gate refuses to advance.

## What to do

Commit your phase 1 work, then say "done phase 1" again. Something like:

```
git add internal/shortener/shortener.go internal/shortener/shortener_test.go
git commit -m "impl: phase 1 base62 + shortener core"
```

Use whatever commit message(s) match how you actually grouped the work — one commit per TODO is fine, one combined commit is fine too. The gate just needs the tree clean so authorship and the test run reflect committed state.

Once `git status --porcelain` is empty for those files, re-signal "done phase 1" and I'll:

1. Re-run the commit-clean check.
2. Run `go test ./...`.
3. If green, ask you the difficulty question (too_easy / right / too_hard).
4. Advance to phase 2 build.

I am staying in `coach` mode. No code from me. No test run yet — the gate halts at step 1a per the skill's "in order" rule.

If you'd rather inspect the diff first before committing, `git diff internal/shortener/shortener.go internal/shortener/shortener_test.go` will show you exactly what's pending.
