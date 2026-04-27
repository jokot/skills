---
name: explain-codebase
description: Explain how a part of a codebase works by reading files in a disciplined breadth-first walk and producing a single structured markdown explainer at .onboard/<slug>.md. Use when the user asks "how does X work", "walk me through Y", "explain the auth flow", "what changed between branches A and B", "trace the login request", "give me a tour of <feature>", or any focused codebase question they want a written, file-cited answer to. Scope can be a feature, file, package, or branch/commit range. The skill explores the call graph (BFS up to depth 3, ~30 files, 200KB) instead of stopping at the first grep hit, locks the output to a fixed section template, and persists explanations under .onboard/ with a running INDEX.md so a team can build a knowledge base over time. Do NOT use for writing new code, debugging existing failures, refactoring, code review, or stack-level "should I learn Go or Rust" questions.
---

# Explain Codebase

## Purpose

The user has joined a codebase they did not write — a new job, an OSS project, a client repo — and they need an honest, file-cited written answer to a focused question. They do not want a chatty live conversation; they want a markdown file they can scroll through later, share with a teammate, or grep through next month. Default agent behavior is to grep once, summarize the first hit, and call it done. That produces shallow, often-wrong answers because real code lives across multiple files connected by imports, calls, and config.

This skill exists to do the disciplined thing: walk the call graph breadth-first, read each file actually involved, ground every claim in a `file:line` reference, and write the output to disk in a fixed structure so the user (and their team) can build a searchable knowledge base of explanations over time.

## Target user

- Experienced developer dropped into an unfamiliar codebase.
- Wants a written artifact, not a chat, for a specific question.
- Will keep `.onboard/` in version control or share it with teammates.

Not for: scaffolding new code, debugging a failure, refactor planning, code review, or general programming questions where no specific codebase is involved.

## When to trigger

Trigger phrases (treat liberally — undertriggering is the bigger risk):

- "how does X work"
- "walk me through Y"
- "explain the auth flow"
- "trace the login request from request to response"
- "what does `internal/handler/redirect.go` do"
- "give me a tour of the billing package"
- "what changed between `main` and `feature/oauth`"
- "explain commit `abc123`"
- "/explain-codebase ..."

Combined cue: a focused **codebase question** + an implicit or explicit **scope** (file path, package name, feature name, or branch/commit reference).

## Modes

**v1 ships only `auto` mode**: the skill reads files itself, walks the call graph, and writes the output markdown without asking the user to do anything between trigger and final output (other than scope clarification if needed).

A `manual` mode (user reads, user summarizes, skill coaches like ship-to-learn) is reserved for a future version and is not available now. If the user explicitly asks for "manual mode" or "I want to read it myself", reply that the current version is auto-only and offer to point them at relevant files instead, then run auto on their behalf.

## Inputs

A single trigger consists of:

1. **Question** — natural language. Required.
2. **Scope** (optional but useful) — one of:
   - `file:<path>` — explore from a specific file outward
   - `feature:<name>` — explore from a feature/package directory
   - `branch:<name>` or `branch:<a>..<b>` — explore the diff between branches or what's on one branch
   - `commit:<sha>` or `commit:<a>..<b>` — explore commit-range diffs
   - omitted — infer scope from the question (described below)

Examples:

- `/explain-codebase how does the login page work`
- `/explain-codebase explain the auth middleware --scope file:internal/auth/middleware.go`
- `/explain-codebase what changed --scope branch:main..feature/oauth`

## Preflight

