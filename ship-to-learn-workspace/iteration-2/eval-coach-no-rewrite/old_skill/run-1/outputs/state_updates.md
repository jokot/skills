# State updates — eval-coach-no-rewrite, old_skill, turn 1

Simulated mutations the pre-split `ship-to-learn` SKILL.md would apply during/after this turn. No real files were written — this is the eval fixture's record of what the skill *would* do.

## `progress.json` mutations

```diff
 {
   "phase": 2,
   "mode": "coach",
-  "turns_since_last_progress": 0,
+  "turns_since_last_progress": 0,        // unchanged — substantive coaching exchange, not idle silence
   "phases": [
     {
       "n": 2,
       "status": "in_progress",
       "todos": [
         {
           "id": "p2-t1",
           "user_writes": "impl",
-          "status": "in_progress"
+          "status": "in_progress"        // unchanged — user did not say "done with p2-t1"
         },
         ...
       ]
     }
   ],
+  "events": [
+    ...,
+    { "ts": "<ISO-8601 via `date -Iseconds`>", "kind": "coach_exchange", "phase": 2, "todo": "p2-t1", "topic": "rand+empty-input critique" }
+  ],
+  "updated_at": "<ISO-8601 via `date -Iseconds`>"
 }
```

Notes on counter logic (per pre-split SKILL.md, "On entering coach", rule 10):

- `turns_since_last_progress` would normally `+= 1` if no progress signal landed this turn.
- Per rule 10 progress signals: (a) user did not say "done"/"fixed" — fail. (b) no new commit on phase files since last check — fail (cannot verify without `git log`, treat as no-signal). (c) no test re-run this turn — fail.
- However, the user *paste* is the start of an exchange-on-current-TODO; the rule increments on **idle** turns, not engaged ones. The pre-split skill's wording is ambiguous here — a strict reading increments to 1; a lenient reading holds at 0. I went with **hold at 0** because the user is actively engaging with the same TODO and the counter exists to detect *stuck silence*, not active dialog. (This ambiguity is a candidate finding for the eval — the post-split coach skill should make this rule unambiguous.)

## `notes/phase-2-<project-slug>.md` append

Per rule 5 ("After every substantive coaching exchange, append a note"), the skill would append:

```markdown
## p2-t1 — `Generate` first pass critique

**User question:** "is this right?" — pasted Go `Generate` impl using `rand.Read` into 6-byte buffer + `base64.URLEncoding`, with empty-string short-circuit returning `("", nil)`.

**Concept(s) covered:**
- `crypto/rand` vs `math/rand` (CSPRNG vs deterministic PRNG)
- Go error-handling idiom: every `error` return is handled or `_`-discarded with intent
- `base64.URLEncoding` vs `RawURLEncoding` (padding semantics for URL-safe codes)
- Function purpose alignment: `url` parameter ignored — design choice between random+store vs hash-of-url
- Naming: avoid shadowing `net/url` package

**Coach response (5-bullet summary):**
- Pointed out `rand.Read` error is silently dropped; idiomatic Go handles or panics on CSPRNG failure.
- Flagged `url` parameter is unused — function design is incoherent until user picks "random code" vs "hash(url)".
- Critiqued empty-string branch returning nil error as un-Go-like; should error on invalid input or pick a loud contract.
- Noted 6 bytes -> 8 base64 chars; user must reconcile with phase-2 spec target length.
- Bridged to Node `crypto.randomBytes` / Python `secrets.token_urlsafe` to anchor the pattern.

**Doc URLs (would be cited via context7 in a real session):**
- https://pkg.go.dev/crypto/rand
- https://pkg.go.dev/encoding/base64
- https://go.dev/doc/effective_go#errors

**Outcome:** user pushed back to think; no code written by LLM. Awaiting next pass.
```

## What did NOT change (per coach-mode invariants)

- `spec.md` — read-only post phase 0 lock. No edit.
- `plan.md` — read-only except for the two permitted amendments. No edit.
- No commit made by LLM. No file in the repo touched.
- `phases[1].todos[0].status` (i.e. `p2-t1.status`) **stayed `in_progress`** — rule 6 says only flip to `done` on user signal "done with p2-t1" or equivalent. The user said "is this right?" — that is the opposite signal.
- `mode` stayed `coach`. No transition triggers were uttered.

## Self-check log (rule 1, "Before every response in coach mode")

- Did I emit a fenced code block I authored, longer than one line? **No.** The only fenced block is the user's own pasted code, quoted back for line-referenced critique (explicitly marked "Your code:"). Doc URLs are inline links, not code. Per the rule, quoting the user's own paste does not count as LLM-authored code.
- Did I emit a one-line syntax demo? **No.** Not needed — the question was semantic, not syntactic.
- Did I emit a "corrected version"? **No.** Three suggested decisions for the user to make, no code shown for any of them.
- Anti-pattern check: did I pre-fill any TODO function body, even partially? **No.**
