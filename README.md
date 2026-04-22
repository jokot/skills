# jokot skills

Agent Skills for experienced developers. Works with Claude Code, Cursor, Windsurf, Cline, Gemini CLI, and any agent that supports the [Agent Skills](https://github.com/vercel-labs/skills) standard.

## Skills

| Name | One-liner |
|---|---|
| [`ship-to-learn`](#ship-to-learn) | Ship a real project in a new stack as a learning exercise, with the LLM as coach instead of autocomplete. |

## Install

```bash
# all skills, project-level (recommended)
npx skills@latest add jokot/skills

# single skill, project-level
npx skills@latest add jokot/skills --skill ship-to-learn

# user-level (global)
npx skills@latest add jokot/skills -g
```

Project install drops content into `.agents/skills/` and symlinks into each agent's dir (`.claude/skills/`, `.cursor/skills/`, etc.). Gitignore these if you are not the author — see [`.gitignore`](./.gitignore).

---

# ship-to-learn

## What it is

A skill that turns your real project into a learning exercise. The LLM scaffolds skeletons and writes failing tests. You write the production logic against those tests. When stuck, the LLM coaches — explaining concepts, bridging from stacks you already know, citing current docs — but does not write the solution.

You end up with a working project you shipped yourself, a git history showing the learning, and a review flagging idiom deviations.

## Who it's for

Experienced developers learning a new language or framework. You ship production code in at least one stack and want to **retain** a second stack, not rent it from autocomplete.

Not for: first-time programmers, one-off scripts, refactors, people who want finished code fast, people who already know the target stack.

## Why not just use Cursor / a tutorial / ChatGPT

| Alternative | Why it fails this user |
|---|---|
| Cursor / Copilot autocomplete | You copy, you do not learn. Zero retention. |
| YouTube tutorial | Toy project, not yours. Stale versions. |
| Docs alone | No scaffolding, no coaching loop, no review. |
| "ChatGPT teach me Rust" | No real codebase, no phase gates, no idiom review. |

This skill keeps your idea, enforces your fingers on the keyboard, cites current docs via context7, and reviews against stack-specific idioms at the end.

## Use

```
/ship-to-learn
```

Or plain English:

- "learn Go by building a URL shortener"
- "I want to practice Rust on a real project"
- "teach me Elixir by shipping something small"

The skill detects intent and starts intake.

## Prerequisites

The skill halts with an install command if any hard prereq is missing.

| Requirement | Install |
|---|---|
| `git` | your package manager |
| `obra/superpowers` skills | `npx skills@latest add obra/superpowers` |
| `upstash/context7` skills | `npx skills@latest add upstash/context7` |
| `context7` MCP server | see next section |
| target-stack toolchain | detected after intake (e.g. `go`, `cargo`, `python3`) |

## context7 MCP setup (project-scope)

| Agent | Command |
|---|---|
| Claude Code | `claude mcp add -s project -t http context7 https://mcp.context7.com/mcp` |
| Cursor | Settings → MCP, or `.cursor/mcp.json`: `{"mcpServers":{"context7":{"url":"https://mcp.context7.com/mcp"}}}` |
| Windsurf / Gemini CLI / other | see agent's MCP docs; endpoint `https://mcp.context7.com/mcp` |

## How it works

On invocation the skill reads `.learn/progress.json` — if it exists, it resumes. Otherwise it starts intake: known stacks, target stack, idea (or use `superpowers:brainstorming` to find one), time budget, scope red flags.

A git worktree is created via `superpowers:using-git-worktrees`. The spec is produced by `superpowers:brainstorming` and saved (locked) to `.learn/spec.md` with a "bridges" table mapping target-stack concepts to stacks you already know. The plan is produced by `superpowers:writing-plans` and saved (locked) to `.learn/plan.md` in TDD-style bite-sized steps. Phase count = `ceil(time_budget / 90min)`; more than 10 phases triggers a push-back.

Each phase runs `build` → `coach` → gate. In `build`, the LLM scaffolds files, writes failing tests, and leaves `[PRACTICE]` steps in `plan.md` for you. In `coach`, you fill those steps; the LLM answers questions with concepts, bridges, and doc links — never code. The phase gate runs the test runner, confirms all TODOs done, asks one difficulty question, and advances. If two phases overrun the 90-minute target by >150%, remaining phases are re-planned smaller.

The final phase is the **capstone** in `solo` mode. "Capstone" = the proof-of-learning phase: one real feature in the project, you write 100% of impl + tests, the LLM writes only a feature spec and one failing test stub. Capstone gate verifies commits on owned files are authored by your git identity. Then `review` mode runs **once, project-level**, and emits `.learn/review.md` against a fixed rubric. There is no per-phase review — only the gate above.

## Modes

| Mode | Who writes code | Purpose |
|---|---|---|
| `build` | LLM | Scaffold phase files, emit failing tests, mark practice TODOs. |
| `coach` | user only (LLM max 1 line syntax demo) | User fills TODOs; LLM explains + cites docs + bridges. |
| `solo` | user only | Capstone. LLM refuses all code writes. |
| `review` | no new code | LLM reviews work against a 4-item rubric. |

`coach` and `solo` enforce a pre-response self-check — if a response would contain more than one line of code, it is rewritten as a hint + doc link. Code-pastes are critiqued by line reference, never rewritten.

## Mode switching

Modes switch in two ways. You never name a mode directly — there is no "switch to build" command. Instead the skill flips modes either automatically (based on its own state) or in response to natural-language signals from you. `coach` and `solo` refuse code writes by design; holding that line is the point of the skill.

**Automatic (skill decides):**

| From | To | When |
|---|---|---|
| intake | `build` | spec + plan written, prereqs green |
| `build` | `coach` | phase scaffolding done, TODOs emitted |
| `coach` | `build` (next phase) | phase gate passes AND next phase is not capstone |
| `coach` | (capstone prompt) | phase gate passes AND next phase is capstone — skill asks "ready for capstone?" |
| (capstone prompt) | `solo` | you say yes |
| `solo` | `review` | capstone gate passes |
| `review` | DONE | review.md written |

**You trigger (skill still runs guards):**

| You say (paraphrase) | Triggers |
|---|---|
| "done phase N" / "next phase" | Phase gate → advance on pass |
| "capstone done" | Capstone gate → auto-emit review.md → DONE (one step, not two) |
| "give up on p1-t2" | One-shot: LLM writes that TODO, marks `gave_up`, returns to coach |
| "give up on capstone" | Mark capstone not completed solo, still run review |
| "pause" / "stop" / "exit skill" | Save state; resume later via `/ship-to-learn` |

Any "just write it" / "switch to build" request in coach or solo is refused. If you genuinely want autocomplete, exit the skill explicitly first.

## Phase gate and review rubric

Phase advances only when: all phase TODOs `status: done`, test runner green, and user logs difficulty feedback (`too_easy`/`right`/`too_hard`). Failure → stay in coach, no fix from LLM.

Review rubric (4 items, every deviation cited with `file:line` + context7 source URL):

1. Idiom match — current-version stack idioms.
2. Error handling — stack-convention compliance (`%w` in Go, `Result<T, E>` in Rust, etc.).
3. Perf foot-guns — allocations in hot paths, N+1 queries, blocking in async.
4. Style — formatter-clean, naming, package organization.

Review ends with one line: the next stack concept worth tackling.

## TODOs and labels

Every practice TODO has three labels:

- `label_diff`: `easy` | `medium` | `hard`
- `label_kind`: `bridge` (analog to your known stack) | `idiom` (target-specific convention) | `logic` (pure problem)
- `user_writes`: `impl` | `test` | `both`

Default per non-capstone phase: **2 easy + 2 medium + 1 hard**.

`user_writes` progression across phases (ramps you into writing tests too):

| Phase | Mix |
|---|---|
| 1 | 5 × impl |
| 2 | 3 × impl + 2 × test |
| 3 | 2 × impl + 2 × test + 1 × both |
| 4+ | 1 × impl + 2 × test + 2 × both |
| Capstone | 100% both |

Overrideable in intake (e.g. "I already know testing, skip test-writing"). Per-TODO escape hatch: say "give up on p1-t2" — skill writes that one solution, marks `gave_up: true`, `practice: false`. Does not count toward completion.

## State directory

```
.learn/
  spec.md              # locked after phase 0
  plan.md              # locked after phase 0
  progress.json        # live ground truth (mode, phase, TODOs, events)
  notes/
    phase-<N>-<slug>.md  # append-only coach notes, per phase
  review.md            # written once, at project end
```

`spec.md` and `plan.md` are read-only after creation — scope change requires a new `/ship-to-learn` session. `progress.json` is updated on every state change. `notes/` captures coaching exchanges for later re-reading. Schema details in [`ship-to-learn/SKILL.md`](./ship-to-learn/SKILL.md).

## Triggers

Only `/ship-to-learn` is a real slash command. Everything else is natural-language intent detection.

| You say (paraphrase) | What happens |
|---|---|
| "done with p1-t2" | Mark that TODO done, stay in coach. |
| "done phase N" / "next phase" | Phase gate: run tests, log feedback, advance. |
| "give up on p1-t2" | LLM writes that TODO's solution; `gave_up: true`. |
| "show progress" / "where am I" | Print phase, mode, TODO counts. No code. |
| "ready for capstone" | Transition to `solo` (with confirmation). |
| "capstone done" | Capstone gate + review. |
| "change the scope" | Refused. Start a new session. |
| "just write it for me" | Refused in coach/solo. Explained. |
| "pause" | Save state. Resume later via `/ship-to-learn`. |

## Example

A 5-year Node/TS dev wants to learn Go by shipping a URL shortener in ~6h. Intake captures stack and budget. Spec at `.learn/spec.md` locks scope and records a Node→Go bridges table (`express.Router` → `http.ServeMux`, `async/await` → goroutines, `try/catch` → explicit `err`, Prisma → `database/sql`). Plan at `.learn/plan.md` splits into 4 phases plus capstone. In phase 1, the LLM scaffolds `shortener.go` + `shortener_test.go` + `go.mod`; tests fail, mapped to `p1-t1`…`p1-t5`. In coach, the user writes `encodeBase62`, gets stuck on `string` vs `[]byte`; the skill explains via `strings.Builder` and the Node `Buffer` bridge, no code. Phase gate passes. Phases 2–4 cover HTTP, SQLite, tests + graceful shutdown, with the user gradually writing tests too. Capstone: user adds `GET /stats/:code` hit counter entirely solo — tests + impl. Review flags three deviations (error wrapping, context propagation, table-driven test style) with `file:line` and context7 citations; closes with "next concept: goroutine leaks."

## Anti-patterns the skill refuses

- Writing code in coach or solo because the user pushed hard.
- Changing scope mid-project.
- Skipping context7 citation because "I already know this".
- Pre-filling practice TODOs "just to get started".
- Running 3+ hours on a single phase — skill re-plans.
- Using `superpowers:executing-plans` or `subagent-driven-development` on practice steps.

## Troubleshooting

| Symptom | Fix |
|---|---|
| "Prereq X missing, halting" | Run the printed install command exactly. |
| Skill writes code in coach mode | Skill bug. Kill the session and report. |
| `context7` tools not callable | Re-run the MCP install for your agent. Restart the agent. |
| `progress.json` out of sync with `plan.md` | Delete `progress.json` and re-invoke `/ship-to-learn`. Spec and plan survive. |
| Capstone review flags "non-user commits" | Check `git config user.email` in the worktree matches the identity recorded in `.learn/progress.json`. |

---

## Contributing

Each skill is a directory with a `SKILL.md`:

```
<skill-name>/
  SKILL.md
```

Frontmatter requires `name` and `description` — the description is the trigger signal, so include phrases agents should detect. See [`ship-to-learn/SKILL.md`](./ship-to-learn/SKILL.md) for a full example.

## Credits

- [`obra/superpowers`](https://github.com/obra/superpowers) — brainstorming, writing-plans, using-git-worktrees. MIT.
- [`upstash/context7`](https://github.com/upstash/context7) — current-version docs. MIT.
- [`vercel-labs/skills`](https://github.com/vercel-labs/skills) — the Agent Skills installer. MIT.

## License

MIT.
