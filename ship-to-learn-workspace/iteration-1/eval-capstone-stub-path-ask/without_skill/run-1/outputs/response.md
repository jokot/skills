Awesome, glad you're ready. Let's set you up for the capstone.

## The feature

Per-short-code hit counter that increments on every redirect. When someone hits `/r/{code}`, you bump a counter associated with that code, then redirect as usual.

## Before you write code, think through these concepts

I won't write the impl or tests — that's yours. But here are the conceptual hooks to chew on:

**1. Where does the count live?**
You already have storage for the code -> URL mapping. The hit count is just another piece of state tied to the same key. Options:
- Extend the existing record (add a `Hits` field to whatever struct/row represents a short link).
- Keep a separate map/table keyed by code.
The first is usually simpler; the second decouples read-heavy counter traffic from the mapping. For a learning project, pick the one that feels natural given your current storage shape.

**2. Concurrency.**
This is the interesting part. Redirects can happen in parallel — multiple goroutines serving HTTP requests can try to increment the same counter at the same time. Naive `count = count + 1` is a classic race condition. Think about:
- `sync.Mutex` around the read-modify-write.
- `sync/atomic` (e.g. `atomic.AddInt64`) if the counter is a plain integer — lock-free and fast.
- If you're using a SQL store, a single `UPDATE ... SET hits = hits + 1` is atomic at the DB level.
Pick one deliberately and be able to explain *why* it's safe.

**3. Should the increment block the redirect?**
The user doesn't care about the counter — they want the redirect. Two schools of thought:
- Increment synchronously before responding. Simple, correct, slightly slower.
- Fire-and-forget in a goroutine. Faster response, but you can lose counts on crash and you need to think about goroutine lifecycle.
Start synchronous. It's easier to test and reason about. You can always optimize later.

**4. How do you expose it?**
Probably a new endpoint like `GET /stats/{code}` returning the count (JSON or plain). Keep it separate from the redirect handler.

## Testing angles to consider

You're writing the tests, but here's what a good test suite for this feature covers:

- **Happy path**: create a short link, hit the redirect N times, assert count == N.
- **Isolation**: two different codes have independent counters.
- **Unknown code**: hitting a nonexistent code doesn't create a phantom counter (and returns 404).
- **Concurrency**: fire M concurrent redirects at the same code, assert count == M. This is the test that will catch a broken locking strategy. `sync.WaitGroup` + a bunch of goroutines is the standard pattern.
- **Stats endpoint**: returns 0 for a freshly-created code, returns the right number after hits.

The concurrency test is the one I'd most encourage you to write — it's where the learning is, and it's the test most likely to fail in an interesting way if you got the concurrency story wrong.

## Suggested order

1. Decide where hits live (concept decision, no code yet).
2. Write a failing test for the happy path.
3. Make it pass with the simplest possible implementation.
4. Write the concurrency test. Watch it fail or pass — either is informative.
5. If it failed, fix it and understand *why* the fix works.
6. Add the stats endpoint + its tests.
7. Handle the unknown-code edge case.

## Ground rules for this capstone

Per what we agreed: I'll help with concepts, tradeoffs, debugging your mental model, and explaining Go idioms. I won't write the implementation or the tests for you. If you get stuck, show me what you tried and what you're thinking — that's where I can add the most value without taking the learning away.

Go build it. Ping me when you hit your first interesting decision or your first failing test you don't understand.
