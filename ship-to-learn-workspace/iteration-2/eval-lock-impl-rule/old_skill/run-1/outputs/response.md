Pause — that change isn't allowed for this TODO, and the test now passes for the wrong reason.

**Why this is locked.** `p2-t3` is a `user_writes: test` TODO. I authored `internal/shortener/validate.go` as the working impl; your job is to write the test in `internal/shortener/validate_test.go` that exercises it. Per the lock-impl rule, the impl regions I scaffolded for test-authoring TODOs are LLM-authored and off-limits — if I let you "fix" the impl to make a stub test pass, the practice loop collapses into autocomplete and the gate's verifier becomes meaningless.

**What your diff actually did.** The original `Validate`:

```
if !strings.HasPrefix(url, "http://") && !strings.HasPrefix(url, "https://") {
    return fmt.Errorf("invalid scheme: %s", url)
}
return nil
```

…rejects any URL whose scheme isn't `http`/`https`. Your replacement (`return nil`) accepts everything — including `ftp://...`, `javascript:...`, the empty string. The test "passes" only because there is now nothing to validate. That's a regression, not a fix.

**Read the situation honestly — two paths:**

1. **Most likely: your test stub was asserting the wrong thing.** The stub I left was a placeholder (`t.Fatal("write this test — plan.md TODO p2-t3")`). Your job is to *replace* that stub with real test cases against the existing impl — e.g. table-driven cases covering: valid `http://...` returns nil, valid `https://...` returns nil, `ftp://...` returns an error mentioning the scheme, empty string returns an error. The behaviors-to-cover list in `plan.md` Task 2.4 is your spec. Bridge from a stack you know: this is the same pattern as Jest's `describe.each` or pytest's `@pytest.mark.parametrize` — one test function, a slice of `{input, wantErr}` cases, loop with `t.Run`. See `testing.T.Run` subtests: https://pkg.go.dev/testing#T.Run.

2. **You genuinely think the impl is wrong.** Then say so explicitly and we'll discuss the spec — cite where you think `Validate` deviates from what `plan.md` says it should do. If you're right, I'll use the give-up escape on `p2-t3`, which produces two commits (`fix(phase 2, p2-t3): impl` then `solve(phase 2, p2-t3): test`), marks it `gave_up: true`, and excludes it from review. That's the only legitimate path to a different impl on a `user_writes: test` TODO. But before we go there: my read is the impl is correct — scheme allow-listing is the documented Go idiom for this kind of validator — so I'd want a concrete spec gap from you, not just "my test failed."

**What to do right now.**

```
git restore internal/shortener/validate.go
```

That puts the impl back. Then open `internal/shortener/validate_test.go`, delete the `t.Fatal` stub, and write a real `TestValidate` covering the behaviors in `plan.md` Task 2.4. When you're stuck on Go test syntax (one line — e.g. how to declare a table struct), ask. I'll explain. I won't write the test for you.

If you want to argue path (2), tell me which behavior you think the impl gets wrong and I'll engage on the spec.
