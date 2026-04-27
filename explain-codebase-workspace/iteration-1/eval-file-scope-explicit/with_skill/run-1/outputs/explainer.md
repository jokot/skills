---
question: explain logger.go and what calls it
scope: file:logger.go
generated_at: 2026-04-25T09:15:04Z
visited_files: 9
depth_reached: 2
budget_state: early-stop
---

# Explain logger.go And What Calls It

## Overview

`logger.go` defines the `Logger` type — the central object every logrus log call goes through (`logger.go:17-56`). A `Logger` bundles an output sink (`Out io.Writer`), a `Formatter`, a `LevelHooks` map, an atomic `Level`, an `entryPool` for reuse, an `ExitFunc`, and a `MutexWrap` that serializes writes (and that can be disabled via `SetNoLock` at `logger.go:358-360`). `New()` constructs one with sensible defaults — stderr, `TextFormatter`, `InfoLevel`, `os.Exit` (`logger.go:91-100`).

The file is nearly all leveled-logging API surface: roughly four parallel families of methods — `Logf/Tracef/...`, `Log/Trace/...`, `Logln/Traceln/...`, and `LogFn/TraceFn/...` — all funneling through two cores: `Logf` (`logger.go:155-161`) and `Log` (`logger.go:206-212`). Each core gates on `IsLevelEnabled` (`logger.go:399-401`), pulls a pooled `*Entry` via `newEntry` (`logger.go:102-108`), delegates the actual formatting/hook-firing/writing to `entry.Logf` / `entry.Log` (which run `entry.log` at `entry.go:279-322`), and finally returns the entry to the pool. `Print*` methods deliberately skip the level gate (`logger.go:175-179, 234-238, 320-324`); `Fatal*` adds a trailing `logger.Exit(1)` (`logger.go:347-353`); panic behavior is triggered inside `entry.log` (`entry.go:319-321`), not here.

What calls `logger.go` falls into three groups. (1) The package-level "standard logger" in `exported.go`: `var std = New()` plus 40-odd thin wrappers (`Info`, `Errorf`, `WithFields`, `SetLevel`, ...) each of which calls the same method on `std` (`exported.go:11-265`). (2) `*Entry`, defined in `entry.go`, which holds a back-pointer `entry.Logger` and reads `Logger.mu`, `Logger.Formatter`, `Logger.Out`, `Logger.BufferPool`, `Logger.ReportCaller`, and `Logger.hooksForLevel` directly (`entry.go:127-129, 290-302, 324-329, 345-358`). (3) `writer.go`, which adds `Logger.Writer` / `Logger.WriterLevel` to expose an `io.PipeWriter` adapter (`writer.go:11-22`). Compile-time assertions in `logrus.go:121-126` lock `*Logger` to the `StdLogger` interface, so any code expecting `log.Logger` shape can swap in a logrus `*Logger`.

## Entry point

`Logger.Logf` is the canonical core that every other formatted level method delegates to:

```go
func (logger *Logger) Logf(level Level, format string, args ...any) {
    if logger.IsLevelEnabled(level) {
        entry := logger.newEntry()
        entry.Logf(level, format, args...)
        logger.releaseEntry(entry)
    }
}
```

(`logger.go:155-161`.) `Tracef`, `Debugf`, `Infof`, `Warnf`, `Errorf`, `Panicf`, and `Fatalf` are one-liners that call `Logf` with their level constant (`logger.go:163-200`). The unformatted `Log` (`logger.go:206-212`) and the lazy `LogFn` (`logger.go:214-220`) follow the same pattern.

## Sequence

A representative call — `logger.Info("hello")` — flows like this:

1. `logger.go:230-232` — `Info(args...)` calls `logger.Log(InfoLevel, args...)`.
2. `logger.go:206-212` — `Log` checks `logger.IsLevelEnabled(level)`; if false, returns immediately (this is the only fast-path gate).
3. `logger.go:399-401` — `IsLevelEnabled` reads `logger.level()` via `atomic.LoadUint32` (`logger.go:362-364`), so it is lock-free.
4. `logger.go:102-108` — `newEntry()` pops a reusable `*Entry` from `logger.entryPool`; on miss it allocates via `NewEntry(logger)` at `entry.go:94-100`.
5. `entry.go:367-371` — `entry.Log` re-checks `IsLevelEnabled` (defensive — `Logger.Log` already gated, but `Entry.Log` is also called from outside) and forwards to `entry.log(level, fmt.Sprint(args...))`.
6. `entry.go:279-322` — `entry.log` duplicates the entry, sets `Time`/`Level`/`Message`, snapshots `ReportCaller` and the buffer pool under `logger.mu`, fires hooks via `logger.hooksForLevel` (`logger.go:385-396`), formats, and writes.
7. `entry.go:340-361` — `entry.write` snapshots `logger.Formatter` under the lock, calls `formatter.Format(entry)` outside the lock (avoids reentrant deadlock), then re-acquires `logger.mu` to call `logger.Out.Write(serialized)`.
8. `logger.go:110-113` — back in step 4's pool path, `releaseEntry` clears `entry.Data` and returns it to `logger.entryPool`.

For `logger.Fatal(...)`: same flow as steps 1–7, then `logger.go:252-255` calls `logger.Exit(1)`, which runs `runHandlers()` from `alt_exit.go:42-46` and finally `logger.ExitFunc(code)` (default `os.Exit`, but overridable for tests — `logger.go:347-353`). For `logger.Panic(...)`: the panic is raised inside `entry.log` when `level <= PanicLevel` (`entry.go:319-321`), so `Logger.Panic` itself never returns.

