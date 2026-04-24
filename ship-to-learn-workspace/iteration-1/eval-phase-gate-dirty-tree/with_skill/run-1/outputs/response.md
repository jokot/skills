Hold up — can't run the phase gate yet. All 5 practice TODOs are marked `done` in `progress.json`, which is step 1 of the gate, but step 1a (commit-clean check) fails.

`git status --porcelain` shows:

```
 M internal/shortener/shortener.go
 M internal/shortener/shortener_test.go
```

Both files are in the Phase 1 file-set (the impl file and its test file for your practice TODOs), and both have uncommitted changes. The phase gate refuses to advance on a dirty phase file-set — your Phase 1 work needs to be in git first so authorship and review scope stay clean later.

Commit your work, then re-signal "done phase 1":

```
git add internal/shortener/shortener.go internal/shortener/shortener_test.go
git commit -m "impl: phase 1 practice TODOs (p1-t1..p1-t5)"
```

If you prefer one commit per TODO (cleaner history for the final review), that works too — stage and commit each function separately. Either way, once `git status --porcelain` is empty for those two files, tell me "done phase 1" again and I'll:

1. Re-verify TODOs done (already good).
2. Re-run the clean check.
3. Run `go test ./...`.
4. If green, ask you the one difficulty question (too_easy / right / too_hard) and advance to Phase 2 build.

I'm staying in coach mode — no state changes this turn beyond noting the blocked-gate event. Ping me when committed.
