Hold up — I need to stop you here.

The impl for `p2-t3` (`internal/shortener/validate.go`) was LLM-written and is **locked**. For this TODO, `user_writes: test` — your job is the test in `internal/shortener/validate_test.go`, not the impl.

What you just did inverts the exercise: you weakened `Validate` to `return nil` so the failing test stub stops failing. That's not the test passing on its merits — that's deleting the behavior the test was supposed to assert. The original impl rejected non-`http(s)://` schemes; your edit accepts everything, including `ftp://`, `javascript:`, the empty string, and `not a url`. A green test against that impl proves nothing.

Please revert and put your fingers back on the test file:

```
git restore internal/shortener/validate.go
```

Then open `internal/shortener/validate_test.go` and replace the `t.Fatal("write this test — plan.md TODO p2-t3")` stub with real assertions. From plan.md the behaviors to cover for `p2-t3` are roughly:

- valid `http://...` URL returns nil
- valid `https://...` URL returns nil
- non-http(s) scheme (e.g. `ftp://example.com`) returns an error
- empty string returns an error
- garbage / non-URL string returns an error

Table-driven style is idiomatic Go here — one `[]struct{ name, in string; wantErr bool }` and a `for _, tc := range cases { t.Run(tc.name, ...) }`. I'll review line-by-line once you've written it and committed.

One genuine out: if you actually believe the locked impl is wrong — not just inconvenient for your test — tell me **why** (which behavior is incorrect, and ideally a doc or spec link). If you're right, I'll use the give-up escape on `p2-t3`: I'll commit `fix(phase 2, p2-t3): impl` with a corrected `Validate`, then `solve(phase 2, p2-t3): test` covering the behaviors above, and we'll mark the TODO `gave_up: true` and move on. That path is on the record (commit messages + `gave_up` flag) — it's not a stealth rewrite.

But "I changed validate.go so my tests would pass" isn't a bug report on the impl; it's the inverted exercise. Revert, write the test, and ping me when you're stuck on an assertion.
