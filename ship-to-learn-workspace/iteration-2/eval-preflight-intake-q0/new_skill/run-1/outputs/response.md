# /ship-to-learn — new project

Welcome. Before we design the project, a few quick checks.

## Preflight (hard prereqs)

| Check | Status |
|---|---|
| `git` installed | ok |
| `superpowers` skills (`brainstorming`, `using-git-worktrees`) | ok |
| `context7` skill (`find-docs`) | ok |
| `context7` MCP server | ok |
| Rust toolchain (`cargo`) | will verify before we transition to build (step 6 of intake) |

CWD: `/tmp/learn-rust-cli` — empty initialized git repo, no `.learn/` directory present. Treating as a fresh project. No `progress.json` to resume.

I noticed you mentioned "a small CLI tool" — good, that's a learning-sized idea and we'll firm it up during intake. I'll keep all your inputs short; one question at a time.

## Q0 — UI project check (asked first, separate turn)

Is this a UI/frontend project (web app, dashboard, design system, any project with a significant visual layer)? **y/n**

A Rust CLI tool sounds like **n** to me, but I won't infer — please confirm. The reason this matters: a "yes" pulls in an extra UI/UX theming sub-skill (`ui-ux-pro-max`) and an extra approval pass on a `theme.md`, and I have to verify that skill is installed before we proceed. A "no" skips all of that.

After your answer I'll ask the rest of intake one question at a time:

1. Known stacks + years of experience
2. Target stack and version (Rust — which edition / toolchain version?)
3. Idea (you have one — "small CLI tool" — we'll narrow it)
4. Time budget (hours across how many sessions)
5. Any scope red flags

Then we create a worktree, run `superpowers:brainstorming` to draft `spec.md`, enrich it with current Rust idioms via `context7`, write `plan.md` directly (not via `superpowers:writing-plans` — its no-placeholders rule conflicts with our `[PRACTICE]` markers), reserve a capstone feature, lock both files, verify `cargo --version`, and start phase 1 in `build` mode.

Waiting on your y/n for Q0.
