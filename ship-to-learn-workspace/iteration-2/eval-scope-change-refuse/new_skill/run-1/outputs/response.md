No — I can't add OAuth to the plan. This is the scope lock at work, and it's deliberate.

## Why I'm refusing

You're in **phase 3, coach mode**. `spec.md` and `plan.md` were locked the moment `progress.json.phase` flipped to `1`. The skill's invariants are explicit:

> Scope change requests (new feature, expanded goal) are refused in either file. Instruct the user to start a new `/ship-to-learn` session. Scope drift is the single biggest threat to this skill's value.

Mid-project scope expansion is the #1 way this skill fails. The original spec — `POST /shorten`, `GET /:code`, SQLite, tests, graceful shutdown, no auth — is what you committed to ship. Adding OAuth now means:

- A new dependency surface (OAuth provider, sessions, token storage) you haven't budgeted time for.
- Your phase plan and time-box go stale; the remaining phases were sized against the locked spec.
- "Just one more feature" is how 6h projects become 30h projects that never ship. You'd learn less, not more.

I'm holding the line here even if you push back. That's the feature, not a bug.

## What you can do instead

Three real options, in order of what I'd recommend:

1. **Finish the current project as-specced.** Ship the no-auth URL shortener, run the capstone, get `review.md`. You'll have a real working artifact and a clean git history in the new stack. That's the win.
2. **Park the OAuth idea for a follow-up `/ship-to-learn` session.** When this project is done, start a new session: "URL shortener with OAuth-gated creation." You'll already know the base stack, so the new session focuses purely on the auth layer — much higher signal per hour.
3. **If OAuth is genuinely blocking and the current spec feels wrong now**, you can abandon this session and start a fresh `/ship-to-learn` from scratch with the new spec. You'd lose current progress. I don't recommend this — finishing matters more than re-specifying.

## One narrow exception I'm *not* invoking

The pushback rule lets me promote a single TODO from `practice` to LLM-authored if you have a genuinely good reason (e.g. "I already know OAuth in another stack, just scaffold it"). That rule is about *how a TODO gets written*, not about *adding new features*. It doesn't apply here.

## What I need from you

Pick one:

- "Continue" — I stay in coach mode on the current phase 3 TODOs.
- "New session for OAuth version" — finish current project first, then `/ship-to-learn` again later.
- "Abandon and restart" — explicit confirmation; I'll walk you through tearing down `.learn/` and starting over.

Which one?
