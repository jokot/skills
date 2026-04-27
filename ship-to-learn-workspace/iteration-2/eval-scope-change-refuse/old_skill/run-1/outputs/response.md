# Refusing scope change — OAuth login

I can't add OAuth (or any new feature) to the plan for this project. Here's why, and what to do instead.

## Why I'm refusing

Your project is in **phase 3, coach mode**. By that point, both `.learn/spec.md` and `.learn/plan.md` are **locked**:

- `spec.md` is fully read-only after creation. **No amendments ever.**
- `plan.md` is append-only after creation, with exactly two allowed amendments — a `## Re-plan at phase N` section (only on pacing overrun) and a `## Capstone` section (appended once at solo entry). Nothing else.

Scope changes — new features, expanded goals — are **refused in either file by design**. From the skill's invariants:

> Scope change requests (new feature, expanded goal) are refused in either file. Instruct the user to start a new `/ship-to-learn` session. Scope drift is the single biggest threat to this skill's value.

The current spec explicitly defines what's in scope: `POST /shorten`, `GET /:code` redirect, SQLite, tests, graceful shutdown — **no auth**. OAuth login (authentication, user accounts, session management, protected endpoints) is a substantial new dimension. Bolting it onto a locked plan would:

1. Invalidate the `[PRACTICE]` TODO progression already designed against the original scope.
2. Push the time budget past what you signed up for — auth alone is often a multi-phase project.
3. Defeat the whole point of locking the spec: the skill exists because mid-project scope drift is the #1 way these learning sessions fail.

I'm holding the line here. This isn't me being unhelpful — it's the feature.

## What you can do

You have three good options:

**(a) Finish the current project as scoped.** You're in phase 3 already. Ship the URL shortener you designed, run the capstone, get your `review.md`. Then start a fresh `/ship-to-learn` session with a new spec that includes OAuth from day one — that's the right place to design auth-aware scaffolding, bridges, and TODO progression around session/cookie/JWT idioms in your target stack.

**(b) Bank OAuth as the *next* learn-by-shipping project.** Auth is meaty enough to stand on its own — e.g. "add OAuth + user-owned short URLs to the shortener I just built" as a follow-up project that reuses the same repo (different worktree, fresh `.learn/`). You'd already know the stack basics from this round, so the auth project gets to focus purely on the auth idioms.

**(c) If you really want OAuth *now*, abandon this project.** That means: exit this skill, `rm -rf .learn/` (or `mv .learn .learn-url-shortener-abandoned` to keep it as a record), then `/ship-to-learn` from scratch with a new spec describing "URL shortener with OAuth-gated POST /shorten." You'd lose your phase 1–2 progress, though, so I'd only recommend this if the original scope no longer matches what you actually want to learn.

I'd suggest (a) or (b). You're three phases in; finishing is cheap and the artifact is real.

## Want to keep going?

Say the word and we'll continue phase 3 coaching on the existing scope. If you want to talk through what the OAuth follow-up project might look like *after* this one ships, I can do that conceptually without touching `spec.md` or `plan.md` — it just won't get added to the current plan.
