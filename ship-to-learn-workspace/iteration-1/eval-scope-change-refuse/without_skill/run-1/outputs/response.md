# Response

Happy to add OAuth login to the plan — but before we tack it on, I want to flag a few things so we make the call deliberately rather than by accident.

## Quick gut check

You're mid-implementation on a focused learning project. The current scope is already a solid Go exercise:

- POST /shorten
- GET /:code redirect
- SQLite persistence
- Tests
- Graceful shutdown

That's a complete, shippable thing. OAuth is a meaningful addition — not a small one — and dropping it into an in-progress build tends to be where side projects stall out.

## What "add OAuth" actually costs

It's rarely just "add a login button." Realistically it pulls in:

1. **Provider choice & setup** — Google? GitHub? Both? Registering an OAuth app, managing client ID/secret, redirect URIs, dev vs prod configs.
2. **Flow implementation** — Authorization Code flow with PKCE, state parameter for CSRF, token exchange, handling errors.
3. **Session management** — Cookies vs JWTs, secure/httpOnly/sameSite flags, session storage (probably another SQLite table), logout, expiry.
4. **User model** — New `users` table, migrations, linking `short_urls` to a `user_id` (schema change to existing table).
5. **Middleware** — Auth middleware on POST /shorten, deciding whether GET /:code stays public (probably yes).
6. **Tests** — Mocking the OAuth provider, testing authenticated vs unauthenticated paths, session expiry edge cases.
7. **Graceful shutdown interaction** — Making sure in-flight auth callbacks don't get cut off.

For a learning project, that's roughly **doubling the scope**. It's also a very different kind of learning (protocol/security/session work) from what you're currently doing (HTTP handlers, DB, lifecycle).

## My recommendation

**Finish the current scope first, then add OAuth as a v2.** Concretely:

1. Ship v1 as planned — get POST /shorten, GET /:code, tests, and graceful shutdown all green and committed.
2. Tag it (`v0.1.0`) so you have a clean baseline.
3. Then open a new branch for OAuth. Treat it as its own mini-project with its own plan.

Reasons:

- **You'll actually finish v1.** Half-done projects teach less than finished ones.
- **OAuth deserves its own focus.** Squeezing it in mid-stream means you'll rush the security-sensitive parts (state param, PKCE, cookie flags) which is exactly where bugs hide.
- **Clean git history.** "Add OAuth" as its own set of commits on a working baseline is easier to reason about and revert if needed.
- **Better learning.** You get to feel the "before" (anyone can shorten) and the "after" (auth-gated) as distinct states, which makes the auth concepts stick.

## If you want to add it now anyway

Totally your call. If you want to push forward, I'd suggest:

- Pick **one** provider for now (GitHub is easiest for a dev tool — simpler scopes, good docs).
- Add a minimal `users` table and a `user_id` FK on `short_urls`.
- Keep GET /:code public (redirects shouldn't require login).
- Use a well-maintained library — `golang.org/x/oauth2` for the flow, and something like `gorilla/sessions` or `scs` for sessions rather than rolling your own.
- Write the auth middleware test first.

Want me to:

**(a)** Stay the course — finish v1 as planned, then plan OAuth as v2?
**(b)** Update the plan now to include OAuth, and re-sequence the remaining work?
**(c)** Park OAuth in a `TODO.md` / `FUTURE.md` so it's captured but not on the critical path?

I'd lean (a) or (c), but it's your project and your learning goals — which of those sounds right?
