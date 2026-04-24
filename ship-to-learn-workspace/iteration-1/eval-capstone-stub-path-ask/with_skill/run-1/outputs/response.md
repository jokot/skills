Locked in. Entering capstone setup.

Before I write the `## Capstone` section into `plan.md` and scaffold the single failing test stub, two quick things:

**1. Working tree check.** I need your tree clean so `capstone.start_commit` is a meaningful ownership boundary. Run:

```
git status --porcelain
```

If anything's modified/untracked, commit or stash it first. Reply "clean" (or paste the output) once done.

**2. Capstone scope sanity check.**

From `spec.md` → `## Reserved for capstone`:

> Hit counter per short code. Increments on each redirect.

That fits the solo budget (1 small feature, ~90 min, 2–3 behaviors). Before I lock it into `plan.md` as `## Capstone`, here's what I'll write — confirm or tweak:

- **Feature:** A per-short-code hit counter. Every successful redirect (`GET /:code` → 302) increments that code's counter by one. A new code starts at zero. Counters persist for the lifetime of the process (in-memory is fine; no DB required unless you already chose one in phase 1).
- **Behaviors to cover (your tests should hit each):**
  1. A brand-new short code has a hit count of zero before any redirects.
  2. Each successful redirect increments that code's counter by exactly one.
  3. Counters are independent per code (hits on `abc` do not bump `xyz`).
  4. Unknown-code requests (404) do not increment any counter.
  5. Concurrent redirects to the same code produce the correct total (thread-safe — this is the "hard" behavior; in Go, expect to reach for `sync/atomic` or `sync.Mutex`).
- **Docs I'll cite when you get stuck:** `sync.Mutex` / `sync/atomic` (Go concurrency), `net/http` handler testing (`httptest.NewRecorder`), table-driven tests.

Want me to drop behavior 5 (concurrency) if that feels like scope creep? Or keep it — it's the single most stack-idiomatic thing to practice in Go.

**3. Where should the one failing test stub live?**

This is the only file I'll create for the capstone. Just a package declaration and one test function whose body is `t.Fatal("write this test — capstone feature in plan.md ## Capstone")`. No impl file, no signatures, no hints — you design the shape.

Give me a path. Examples for your layout:

- `internal/stats/stats_test.go` — if you want a new `stats` package
- `internal/shortener/stats_test.go` — if hit-counting lives alongside the shortener
- `internal/handler/redirect_stats_test.go` — if you want to test it at the HTTP handler layer

Pick one of those or name your own. I'll create exactly that file, commit it as `scaffold(capstone, task C.1): failing stub at <path>`, record the commit SHA in `progress.json.capstone.scaffold_commits`, flip to `solo` mode, and then it's all you.

Reply with: (a) "clean" or `git status` output, (b) capstone scope confirmed / tweaks, (c) the stub path.
