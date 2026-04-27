Locked in. Entering capstone setup.

## 1. Working-tree check (capstone start)

Before I record the start commit, the tree must be clean — every diff after this point counts as your capstone work.

```bash
git status --porcelain
```

If that prints anything, commit or stash first, then say "ready" again. Assuming clean, I'll record:

```bash
git rev-parse HEAD   # → capstone.start_commit
```

I'll also initialize `capstone.turns_since_last_progress = 0` and `capstone.last_seen_commit = start_commit` in `progress.json`.

## 2. Capstone size check

Your `spec.md` reserves: *"Hit counter per short code. Increments on each redirect."*

Walking the scope mentally:
- Add a counter store (in-memory map keyed by short code, or extend whatever store you already have).
- Increment on the redirect path.
- Expose a way to read the count (likely a `GET /stats/:code` endpoint, since you've already wired routing in earlier phases).
- 3-5 tests covering the behaviors below.

That's ~1-3 behaviors and well under 90 min at your pace from earlier phases. Size is fine — proceeding.

## 3. Appending `## Capstone` to `plan.md`

This is one of the two permitted post-lock amendments to `plan.md` (per STATE-SCHEMA.md invariants). I'll append the following block, then stop touching plan.md for the rest of the project:

```markdown
## Capstone
<!-- appended at solo entry -->

### Feature
A redirect-hit counter. Every time a short code is resolved and redirected, its hit count increments. The current count for any short code is readable on demand. The counter persists across redirects within a single process lifetime; durability beyond that is out of scope.

### Task C.1
A single failing test stub at the path you choose in the next message. Body uses Go's unimplemented-test pattern: `t.Fatal("write this test — capstone feature in plan.md ## Capstone")`. You design every other file, signature, and test.

### Behaviors to cover
- A brand-new short code reports a hit count of zero before any redirect.
- Each successful redirect for a code increments that code's count by exactly one.
- Counts are independent per short code (incrementing code A does not affect code B).
- Reading a non-existent short code's count is handled deliberately (return zero, or a not-found signal — your call, but be consistent and tested).
- Concurrent redirects to the same code do not lose increments (thread-safe under contention).

### Docs
- `sync` package overview (Mutex, atomic counters): https://pkg.go.dev/sync
- `sync/atomic` for lock-free counters: https://pkg.go.dev/sync/atomic
- `net/http/httptest` for exercising the redirect handler in tests: https://pkg.go.dev/net/http/httptest
```

(Doc URLs are placeholders pending a `find-docs` lookup against your locked stack version — I'll cite the resolved ones in the actual append.)

No `[PRACTICE]` markers. You design the file structure, the API shape, the storage type, the concurrency primitive — all of it. Ownership at the gate is computed from `git diff <start_commit>..HEAD` minus my one scaffold commit, so the capstone needs no per-TODO tagging to work.

## 4. Where should the failing test stub live?

Capstone needs exactly one test file from me to hold the failing stub. **You pick the path.** I'll create only that one file: package declaration, the imports the Go test runner strictly needs to parse it, and one test function whose body is `t.Fatal("write this test — capstone feature in plan.md ## Capstone")`. Nothing else. No impl file, no signature stubs, no hint comments, no TODO markers anywhere else.

Examples in the shape of your existing layout:
- `internal/stats/stats_test.go`
- `internal/shortener/hits_test.go`
- `internal/redirect/counter_test.go`

Tell me the path and I'll create the stub, commit it as `scaffold(capstone, task C.1): failing stub at <path>`, capture the SHA into `capstone.scaffold_commits`, and flip mode to `solo`.

After that: any new or changed file since `start_commit` is your work. Commit under your git identity (`progress.json.git_identity`). Say "capstone done" when you're ready for the gate. Until then, I answer concept questions, point at docs, run your tests on request, and critique code by line reference — but I don't write code.
