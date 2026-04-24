No — I'm not going to add OAuth to the plan. Holding the line on scope lock is a deliberate feature of this skill, not me being difficult.

Here's the state:

- Phase 3, mode `coach`. `spec.md` and `plan.md` were locked the moment `progress.json.phase` went to 1.
- `spec.md` is **fully read-only** after creation — no amendments ever.
- `plan.md` is **append-only** with exactly two allowed amendments: `## Re-plan at phase N` (pacing trigger) and `## Capstone` (at solo entry). "Add OAuth" is neither.
- The original spec explicitly has no auth. Adding OAuth would: (a) change the scope contract you approved at phase 0, (b) blow the time budget (OAuth flows, session/token storage, middleware, and tests are realistically a multi-hour addition on top of a URL shortener), and (c) pull focus away from the stack concepts the phases were sized around.

Scope drift is the single biggest threat to this skill's value — the reason you get real retention out of ship-to-learn is that the spec you approved up front is the spec you ship against. If I let it flex here, the next phase's TODOs stop matching the plan's idiom progression, the capstone reservation becomes stale, and review has nothing stable to grade you against.

Your options:

1. **Ship this project as specified.** Finish phase 3, phase 4 (if any remain), and the capstone. You'll have a real, tested URL shortener in the new stack with a clean git history. Then:
2. **Start a new `/ship-to-learn` session** for an auth-focused project — e.g. "URL shortener with OAuth + per-user short URLs" as a fresh spec, sized for learning your target stack's auth idioms (middleware, session storage, CSRF, token validation). That's a meaty learning project on its own and deserves its own intake, its own phase plan, and its own capstone reservation. Don't bolt it onto this one.
3. **If you just want OAuth in this codebase for real-world reasons (not learning),** exit the skill and add it in a regular coding session. Inside ship-to-learn, the rules hold.

If you want, I can note OAuth as a "next stack concept to tackle" candidate so it shows up in your `review.md` at the end — that's allowed, and it keeps the idea in your followup queue without corrupting the current plan.

For now: back to phase 3. What TODO are you on, and what's blocking?
