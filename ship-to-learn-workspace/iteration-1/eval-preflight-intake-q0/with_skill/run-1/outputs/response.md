Welcome to `/ship-to-learn`. Before we design anything, two quick things:

**Preflight check (hard prereqs):**

- `git` — present
- `superpowers` skills (`brainstorming`, `using-git-worktrees`) — present
- `context7` skill (`find-docs`) — present
- `context7` MCP server — present

All hard prerequisites green. No `.learn/` directory detected — this is a new project.

Note: I will check for the Rust toolchain (`cargo --version`) later, right before we scaffold Phase 1. If it's missing at that point I'll halt and ask you to install it. The `ui-ux-pro-max` skill is only required if you answer "yes" to Q0 below.

---

**Q0 — UI project check (asked first, on its own):**

Is this a UI/frontend project — a web app, dashboard, design system, or anything with a significant visual layer? **y / n**

(For a Rust CLI tool I'd expect "n", but I'm asking explicitly rather than assuming — your answer determines whether we pull in the UI/UX theming skill before Phase 1. Please answer "y" or "n" and I'll take it from there; the rest of the intake questions — known stacks, target Rust version, idea sizing, time budget, scope red flags — come one at a time after this.)
