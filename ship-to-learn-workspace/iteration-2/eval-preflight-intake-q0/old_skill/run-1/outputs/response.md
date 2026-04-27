Welcome to `/ship-to-learn` — you'll learn Rust by shipping a real small CLI tool, with me as your coach (not your autocomplete). Tests as the spec, your fingers on the keyboard.

## Preflight check

- `git` — installed
- `superpowers` skills (`brainstorming`, `using-git-worktrees`) — available
- `context7` skill (`find-docs`) — available
- `context7` MCP server — reachable

All hard prereqs green. (Rust toolchain `cargo` will be verified later, just before phase 1 build — installing it now is fine but not required yet.)

No existing `.learn/` detected — starting a fresh intake.

## Q0 — UI project check (asked first, on its own)

Before anything else: **is this a UI/frontend project?** (web app, dashboard, design system, or anything with a significant visual layer.)

A CLI tool is almost certainly **no** — terminal output, argument parsing, no visual layer beyond text. But I want you to confirm explicitly, because if it *is* visual (e.g. a TUI with `ratatui`, rich rendered tables, an interactive dashboard) we need a UI/UX theming skill installed before we proceed.

**Please answer y / n.**

- **n** (most likely for a plain CLI) → I skip the UI/UX skill check and we move on to the rest of the intake (known stacks, target Rust version, project idea, time budget, scope check) one question at a time.
- **y** (only if you want a TUI / rich rendered visual layer) → I'll verify a UI/UX theming skill (e.g. `ui-ux-pro-max`) is installed; if it isn't, I'll halt with the install command and you'll need to install + restart the agent + re-run `/ship-to-learn`.

Once you answer, I'll ask the next intake question.
