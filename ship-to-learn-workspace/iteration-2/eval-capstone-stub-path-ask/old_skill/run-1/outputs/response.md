Capstone confirmed. Entering `solo` mode.

Before I touch anything, two prep checks:

**1. Working tree must be clean.** I need to capture `capstone.start_commit` from a clean tree so the gate can compute "what did the user author" via diff. Run:

```
git status --porcelain
```

If it prints anything, commit or stash first, then tell me you're clean.

**2. Capstone size sanity-check.** The reserved feature in `spec.md` is:

> Hit counter per short code. Increments on each redirect.

That sizes to ~90 min and 3 behaviors (increment on each redirect, starts at zero for a new code, safe under concurrent hits). Fits the capstone budget — proceeding.

---

Once you confirm the tree is clean, here's what happens next, in order:

1. I record `capstone.start_commit = git rev-parse HEAD` and initialize `capstone.turns_since_last_progress = 0`.
2. I append a `## Capstone` section to `.learn/plan.md` (one of the two permitted plan amendments) elaborating the reserved feature into a full spec — feature description, Task C.1 (the single failing-test stub I'll write), Behaviors to cover (3–7 bullets), and a Docs line with context7 URLs. **No `[PRACTICE]` markers, no file paths, no function signatures.** You design the shape.
3. **I need one decision from you before I write any code: where should the single failing-test stub live?**

   Capstone gets exactly one LLM-authored file: a test file containing a package/module declaration and one test function whose body is the stack's unimplemented-test pattern (for Go: `t.Fatal("write this test — capstone feature in plan.md ## Capstone")`). No impl files. No other test files. No signature stubs. No hint comments anywhere else. You design everything else — the stats package layout, the counter API, the storage approach, all of it.

   Pick a path. Examples for a Go URL-shortener:
   - `internal/stats/stats_test.go`
   - `internal/shortener/stats_test.go` (if you want stats to live inside shortener)
   - `pkg/stats/stats_test.go`

   Whatever you pick, I'll create only that one file, commit it as `scaffold(capstone, task C.1): failing stub at <your-path>`, record the commit SHA into `capstone.scaffold_commits` (review excludes it from your idiom critique later), and hand the keyboard over.

4. After the stub commit, I switch to `mode: solo`. From that point: any new or changed file since `start_commit` counts as your work. Commit your progress with your git identity. When ready, say **"capstone done"** and I'll run the capstone gate (clean tree → diff for owned files → tests → authorship check).

Reminder of the rules in solo, since they're stricter than coach:

- I will not write any code. No syntax-demo escape hatch this time.
- I will explain concepts, point to docs (cited via context7), and interpret test output.
- If you paste code and ask "is this right", I'll critique by line reference — no rewrites.
- If you ask me to "just write it this once", I'll refuse and remind you why.
- If you genuinely want out, say "give up on capstone" — I mark `capstone.completed: false`, transition to review, and review.md notes it. The non-capstone work is still real.

Reply with:
1. confirmation the working tree is clean (or "clean now" after you commit/stash), and
2. the path for the single failing-test stub.
