# Coach response — p2-t1 (user_writes: impl)

You're on `p2-t1` and you wrote the impl. I'll critique by line reference. I will not post a corrected version — that's the rule for this TODO.

Your code (re-quoted for line-referenced critique):

```
1: func Generate(url string) (string, error) {
2:     if url == "" {
3:         return "", nil
4:     }
5:     b := make([]byte, 6)
6:     rand.Read(b)
7:     return base64.URLEncoding.EncodeToString(b), nil
8: }
```

## Issues, by line

**Lines 2–4: empty-input handling silently returns success.**
Look at the failing test I scaffolded for `p2-t1`. It almost certainly asserts that `Generate("")` returns a non-nil error (something like `ErrEmptyURL`), not `("", nil)`. Returning `nil` for an empty input means the caller can't distinguish "no URL given" from "successfully shortened to empty string". Re-read the test name and the assertion — the spec is in the test, not in your head. Bridge from a stack you know: this is the same trap as a JS function that returns `""` on bad input instead of throwing — callers stop checking and bugs hide.

**Line 6: `rand.Read` returns `(n int, err error)`. You're discarding both.**
This is the most important issue, and it's the concept this TODO is teaching. Two separate problems hiding here:

1. **Which `rand`?** If you imported `math/rand`, this is not safe for generating identifiers that anyone could guess or collide on — `math/rand` is deterministic and seeded weakly by default. For anything resembling a token or short-link slug, you want `crypto/rand`. See the package overview: https://pkg.go.dev/crypto/rand — note the opening sentence: "Package rand implements a cryptographically secure random number generator." That phrasing is the whole reason it exists as a separate package from `math/rand`.

2. **Ignoring the error.** Even with `crypto/rand`, `Read` can fail (entropy source unavailable, etc.). The idiom in Go is to handle every `error` return at the call site, or explicitly document why you're ignoring it. The signature is documented here: https://pkg.go.dev/crypto/rand#Read — note it returns `(n int, err error)`, and the docs say "On return, n == len(b) if and only if err == nil." That's the contract you're throwing away on line 6.

Bridge from your known stack: in Node, this is the difference between `Math.random()` and `crypto.randomBytes()` — same split, same reason. You'd never use `Math.random()` for a session token; same instinct applies here.

**Line 7: `base64.URLEncoding` vs `base64.RawURLEncoding`.**
`URLEncoding` produces padded output (`=` chars). For a short-link slug that ends up in a URL path, padding is usually unwanted and sometimes invalid depending on your routing. Look at the encoding options and pick deliberately: https://pkg.go.dev/encoding/base64#pkg-variables — compare `URLEncoding` (padded) vs `RawURLEncoding` (unpadded). Which one matches the test's expected output length? Count the bytes-in vs chars-out: 6 random bytes encodes to 8 chars; with `URLEncoding` you get padding only when input length is not a multiple of 3 (6 is, so no `=` here). So in this *specific* case the output happens to coincide — but the test for `p2-t1` may use a different byte length where it won't. Pick based on the contract, not the coincidence.

**Line 5: where did `6` come from?**
The test should pin the slug length. Is `6 bytes -> 8 chars` what it expects, or 7, or 11? Don't hardcode a literal that you guessed — read the assertion and either match it exactly or, better, define a `const slugBytes = N` near the top of the file with a comment explaining the length-vs-collision tradeoff.

## What to do next

1. Open the test file for `p2-t1`. Read the assertions out loud.
2. Decide: which `rand` package, padded vs unpadded base64, what error sentinel for empty input.
3. Edit your impl. Re-run `go test ./...`.
4. When it's green, say "done with p2-t1" and I'll mark it.

If you want to push back on any of these critiques — e.g. "no, the test actually expects `URLEncoding` with padding" — paste the failing test output and we'll work from there. I will not rewrite the function for you. That's `p2-t1`'s whole point.

## Self-checks (per PHASE-LOOP.md coach rule 1)

**(a) Code self-check:** No fenced code block in this response is LLM-authored. The one block above is your pasted code, re-quoted with line numbers solely for reference critique (rule 1a allowed exception, rule 3 explicit allowance). No corrected version anywhere — rule 3 forbids it.

**(b) Citation self-check:** Three current-version Go stdlib doc URLs cited inline:
- https://pkg.go.dev/crypto/rand (CSPRNG framing)
- https://pkg.go.dev/crypto/rand#Read (Read signature + n/err contract)
- https://pkg.go.dev/encoding/base64#pkg-variables (URLEncoding vs RawURLEncoding)

All surfaced via `find-docs` / context7, not training memory. Rule 4 (citation non-optional) satisfied.
