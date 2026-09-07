# jokot skills

Agent Skills for experienced developers. Works with Claude Code, Cursor, Windsurf, Cline, Gemini CLI, and any agent that supports the [Agent Skills](https://github.com/vercel-labs/skills) standard.

## Skills

| Name | One-liner |
|---|---|
| [`ship-to-learn`](#ship-to-learn) | Ship a real project in a new stack as a learning exercise, with the LLM as coach instead of autocomplete. |
| [`explain-codebase`](#explain-codebase) | Walk an unfamiliar codebase BFS-style and produce a structured, file-cited markdown explainer at `.onboard/<slug>.md` — feature, file, or branch/commit-range scope. |
| [`learn-tech`](#learn-tech) | Learn a technical topic over multiple sessions in a tracked teaching workspace, with lessons calibrated to your background and, on request, a real build guided alongside them. |

## Install

```bash
# all skills, project-level (recommended)
npx skills@latest add jokot/skills

# single skill — path syntax (shorter)
npx skills@latest add jokot/skills/ship-to-learn
npx skills@latest add jokot/skills/explain-codebase
npx skills@latest add jokot/skills/learn-tech

# single skill — flag syntax (equivalent)
npx skills@latest add jokot/skills --skill ship-to-learn

# user-level (global)
npx skills@latest add jokot/skills -g
```

Project install drops content into `.agents/skills/` and symlinks into each agent's dir (`.claude/skills/`, `.cursor/skills/`, etc.). Gitignore these if you are not the author — see [`.gitignore`](./.gitignore).

After installing or updating a skill, **restart your agent session** so the skill listing refreshes.

---

# ship-to-learn

The LLM scaffolds skeletons and writes failing tests; **you** write the production logic against those tests. When stuck, the LLM coaches — concept, bridge from a stack you already know, doc link — but never the solution. You end up with a real project shipped on your fingers, real git history, and a final review against current-version stack idioms.

For experienced devs picking up a new language or framework. Not for tutorials, refactors, code review, or stack comparisons.

## Use

```
/ship-to-learn
```

Or plain English: "learn Go by building a URL shortener", "I want to practice Rust on a real project", "teach me Elixir by shipping something small".

## Trade-off

Slower than default Claude (per-phase ~90 min, project ~6 h+). Output is a working project + git history + idiom review. Worth the speed cost when you want to **retain** a stack, not rent it.

## Skill internals

Read the skill in [`ship-to-learn/SKILL.md`](./ship-to-learn/SKILL.md) — the entry point — and follow links to sibling docs covering composing with `superpowers`, the `.learn/` state schema, modes + transitions, the phase loop, capstone, and review.

---

# explain-codebase

You join an unfamiliar codebase and ask "how does the login flow work?" Default Claude greps once, summarises the first hit, calls it done — shallow, often wrong. This skill walks the call graph **breadth-first** across multiple files (depth ≤ 3, ≤ 30 files, ≤ 200 KB), grounds every claim in a `file:line` citation, and writes the answer to `.onboard/<slug>-<date>.md` using a fixed 6-section template. A running `INDEX.md` builds a knowledge base of past explanations.

For experienced devs dropped into someone else's code — new job, OSS project, client repo. Not for writing code, debugging, refactoring, or general programming questions.

## Use

```
/explain-codebase how does the login page work
/explain-codebase explain payments.go --scope file:internal/billing/payments.go
/explain-codebase what changed --scope branch:main..feature/oauth
```

Scope (`file:` / `feature:` / `branch:` / `commit:`) can be omitted — the skill infers from the question.

## Trade-off

~3× slower and ~1.7× more tokens than default Claude on the same question. Output is a structured, file-cited markdown explainer that compounds across questions. Worth it for multi-file flows; overkill for "what's a struct in Go".

## Skill internals

Read the skill in [`explain-codebase/SKILL.md`](./explain-codebase/SKILL.md) — covers BFS protocol, scope detection, output template, `.onboard/` layout, anti-patterns, and troubleshooting.

---

# learn-tech

You want to learn a technical topic properly, not get one chat answer. This skill keeps a teaching workspace in the current directory — a mission, your stated background, cited sources, a glossary, and numbered lessons — and calibrates each lesson to what you already know. Every lesson front-loads the terms it uses that you have not met yet, and skips the ones your background already covers. On request, it also guides you through building a real thing alongside the lessons: real learning stages, a locked spec, and a build-guide section per lesson that gives you a skeleton and a citation, not a finished solution.

For anyone who wants to learn a topic in depth across sessions, with or without a project attached to it. Not for a single quick explanation, a syntax lookup, or debugging existing code.

## Use

```
/learn-tech
```

Or plain English: "teach me WebAssembly", "I want to learn Kubernetes properly", "teach me Rust and help me build a CLI while I learn it".

## Trade-off

Slower than one chat answer — it writes a workspace of files before it writes a word of lesson content, and asks for your background up front. Output is a reusable teaching workspace: lessons you can reopen, a glossary in your own words, and, on request, a real thing you built alongside them. Worth it when you plan to come back to the topic; overkill for a one-off question.

## Skill internals

Read the skill in [`learn-tech/SKILL.md`](./learn-tech/SKILL.md) — the entry point — and follow links to sibling docs covering the workspace layout and file formats, the lesson format, and the build track (brainstorming, teaching spec, per-stage plans).

---

## Contributing

Each skill is a directory with a `SKILL.md`:

```
<skill-name>/
  SKILL.md
```

Frontmatter requires `name` and `description`. The description is the trigger signal — include realistic phrases agents should detect. Use a `>`-folded YAML block for long descriptions to keep source readable. When the skill grows beyond ~500 lines, split detail into sibling `.md` files referenced from `SKILL.md`'s nav table (see `ship-to-learn/` for an example).

Test artifacts (eval definitions, benchmark workspaces) live in `<skill-name>-workspace/`, not inside the skill folder — that keeps installs lean.

## Credits

- [`obra/superpowers`](https://github.com/obra/superpowers) — brainstorming, writing-plans, using-git-worktrees. MIT.
- [`upstash/context7`](https://github.com/upstash/context7) — current-version docs. MIT.
- [`vercel-labs/skills`](https://github.com/vercel-labs/skills) — the Agent Skills installer. MIT.
- [`mattpocock/skills`](https://github.com/mattpocock/skills) — repo-layout reference for split skills + path-syntax install, and the `teach` skill's workspace design (mission, lessons, glossary, learning records) that `learn-tech` builds on. MIT.
- [`ste-writing`](./ste-writing) — this repo's own ASD-STE100 skill, required by `learn-tech` for every generated doc. MIT.

## License

MIT.
