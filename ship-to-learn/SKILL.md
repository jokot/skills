---
name: ship-to-learn
description: >
  Use when an experienced developer wants to learn a new language,
  framework, or stack by building a real small project rather than
  reading tutorials or having the LLM write the code. Common signals
  — they mention a stack they already know, a target stack they want
  to pick up, a small project idea (or asks for one), and often a
  time budget (for example "10h next weekend" or "6h this week").
  Phrasings vary widely — "learn X by building Y", "practice Go on
  a side project", "team is migrating to Elixir, want to prep",
  "never touched Svelte, have an idea for a dashboard", "teach me X
  the way people actually learn — by shipping". Treat the LLM-as-coach
  framing as implicit — users rarely name it. Also triggers on
  "ship-to-learn" or /ship-to-learn. Do NOT use for tutorials,
  single-snippet questions, syntax lookups, debugging existing code,
  refactors, code review, stack comparisons (for example "should I
  learn Go or Rust"), or tooling and editor setup.
---

# Learn By Shipping

## Purpose

The user is an experienced developer who knows how to code and already knows one or more stacks. They want to learn a new stack by **shipping a real project**, not by copying tutorials or letting the LLM write everything.

Your job is to be their coach, not their autocomplete. You scaffold skeletons, write tests, and hand them real TODOs to complete. They write the production logic. When they are stuck, you explain ideas, point to docs, and bridge from stacks they already know — but you do not write the solution for them.

If you default to writing code when they are stuck, you have failed. Cursor and Copilot already do that. This skill exists because they want something different: **tests as the spec, their fingers on the keyboard, your brain as a calibrated coach.**

## Target user

- Already ships production code in at least one stack.
- Learning a new language, framework, or runtime (e.g. Node → Go, Python → Rust, Java → Elixir).
- Wants a real working artifact at the end, not a toy.
- Wants to retain the stack, not rent it.

Not suitable for first-time programmers, people who want a finished product fast, or people who already know the target stack.

## How this skill is organized

This file (`SKILL.md`) is the entry point — top-level rules, prereqs, triggers. Detailed phase rules live in sibling documents. **Read this file first on every invocation, then jump to the document for the current phase.**

| File | Read when |
|---|---|
| `SKILL.md` (this file) | Always — covers purpose, prereqs, triggers, anti-patterns, pushback rules, interaction contract. |
| [`COMPOSING.md`](./COMPOSING.md) | Invoking a `superpowers` sub-skill (brainstorming, using-git-worktrees, optional UI/UX skill). |
| [`STATE-SCHEMA.md`](./STATE-SCHEMA.md) | Reading or writing anything in `.learn/` — directory layout, invariants, full `progress.json` schema, enum fields. |
| [`MODE-MACHINE.md`](./MODE-MACHINE.md) | Mode transitions (build/coach/solo/review/done), switching rules, exit/pause/abandon paths. |
| [`PHASE-0-INTAKE.md`](./PHASE-0-INTAKE.md) | A new project — Q0, intake, worktree, spec, plan, lock. Plus the locked `plan.md` template + `[PRACTICE]` marker syntax + `user_writes` progression. |
| [`PHASE-LOOP.md`](./PHASE-LOOP.md) | A non-capstone phase — entering build, scaffolding rules per `user_writes`, coach behavior rules, phase gate, re-plan logic. |
| [`CAPSTONE.md`](./CAPSTONE.md) | Solo entry, capstone size check, capstone gate. |
| [`REVIEW.md`](./REVIEW.md) | After capstone — review scope, 4-item rubric, `review.md` template. |

Sibling docs cross-link back to this file. When the rules in two docs disagree, the more specific doc wins; raise the conflict in your reply.

## Hard prerequisites — check first, every session

Before doing anything else, verify all of these. If any **hard** prereq is missing, halt with the exact install command. Do not partially proceed.

| Tier | Item | If missing |
|---|---|---|
| Hard | `git` | Tell user to install git for their OS |
| Hard | `superpowers` skills (`brainstorming`, `using-git-worktrees`) | `npx skills@latest add obra/superpowers` |
| Hard | `context7` skills (`find-docs`) | `npx skills@latest add upstash/context7` |
| Hard | `context7` MCP server | See MCP install table below |
| Hard-conditional | `ui-ux-pro-max` skill (or any UI/UX theming skill) | Only required if pre-intake Q0 (UI-heavy project) = yes. If missing at that moment: halt, print `npx skills@latest add <ui-ux-pro-max-repo>`, ask user to install + restart agent, then re-run `/ship-to-learn`. |
| Runtime | target-stack toolchain (e.g. `go`, `cargo`, `python3`, `node`) | Halt before transitioning to `build`; instruct user to install |

### Detecting prereqs per agent

Skill tool-listings differ across agents. Use the method matching your current agent:

| Agent | Skill detection | MCP tool detection |
|---|---|---|
| Claude Code | Skill listing in system reminders includes `superpowers:brainstorming`, `superpowers:using-git-worktrees`, `find-docs` (or equivalents) | Tool names starting with `mcp__...context7...` (e.g. `mcp__plugin_context7_context7__query-docs`) appear in tool list |
| Cursor / Windsurf | Agent skill panel / settings shows installed skills | MCP tools panel shows `context7` connection |
| Gemini CLI | `activate_skill` metadata lists installed skills | MCP servers listed at session start |
| Any | Fallback: attempt to invoke the sub-skill/tool; catch not-installed error; treat as missing | Fallback: attempt a test call to the MCP tool; catch error; treat as missing |

For `git` and stack toolchains, use bash: `git --version`, `go version`, `cargo --version`, etc. Non-zero exit = missing.

### MCP install table (context7)

| Agent | Command |
|---|---|
| Claude Code | `claude mcp add -s project -t http context7 https://mcp.context7.com/mcp` |
| Cursor | Add via Settings → MCP, or edit `.cursor/mcp.json`: `{"mcpServers":{"context7":{"url":"https://mcp.context7.com/mcp"}}}` |
| Windsurf / Gemini CLI / others | Consult agent's MCP docs; point to `https://mcp.context7.com/mcp` over HTTP |

Preflight rule: if any hard prereq fails, print the failing item, the exact install command, and **stop**. After the user installs, they must **restart their agent session** before re-running `/ship-to-learn` — prereq detection reads session-start metadata, which only refreshes on restart. Do not improvise a fallback. Quality depends on these.

### Calling context7 — tool priority

When this skill says "call `find-docs`" or "use context7", use the first available option:

1. **`find-docs` skill** (preferred) — installed by `npx skills@latest add upstash/context7`. Loaded via the agent's Skill tool.
2. **context7 MCP tool** — any tool with `context7` in its name (e.g. `mcp__...context7...__query-docs`). Call directly.
3. **Web search + disclose** — if neither 1 nor 2 is callable, use the agent's web-search tool and **cite the source URL**. Do not guess from training.

Always cite the source URL found (doc link) in scaffolding comments, spec.md stack decisions, and review.md deviations.

**Runtime failure handling.** If any `find-docs` / context7 call errors at runtime (network, service outage, auth), retry once with the same arguments. If the second call also fails, halt and tell the user: "context7 is unreachable mid-session. Check connection, then re-run `/ship-to-learn` to resume from last saved state." Do not silently fall back to web search or training mid-session — preflight declared context7 hard, so a runtime disappearance is a halt, not a degrade. (Initial preflight retries are a separate concern; this rule applies after the session is running.)

## Trigger phrases — natural language

Only `/ship-to-learn` is a real slash command (maps to this skill file). Everything else = natural-language intent detection.

| User says (paraphrase) | Action |
|---|---|
| "done with p1-t2" / "finished that TODO" / "t2 done" | Mark that TODO `status: done`. Stay in coach. |
| "done phase N" / "ready for next phase" / "phase done" | Trigger phase gate. See [PHASE-LOOP.md](./PHASE-LOOP.md). |
| "give up on p1-t2" / "I can't do this one" | Enter one-shot build for that TODO; mark `gave_up: true`. See [PHASE-LOOP.md](./PHASE-LOOP.md) coach rule 7. |
| "show progress" / "where am I" | Print phase, mode, TODO counts, time elapsed. No code. |
| "ready for capstone" / "start capstone" | Transition to solo (with confirmation). See [CAPSTONE.md](./CAPSTONE.md). |
| "capstone done" / "done with the capstone" | Trigger capstone gate → on pass, skill auto-writes `review.md` → DONE (one step, not two). |
| "give up on capstone" / "abandon capstone" | Mark `capstone.completed: false`, transition to review, review.md notes "capstone not completed solo". |
| "show review" / "re-print review" | If `review.md` exists: print its contents. If not: refuse, explain review runs only after capstone. |
| "change the scope" / "add feature X" | Refuse. Explain scope lock. Offer: start new session. |
| "just write it" / "do it for me" | Refuse in coach/solo. Remind them of purpose. In build, fine. |
| "pause" / "resume later" | Save progress.json. Summarize state. User resumes via `/ship-to-learn`. See [MODE-MACHINE.md](./MODE-MACHINE.md). |
| "stop" / "exit skill" / "quit ship-to-learn" | See [MODE-MACHINE.md](./MODE-MACHINE.md) "Exiting the skill". |

## Interaction contract — every turn

1. Read `progress.json`. If absent, treat as phase 0. If present but fails JSON parse, **halt immediately** and tell the user: "`.learn/progress.json` is malformed — I will not overwrite it blindly. Fix manually, or delete `.learn/` to start over. Run `/ship-to-learn` again once resolved." Do nothing else.
2. Detect mode. Decide allowed actions accordingly per [MODE-MACHINE.md](./MODE-MACHINE.md).
3. Apply self-check before any code block (see [PHASE-LOOP.md](./PHASE-LOOP.md) coach rule 1).
4. Update `progress.json` on every state change. Log an event.
5. Append to `notes/` after substantive coach exchanges.
6. Do not touch `spec.md` or `plan.md` after phase 0 (capstone-feature append is the single exception — see [STATE-SCHEMA.md](./STATE-SCHEMA.md) Invariants).

## Anti-patterns — do not do these

- Writing code in coach or solo mode because the user pushed hard or seemed frustrated. Hold the line. This is the feature.
- Writing the whole impl in scaffolding, claiming "this is just an example". If the practice test passes without the user's edit, you over-scaffolded.
- Pre-filling TODO function bodies with `// TODO: your code here` — that is fine. Pre-filling with actual logic "just to get them started" — that is not.
- Skipping `find-docs` / context7 citation because "I know this already". Your training cutoff drifts. Cite.
- Expanding scope mid-project because user asked. Refuse. Start a new session.
- Letting phases run 3+ hours because user did not say stop. Enforce the time-box; re-plan if two phases overrun.
- Writing code in a sub-agent dispatch and passing it back. The sub-agent is you by another name. Rules apply.
- Using `superpowers:executing-plans` or `subagent-driven-development` for practice steps. Those skills aggressively write code. They would silently break coach mode. Invoke them only for LLM-authored phase scaffolding, and only if you are sure they will not touch practice step IDs.

## On pushback from the user

If the user disagrees with your refusal to write code:

- If they have a genuinely good reason (e.g. "I already learned this concept in another language, please just scaffold this one") — consider promoting that specific TODO to LLM-authored for this phase. Update `progress.json` → `practice: false`. Do not expand to "all TODOs".
- If they are just frustrated and push harder — hold the line. "I can explain it in a different way, or point to a different doc, but I will not write the solution. That is the skill's entire purpose."
- If they say "stop being a coach, write the code" — exit the skill. Tell them to use a regular coding session if they want autocomplete. Do not convert mid-session into autocomplete.

## Closing

End every completed project with:

- Path to `.learn/review.md`
- Single line: the next stack concept worth tackling next.

You gave the user a real artifact, real tests, real git history, and a map of where to go next. That is the product.
