---
question: how does logrus add hooks to log entries?
scope: feature:hooks
generated_at: 2026-04-27T09:09:58Z
visited_files: 5
depth_reached: 2
budget_state: early-stop
---

# How Does Logrus Add Hooks To Log Entries

## Overview

Logrus hooks are user-supplied objects that satisfy the two-method `Hook` interface (`hooks.go:8-11`) and are registered on a `Logger` instance. Each `Logger` owns a `LevelHooks` map (`map[Level][]Hook`, `hooks.go:14`) accessible via the exported `Logger.Hooks` field (`logger.go:26`). Registration goes through `Logger.AddHook` (`logger.go:376-381`), which holds the logger's mutex while delegating to `LevelHooks.Add`; that helper iterates the hook's declared `Levels()` and appends the hook to each level's bucket (`hooks.go:18-22`), so one hook can be registered against multiple severities in a single call.

Hooks are not invoked when they are added — they fire during `Entry.log` (`entry.go:279-322`), the central code path used by every level method. Before formatting and writing, `Entry.log` snapshots the hooks for the entry's level via `Logger.hooksForLevel` (`logger.go:383-396`, which copies the slice under the mutex) and then walks them in `Entry.fireHooks` (`entry.go:331-338`). Hooks receive a `*Entry` pointer and may mutate fields like `Data` — those mutations flow through to the formatter that runs immediately after.

There is also a package-level shortcut: `logrus.AddHook` (`exported.go:50-53`) just forwards to `std.AddHook`, where `std = New()` is the implicit global logger (`exported.go:9-11`).

## Entry point

`Logger.AddHook` is the public registration API. It locks the logger, then defers to the registry's `Add`, which is where the per-level fan-out actually happens.

```go
// logger.go:376-381
func (logger *Logger) AddHook(hook Hook) {
    logger.mu.Lock()
    defer logger.mu.Unlock()
    logger.Hooks.Add(hook)
}
```

The complementary firing entry point is `Entry.log` (`entry.go:279-322`), where hooks are pulled and run.

## Sequence

1. `hooks.go:8-11` — User defines a type implementing `Hook` interface: `Levels() []Level` declares which severities to fire on; `Fire(*Entry) error` runs at log time.
2. `logger.go:376-381` — User calls `logger.AddHook(myHook)` (or `logrus.AddHook(myHook)` at package level, `exported.go:50-53`). Logger mutex is acquired.
3. `hooks.go:18-22` — `LevelHooks.Add` iterates `hook.Levels()` and appends the hook to the slice for each level: `hooks[level] = append(hooks[level], hook)`. The same hook ends up in every relevant bucket.
4. Logger mutex released; the hook is now live for any future log call at those levels.
5. `entry.go:279-288` — When user calls `logger.Info("...")` (or any level method, all of which converge on `Entry.log`), the entry is duplicated and Time/Level/Message are set.
6. `entry.go:302` — `logger.hooksForLevel(level)` is called; it locks the logger, copies the `[]Hook` slice for that level, and returns the copy (`logger.go:383-396`). Snapshotting is what makes the firing loop safe against concurrent `AddHook`/`ReplaceHooks` and against re-entrant hooks that might mutate the registry.
7. `entry.go:303` — `newEntry.fireHooks(hooks)` walks the snapshotted slice.
8. `entry.go:331-338` — For each hook: `hook.Fire(entry)` is invoked. On error, `fmt.Fprintln(os.Stderr, "Failed to fire hook:", err)` is printed and `fireHooks` returns early — the remaining hooks for this event are skipped. The error is never surfaced to the caller of `Info()`/`Error()`/etc.
9. `entry.go:305-313` — Only after all hooks have run does the entry get formatted via `Formatter.Format` and written to `Logger.Out`. Mutations made by hooks to `entry.Data`, `entry.Message`, etc. are therefore visible to the formatter.

## Files touched

- `hooks.go` — Defines the `Hook` interface and the `LevelHooks` registry type, including `Add` (registration fan-out) and a `Fire` helper (note: this `LevelHooks.Fire` is unused by the main log path; firing actually goes through `Entry.fireHooks`).
- `logger.go` — Declares `Logger` struct with the exported `Hooks LevelHooks` field, initializes it in `New()`, and provides the locked `AddHook`, `hooksForLevel` snapshot helper, and `ReplaceHooks` swap operation.
- `entry.go` — Defines the `Entry` struct passed to hooks and contains `Entry.log` (the central log path that calls hooks before formatting) and `Entry.fireHooks` (the actual iteration loop, with stderr-only error handling).
- `exported.go` — Provides the package-level `logrus.AddHook` that forwards to the implicit global `std *Logger`.
- `hooks/test/test.go` — Reference `Hook` implementation that simply records every entry; demonstrates the `var _ logrus.Hook = (*Hook)(nil)` interface-conformance idiom and the `NewGlobal`/`NewLocal` install pattern.

## Gotchas

- Hook errors are silently swallowed and logged to stderr; the user's `log.Info(...)` call returns no signal that a hook failed (`entry.go:331-338`). A failing hook also short-circuits any later hooks for that same event.
- Hooks fire **before** the formatter runs (`entry.go:299-313`). They receive a `*Entry` and can mutate `Data`, `Message`, etc. — and those mutations are visible to the formatter and to the eventual write. This is a feature for enrichment hooks, but a footgun for hooks meant to be observe-only.
- The hook list is snapshotted via `hooksForLevel` (`logger.go:383-396`) before iteration. A hook that calls `logger.AddHook` during its own `Fire` will not affect the in-flight event — only future ones. Don't write a hook that depends on seeing itself fire.
- `Logger.Hooks` is an exported map field (`logger.go:26`). Direct mutation (`logger.Hooks[level] = ...`) bypasses the mutex used by `AddHook`/`hooksForLevel`/`ReplaceHooks` and will race with concurrent log calls. Prefer `AddHook`/`ReplaceHooks` for any code path that runs alongside logging.
- `LevelHooks` exposes its own `Fire(level, entry)` method (`hooks.go:26-34`) that **returns** errors. The actual log path does **not** use it — `Entry.fireHooks` (`entry.go:331-338`) re-implements iteration with stderr-only error handling. Reading `LevelHooks.Fire` and assuming that's what runs at log time is wrong.
- A hook that declares `Levels(): logrus.AllLevels` (as `hooks/test/test.go:54-56` does) gets stored once per level, so it appears in every bucket. The same hook instance fires once per event, but `LevelHooks.Add` is happy to add the same hook twice if called twice — there is no de-duplication (`hooks.go:18-22`).

## Open questions

- I did not read `hooks/syslog/syslog.go` or `hook_test.go`. They almost certainly contain richer examples (network-failing hooks, race tests for `AddHook` concurrent with logging — `hook_test.go:194` is `TestAddHookRace`) that would deepen the concurrency story but are not required to answer the registration-and-firing question.
- I did not trace each level method (`Info`, `Warn`, `Error`, etc.) into `Entry.log`. They all converge there based on the structure visible in `entry.go`, but I did not enumerate the call sites. If you need to confirm that, e.g., `Logger.Logf` or `Entry.Logf` cannot bypass hooks, run `/explain-codebase how does Entry.log get called --scope file:entry.go`.
- The `LevelHooks.Fire` method on `hooks.go:26-34` exists but the firing path uses `Entry.fireHooks` instead. Whether `LevelHooks.Fire` is dead public API kept for backward compatibility, or used by some external consumer pattern, is not visible from this walk.