## Files touched

- `logger.go` — defines `Logger`, `MutexWrap`, `LogFunction`; provides `New`, the level-method families (`Logf/Log/Logln/LogFn` × 7 levels), `Exit`, `SetLevel`/`GetLevel`/`IsLevelEnabled`, `AddHook`, `hooksForLevel`, `SetFormatter`/`SetOutput`/`SetReportCaller`/`ReplaceHooks`/`SetBufferPool`, `SetNoLock`, and the entry-pool helpers `newEntry`/`releaseEntry`.
- `exported.go` — the package-global `std` logger and ~40 forwarder functions; the most common caller of `Logger` methods.
- `logrus.go` — declares the `Level` type, the seven level constants (`PanicLevel`...`TraceLevel`), the `StdLogger`/`FieldLogger`/`Ext1FieldLogger` interfaces, and the compile-time interface satisfaction checks for `*Logger`.
- `entry.go` — `Entry` type, `NewEntry`, the real `entry.log` pipeline (lock dance, hooks, formatter, writer, optional panic). `Entry` reads many `Logger` fields directly via `entry.Logger`.
- `hooks.go` — `Hook` interface (`Levels()`, `Fire(*Entry)`) and `LevelHooks = map[Level][]Hook`; `Logger.Hooks` is this type.
- `alt_exit.go` — package-global `handlers` and `runHandlers()`; `Logger.Exit` calls `runHandlers()` before `ExitFunc`. `RegisterExitHandler`/`DeferExitHandler` are the public registration API.
- `formatter.go` — the `Formatter` interface (`Format(*Entry) ([]byte, error)`); `Logger.Formatter` holds one; `entry.write` invokes it.
- `buffer_pool.go` — package-level fallback `bufferPool` (a `sync.Pool` of `*bytes.Buffer`) used when `Logger.BufferPool` is nil; `SetBufferPool` replaces it globally.
- `writer.go` — adds `Logger.Writer()` and `Logger.WriterLevel(level)` returning an `*io.PipeWriter` that funnels lines into `NewEntry(logger).WriterLevel(...)`. Useful for adapting stdlib `log.SetOutput` into logrus.

## Gotchas

- `Print`, `Println`, `Printf`, and `PrintFn` deliberately bypass `IsLevelEnabled` (`logger.go:175-179, 234-238, 320-324`). Setting a high level does NOT silence them — they always emit at info-formatter level via `entry.Print*`.
- `Fatal*` and `FatalFn` exit even when `FatalLevel` is below the configured level — the call to `logger.Exit(1)` is unconditional after `Logf`/`Log`/`Logln`/`LogFn` returns (`logger.go:193-196, 252-255, 291-294, 338-341`). The `IsLevelEnabled` gate only suppresses the message; it does not stop the exit.
- `Panic*` does not panic in `logger.go`. The panic is raised inside `entry.log` at `entry.go:319-321`, only after the entry has been formatted and written. If the log line below `PanicLevel` is gated out, the panic is also gated out — surprising compared to `Fatal*`.
- `WithField`/`WithFields`/`WithError`/`WithContext`/`WithTime` look like leaks (they `releaseEntry` on a defer right after handing the entry to `entry.With*`) but the `entry.With*` methods return a *new* `*Entry` (`entry.go:147-215`), so releasing the pooled one is safe.
- `Level` is read with `atomic.LoadUint32` and written with `atomic.StoreUint32` (`logger.go:362-369`), but every other field (`Out`, `Formatter`, `Hooks`, `ReportCaller`, `BufferPool`) is protected only by `logger.mu`. Mutating those fields directly from outside without going through `SetOutput`/`SetFormatter`/etc. is racy.
- `SetNoLock` (`logger.go:358-360`) sets `MutexWrap.disabled = true`, after which Lock/Unlock become no-ops. Safe only if every writer is independently goroutine-safe (the comment cites file appends ≤4KB on Linux). Using it elsewhere will produce torn writes silently.
- Numeric ordering of `Level` is inverted from intuition: `PanicLevel=0`, `TraceLevel=6` (`logrus.go:100-119`). `IsLevelEnabled(level)` returns `logger.level() >= level`, so a higher numeric configured level is more verbose. Comparisons in your own code that reuse these constants must respect this.
- `hooksForLevel` (`logger.go:385-396`) returns a snapshot copy. If a hook modifies the entry, that change is visible to the formatter and to other hooks for the same call (because hooks share the duplicated `*Entry` from `entry.log` step 1) — but adding/removing hooks during a fire does NOT affect the in-flight call.

## Open questions

- `level.go` was not read. Given that the level constants are actually declared in `logrus.go:100-119`, it most likely contains test helpers or string parsing extensions; nothing in `logger.go` references it. Confirm with a follow-up.
- `text_formatter.go` and `json_formatter.go` were not read. They implement the `Formatter` interface invoked from `entry.write` (`entry.go:340-361`), so they affect the *output* of every `Logger` call but not its *control flow*. Out of scope for `--scope file:logger.go`; recommend `/explain-codebase how does TextFormatter format an entry --scope file:text_formatter.go` if you need it.
- The `_test.go` files (logger_test.go, entry_test.go, hook_test.go, ...) were intentionally skipped — tests would document expected behavior but not the production call graph. They are good targets for a separate `feature:tests` walk.

