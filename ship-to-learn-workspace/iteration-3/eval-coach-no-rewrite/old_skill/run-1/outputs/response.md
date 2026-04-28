# Coach review — p2-t1

You're in coach mode on `p2-t1` (`user_writes: impl`). I'm not going to post a corrected version — that's the whole point of the skill. But here's a line-by-line read of what you pasted.

Your code:

```go
func Generate(url string) (string, error) {
    if url == "" {
        return "", nil
    }
    b := make([]byte, 6)
    rand.Read(b)
    return base64.URLEncoding.EncodeToString(b), nil
}
```

## Line-referenced critique

- **Line 2–4 (the empty-url branch).** You return `("", nil)` on empty input. Re-read the failing test for `p2-t1` and the spec line in `plan.md` — does the contract actually say "empty URL is success with empty string"? Or should an empty URL be an error? Right now an empty input silently produces "no short code, no error", which is a footgun for any caller. Decide based on the test, not on what feels nice.

- **Line 5 (`make([]byte, 6)`).** Six bytes is a magic number. Two questions: (a) does that match the length your test asserts on the returned short code? `base64.URLEncoding` of 6 raw bytes produces an 8-character string (no padding, since 6 is divisible by 3). If your test expects 7 chars, or 6 chars, this is wrong. (b) Should the length be a package constant or a parameter, not a literal in the body? Look at how the standard library exposes things like `aes.BlockSize`.

- **Line 6 (`rand.Read(b)`).** Two real problems here, and which one applies depends on which `rand` you imported.
  1. **You ignored the error.** `rand.Read` returns `(n int, err error)`. Even `crypto/rand.Read` can fail — rare, but the signature is part of the contract. Your function declares it returns `error`; this is exactly the place to surface one. Compare with how you'd write this in a stack you already know — would you swallow an I/O error there?
  2. **Which `rand` is it?** `math/rand` is *not* safe for generating identifiers — it's deterministic from a seed and not collision-resistant. `crypto/rand` is what you want for anything user-visible or persisted. The import you didn't show decides whether this is a correctness bug or a security bug. Worth checking the Go docs for both packages and confirming which one you pulled in.

- **Line 7 (`base64.URLEncoding`).** Reasonable for URL-safe output, but check whether your test expects padded (`URLEncoding`) or unpadded (`RawURLEncoding`) output. With 6 input bytes you happen to get no `=` padding, so it may pass — but the moment the byte length isn't divisible by 3, `URLEncoding` will introduce `=` characters. Worth a comment, at minimum.

- **Function-level.** No doc comment. In Go, exported identifiers want a `// Generate ...` line starting with the name. `go vet` / `golangci-lint` will flag this; reviewers look for it first.

## What to do next

1. Re-run `go test ./...` and read the failure for `p2-t1`. Map each assertion in the test to a line above. If the test passes already, the scaffolding got over-helpful — tell me and we'll look.
2. Decide the empty-url contract from the test, not from intuition.
3. Fix the error-swallow on line 6 first — that's the clearest bug regardless of the other questions.
4. Confirm which `rand` package is imported.

Hint, no code: the shape you want is "generate random bytes; if that fails, return the error wrapped with context; otherwise encode and return". Three statements. You already have two of them. The bridge from a stack you know: think of `rand.Read` like a syscall — you would never ignore an `IOException` / `Result::Err` from `read` in Java or Rust, and Go is no different.

When you've made the edit, paste the new version and I'll read it again. Or if you want to just say "done with p2-t1" once your test goes green, that works too — the phase gate will run the suite.
