# Review mode

> Final project review against a fixed rubric. Runs once after capstone. See [SKILL.md](./SKILL.md) for the entry point.

## Scope — what review examines

Review covers **only user-authored regions**. Exclude LLM-authored code entirely (scaffolding, failing tests you wrote, give-up solutions).

1. **Phase-practice regions:** for every TODO in `progress.json.phases[*].todos[*]` where `practice == true` AND `gave_up == false` AND `status == done`: read the TODO's target file region (`impl_file` for `user_writes: impl|both`, `test_file` for `user_writes: test|both`). Those are user-authored.
2. **Capstone regions:** run `git log --format="%H" <capstone.start_commit>..HEAD`, filter OUT every SHA listed in `progress.json.capstone.scaffold_commits`. For the remaining user commits, compute the per-commit diff and include those hunks.
3. **Excluded explicitly:** any file region from a TODO with `gave_up == true` — that's LLM's code. Any region introduced by a SHA in `capstone.scaffold_commits` — also LLM's code. Any file region unchanged since the beginning of the project.

## Steps

1. Compute review scope per above. If the scope is empty (no user-authored regions), skip the rubric and note "No user-authored code detected" in `review.md`. This usually indicates a bug; still emit the file.
2. Read `spec.md` stack decisions + bridges for expected idioms.
3. Call `find-docs` / context7 for the specific idioms implicated by deviations — budget ≤10 calls, scoped to suspected deviations (not every concept on the page). Cite each URL.
4. Evaluate against the fixed 4-item rubric:
   1. **Idiom match** — does user code follow current-version stack idioms? Cite context7 source per deviation.
   2. **Error handling** — stack-convention compliance (e.g. `%w` wrap in Go, `Result<T, E>` in Rust, explicit exceptions in Python).
   3. **Perf foot-guns** — allocations in hot paths, N+1 queries, blocking calls in async, obvious O(n²) with large n.
   4. **Style** — formatter-clean, naming conventions, package/module organization.
5. Emit `.learn/review.md` per the template below. Every deviation needs `file:line` + context7 citation URL. One-liner at end: the single next stack concept to tackle (e.g. "next: read about goroutine leaks").
6. Transition to DONE. Log a `done` event in `progress.json` and set `mode: "done"`.

## `review.md` template

```markdown
# Review — <project-name>

**Completed:** <ISO date>
**Stack:** <language version>
**Phases done:** <N>/<total> | **Practice TODOs:** <done-solo>/<total> | **Gave up:** <Z> TODOs + capstone=<true|false completed-solo>

## Metrics

- Practice TODOs completed solo (not gave_up): `<done-solo>`/`<total>`
- Practice TODOs gave up (LLM solved one-shot): `<Z>` — **excluded from rubric below**
- Capstone completed solo: `<true | false>` — if false, note which stage they stopped at
- Non-user capstone commits flagged: `<list of file:author pairs or "none">`

## Rubric

| Dimension | Verdict | Refs |
|---|---|---|
| Idiom match | good / ok / needs work | file:line, file:line |
| Error handling | ... | ... |
| Perf foot-guns | ... | ... |
| Style | ... | ... |

## Deviations (ranked by impact)

### 1. <short title> — `<path>:<line>`

**You wrote:**
\`\`\`<lang>
<their code>
\`\`\`

**Idiomatic:**
\`\`\`<lang>
<idiomatic code>
\`\`\`

**Why:** <reason>
**Cite:** <context7 source URL>

### 2. ...

## What's solid
- <concrete item with file:line>
- ...

## Gave-up TODOs (not evaluated)
- `<todo-id>` — <one-line why user bailed, if notes captured it>
- ...

## Next stack concept to tackle
<one recommendation grounded in the gaps observed — usually pointing at the most common deviation category or the area where the user gave up most>
```