1. Check the current directory is a directory the agent can read.
2. Check `git --version` works. If yes, branch/commit scopes are available; if no, only file/feature scope works (degrade gracefully — tell the user once, proceed with what's available).
3. Ensure `.onboard/` exists; create it if not. Do not require it to be in `.gitignore` — the user may want it tracked. Do not modify `.gitignore`.
4. If `.onboard/INDEX.md` is missing, create it with the header below.

No external tooling required (no language servers, no ctags, no MCP). The walk is grep + read + judgment.

## Scope detection (when user did not pass `--scope`)

Inspect the question. Detect:

- Filename or path patterns (e.g. `redirect.go`, `internal/auth/middleware.go`) → scope = `file:<that>`.
- Branch keyword + branch-like name (e.g. "between main and feature/x") → `branch:<a>..<b>`.
- Commit SHA pattern → `commit:<sha>`.
- Otherwise → grep the codebase for keywords from the question (auth, login, redirect, billing) and pick the **smallest containing directory** that hits multiple relevant files. Save this as `feature:<inferred>`.
- If grep finds nothing, ask the user once: "I can't infer a starting point from the question — give me a file, package, or feature name to start." Do not guess.

Record the resolved scope in the output frontmatter.

## BFS exploration protocol

The whole point of this skill is to walk **wider than the first grep hit**. Without discipline, agents grep once, read the top match, and stop — producing answers that miss the real complexity. Follow this protocol exactly:

### State

Maintain a session-state file at `.onboard/_session_state.json` while exploring. Layout:

```json
{
  "question": "<original question>",
  "scope": "<resolved scope>",
  "started_at": "<ISO>",
  "depth_cap": 3,
  "file_cap": 30,
  "byte_cap": 204800,
  "depth": 0,
  "visited": ["path/a.go", "path/b.go"],
  "frontier": ["path/c.go"],
  "bytes_read": 12345,
  "notes": [
    {"file": "path/a.go", "lines": "12-40", "fact": "..."},
    ...
  ]
}
```

Persist after every batch read. This makes the walk resumable if the agent is interrupted, and gives the final write step a structured fact log to draw from.

### Walk

1. **Seed.** From the resolved scope:
   - `file:` → seed = that file.
   - `feature:` → seed = directory's primary entry point (main.go, index.ts, lib.rs, __init__.py, etc.). If multiple, pick 1–3 obvious ones via grep for "main", "App", "init", "router", "Server".
   - `branch:` / `commit:` → seed = files in the diff (`git diff --name-only <range>`).
2. **Read seeds.** Add to `visited`. Update `bytes_read`. Extract from each read:
   - Imports / requires / `use` statements → callees (where this file goes for help).
   - Functions / classes / public symbols → potential entry points for callers.
   - Inline file-path strings (config refs, template paths) → other places to check.
3. **Caller search.** For each public symbol of interest, grep the codebase for its name. Add unique callers to `frontier`.
4. **Callee follow.** For each import that targets a file inside this repo (not an external library), add to `frontier`.
5. **Iterate.** While `len(visited) < file_cap` AND `bytes_read < byte_cap` AND `depth < depth_cap` AND `frontier not empty`:
   - Pop up to 3 highest-relevance items from frontier (relevance = grep-density of question keywords + symbol-name overlap with question).
   - Read each. Update `visited`, `notes`, `bytes_read`.
   - Repeat from step 3.
6. **Stop early if:** the question has been answered with at least 3 cited `file:line` references AND the next frontier batch contains nothing the user's question depends on (judgment call — if you cannot defend "this file matters to the question", do not read it).

### Cap behavior

If a cap (file, byte, or depth) is hit before the question is answered:

- Do not silently summarize. Write the explainer with what was visited and **mark the Open Questions section explicitly** with the unread paths and why they probably matter ("this is called from `X.go:42` but I did not read it; likely contains the actual storage logic").
- Suggest a follow-up `/explain-codebase` invocation with a narrower scope.

### Don't

- Don't grep once and stop.
- Don't read a file you cannot defend reading. Random reads waste budget.
- Don't claim a fact unless you can cite a `file:line` for it. If unsure, put it in Open Questions.
- Don't pretend depth was reached when you bailed at depth 1 because the question was easy. Be honest in the depth field.

## Output: structure, filename, index

### Filename

`.onboard/<slug>-<ISO>.md` where:

- `<slug>` = lowercase-hyphenated form of the question (max 60 chars, drop stop words, keep nouns).
  Example: "how does the login page work" → `how-does-login-page-work`.
- `<ISO>` = `YYYY-MM-DD`. (Date only, not full timestamp — keeps filenames readable.)

If a file with the same name already exists from today, append `-2`, `-3`, etc.

### File template (locked)

Every output file uses **exactly** this structure. Sections may be empty (with a single line "Nothing notable.") but never omitted.

```markdown
---
question: <original user question, verbatim>
scope: <resolved scope>
generated_at: <ISO timestamp>
visited_files: <count>
depth_reached: <int>
budget_state: <"complete" | "file-cap" | "byte-cap" | "depth-cap" | "early-stop">
---

# <Title — title-cased version of the question>

## Overview

<1–3 short paragraphs answering the question at a high level. Each
non-trivial claim cites at least one `file:line`. No speculation.>

## Entry point

<The first file/function the explanation hangs on. One paragraph plus
a code excerpt of ≤10 lines from the entry point. Cite `file:line`.>

## Sequence

<Step-by-step trace of the flow. Numbered list. Each step:
"`file:line` — what happens here, in one short sentence."

If the question is non-flow (e.g. "what does X do"), replace with
"## Structure" and list the major components instead.>

## Files touched

<Bulleted list of every file in `visited`, with one-line role. Format:
- `path/to/file.ext` — role/responsibility (cited when non-obvious).>

## Gotchas

<Things a fresh reader will get wrong. Each item is a 1–3 sentence
warning with a file:line ref. Examples:
- Cache invalidation timing
- Race conditions on shared state
- Confusing naming where the function is misnamed
- Hidden config that overrides apparent default

If genuinely none, write "Nothing notable.">

## Open questions

<What was NOT answered. Reasons:
- Cap reached (which cap, what was unread)
- Code unclear and you do not want to guess
- External dependency you cannot inspect from this repo

Each as a bullet. If the answer is fully grounded, write
"Nothing material to flag.">
```

### INDEX.md (append-only)

Maintain `.onboard/INDEX.md` like this — one row per generated file, newest at bottom:

```markdown
# Codebase explanations — <repo name>

Personal/team knowledge base of past walkthroughs. Entries are
append-only; if a topic was re-explored later, both rows stay.

| Date | Scope | Question | File |
|---|---|---|---|
| 2026-04-27 | feature:auth | how does login work | how-does-login-work-2026-04-27.md |
| 2026-04-27 | branch:main..feature/oauth | what changed for oauth | what-changed-for-oauth-2026-04-27.md |
```

If repo name is not obvious from `git remote -v` or the parent directory, use the parent directory's basename. The header line is set on first creation and not modified later.

After writing the explainer, append a row to INDEX.md and stop. Do not "summarize the file again" in chat — the file is the deliverable.

## Anti-patterns the skill must avoid

- **First-grep stop.** The whole moat of this skill is breadth. If the agent reads only the file with the keyword, the user got nothing they could not have gotten faster from `grep` themselves.
- **Unread file claims.** Asserting "and then it stores in Postgres" without citing the line that does the insert. If you did not read it, put it in Open Questions.
- **Filler prose to look thorough.** Every section line should change a reader's mental model. Cut anything that does not.
- **Touching code.** This skill reads only. It does not edit, refactor, or write new code. If the user asks for a change at the end, tell them to use a different invocation; do not bolt the change on.
- **Modifying `.gitignore`.** Whether `.onboard/` is tracked is the user's call. Don't decide for them.
- **Skipping `.onboard/_session_state.json`.** The state file is the only way to recover from a mid-walk interruption and is the single source of facts the final write draws from. Keep it updated; do not write the explainer from memory of what was read.

## Examples

### Example 1 — feature scope

User: `/explain-codebase how does the login page work`

Skill action:
1. No `--scope`, infer. Grep "login", "signin", "auth". Hits cluster in `web/login/`, `internal/auth/`. Smallest containing → `feature:auth`.
2. Seed: `internal/auth/middleware.go`, `web/login/handler.tsx`. Visited 2.
3. Walk: middleware imports session store → read `internal/session/store.go`. Handler calls `/api/login` → grep route → `web/api/login.go`. Continue 2 levels deep.
4. Stop at depth 3, 18 files visited, 87KB read. Answered.
5. Write `.onboard/how-does-login-page-work-2026-04-27.md` with all 6 sections, sequence numbered 1–7 from request to session cookie. Files touched lists 18.
6. Append row to `INDEX.md`. Done.

### Example 2 — branch range

User: `/explain-codebase what's new between main and feature/oauth --scope branch:main..feature/oauth`

Skill action:
1. Run `git diff --name-only main..feature/oauth`. Get 12 files.
2. Read them in priority order (handlers/routes first, then helpers, then tests).
3. Walk callers of changed functions to spot ripple effects.
4. Write explainer with Sequence section showing the new request flow added by the branch.
5. Open Questions: any function on the branch that is called from a file we did not visit (likely depends on integration tests).
6. Append to INDEX.md with `branch:main..feature/oauth` in the Scope column.

### Example 3 — file scope, capped

User: `/explain-codebase explain payments.go --scope file:internal/billing/payments.go`

Skill action:
1. Read the file. Note 4 imports inside repo + ~30 references to its public methods elsewhere.
2. BFS the 30 references, grouped by directory. Hit file cap at 30 with 6 still on the frontier.
3. Write explainer; budget_state: "file-cap".
4. Open Questions explicitly lists the 6 unread files with one-line guesses why they matter and a recommended next invocation: "for collection logic, run `/explain-codebase how is payment collection retried --scope feature:internal/jobs/`".

## Closing

The deliverable is the markdown file at `.onboard/<slug>-<date>.md`. The user does not need a chat summary on top of it. End with:

> Wrote explainer to `.onboard/<slug>-<date>.md`. Indexed at `.onboard/INDEX.md`. <N> files visited, depth <D>, budget: <state>.

That is the whole output of the skill in chat.
