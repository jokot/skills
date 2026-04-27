# `logger.go` — what it is and what calls it

`logger.go` defines the central type of the `logrus` package: the `Logger` struct, its constructor, and essentially the entire public method surface that users (and the rest of the package) call to emit log records, configure the logger, and manage hooks.

It lives at `/tmp/explain-codebase-fixture/logrus-file-scope/logger.go`.

---

## What's inside

### 1. The `Logger` struct (lines 17–56)

The fields collectively describe a configured logger:

- `Out io.Writer` — destination for serialized log records (default `os.Stderr`); writes are guarded by a mutex.
- `Hooks LevelHooks` — per-level hook slices fired alongside each entry (e.g., error tracking, StatsD, core dumps on fatal).
- `Formatter Formatter` — turns entries into bytes; `TextFormatter` by default, `JSONFormatter` is the other built-in.
- `ReportCaller bool` — whether to include the calling function/file/line.
- `Level Level` — current min level; read/written via `sync/atomic` (see `level()` / `SetLevel()` lines 362–369).
- `mu MutexWrap` — wraps `sync.Mutex` so it can be disabled (see `SetNoLock`, line 358).
- `entryPool sync.Pool` — reuses `*Entry` allocations (`newEntry` / `releaseEntry`, lines 102–113).
- `ExitFunc func(int)` — defaults to `os.Exit`; called by `Fatal*` after `runHandlers()`.
- `BufferPool BufferPool` — optional per-logger buffer pool; falls back to package default when nil.

`MutexWrap` (lines 58–77) is a thin wrapper over `sync.Mutex` with a `Disable()` method so `SetNoLock()` can turn locking off when external append-mode atomicity is sufficient.

### 2. Construction

`New()` (lines 91–100) returns a `*Logger` with sensible defaults: `os.Stderr`, a fresh `TextFormatter`, an empty `LevelHooks`, `InfoLevel`, and `os.Exit`. Doc comment also shows the alternative struct-literal pattern.

### 3. Entry pool helpers

- `newEntry()` pulls a reusable `*Entry` from `entryPool`, falling back to `NewEntry(logger)` (defined in `entry.go`).
- `releaseEntry()` clears `entry.Data` and returns it to the pool.

These are used pervasively by every `With*` / `Log*` method to avoid per-call allocations.

### 4. Field/context decorators (lines 119–153)

`WithField`, `WithFields`, `WithError`, `WithContext`, `WithTime` — each grabs an entry from the pool, delegates to the matching `Entry` method, and (note!) defers `releaseEntry` *before* returning. The returned `*Entry` is what the caller then chains a real log call onto. (Releasing the entry while still returning it relies on `Entry.WithField` etc. allocating a *new* entry internally.)

### 5. Logging methods — three families

For each level (Trace / Debug / Info / Warn / Warning / Error / Fatal / Panic) plus a generic `Log…` form, there are three flavors:

| Family | Example | Semantics |
|---|---|---|
| Plain `args ...any` | `Info`, `Infof` is its `f`-variant | `fmt.Sprint`-style |
| `…f` (formatted) | `Infof(format, args...)` | `fmt.Sprintf`-style |
| `…ln` | `Infoln(args...)` | `fmt.Sprintln`-style (space-separated) |
| `…Fn` | `InfoFn(fn LogFunction)` | Defers message construction until level is enabled |

The `LogFunction` type (line 15) and the `*Fn` methods exist as an optimization for expensive log payloads — `LogFn` only invokes `fn()` after `IsLevelEnabled` passes.

The core dispatchers are `Logf` (155), `Log` (206), `LogFn` (214), and `Logln` (300). They all follow the same pattern: level-gate → pool an entry → delegate to the corresponding `Entry` method → release. `Print`/`Printf`/`Println` are unconditional (they ignore level, matching stdlib `log`).

`Fatal*` adds `logger.Exit(1)`; `Panic*` does not actually panic at this layer (the doc note on `Log`, lines 202–205, calls this out — `Logger.Panic*` actually does panic via `Entry.Panic*`, but `Log(PanicLevel, …)` itself does not).

### 6. Lifecycle / configuration

- `Exit(code int)` (347) — runs `runHandlers()` (the alt-exit registry from `alt_exit.go`) then calls `ExitFunc`.
- `SetNoLock` — disables the mutex.
- `level` / `SetLevel` / `GetLevel` / `IsLevelEnabled` — atomic level access (lines 362–401).
- `AddHook`, `ReplaceHooks`, `hooksForLevel` — hook management. `hooksForLevel` (385) returns a *shallow copy* of the hooks slice so callers can iterate without holding `mu`.
- `SetFormatter`, `SetOutput`, `SetReportCaller`, `SetBufferPool` — straightforward locked setters.

---

## What calls `logger.go`

There are three concentric rings of callers:

### A. Inside the package — code that directly reaches into `Logger`

- **`entry.go`** — `Entry` carries a `Logger *Logger` back-reference (line 56). Inside `Entry.log`, `fireHooks`, and `write`, it reads `entry.Logger.Formatter`, `entry.Logger.Out`, `entry.Logger.mu`, `entry.Logger.BufferPool`, calls `entry.Logger.IsLevelEnabled`, `entry.Logger.hooksForLevel`, and `entry.Logger.Exit(1)` (the last one is what makes `Entry.Fatal*` actually exit). `NewEntry(logger *Logger)` is also the constructor `Logger.newEntry()` falls back on.
- **`writer.go`** — defines two methods *on* `Logger`: `Writer()` and `WriterLevel(level)` (lines 11 and 20), which return `*io.PipeWriter`s that pipe arbitrary text into log calls. They build on `NewEntry(logger).WriterLevel(level)`.
- **`exported.go`** — holds the package-level `std = New()` (line 11) and a parallel set of free functions (`SetOutput`, `SetFormatter`, `SetLevel`, `AddHook`, `WithField`, `WithError`, `WithContext`, `Info`, `Infof`, `Fatal`, `Panic`, …) that just forward to `std.…`. This is how the common `logrus.Info("hello")` style works without users ever instantiating a `Logger`.
- **`logrus.go`** — defines shared types (`Fields`, `Level`, `Hook`, `LevelHooks`, etc.) consumed by `logger.go`'s API.
- **`hooks.go`**, **`formatter.go`**, **`text_formatter.go`**, **`json_formatter.go`** — collaborators referenced by the `Logger` fields, not callers per se, but they get exercised during `Entry.write` (which is reached via `Logger.Log…`).

### B. Tests in the same package

- **`logger_test.go`**, **`logger_bench_test.go`** — exercise `New`, `WithField`, `Logf`, level filtering, mutex behavior, and benchmark the pool.
- **`entry_test.go`**, **`entry_bench_test.go`**, **`hook_test.go`**, **`writer_test.go`**, **`text_formatter_test.go`**, **`logrus_test.go`**, **`formatter_bench_test.go`**, **`example_*_test.go`** — instantiate `Logger`s (mostly via `New()` or struct literals) to test the surrounding pieces.

### C. Subpackages shipped with logrus

- **`hooks/test/test.go`** — provides a `Hook` plus helpers; `NewGlobal()` calls `AddHook(hook)` on the standard logger; `NewLocal(logger *Logger)` registers on a caller-supplied one.
- **`hooks/syslog/syslog.go`** and **`hooks/slog/slog.go`** — additional hook implementations whose tests construct `Logger`s and call `AddHook`.
- **`internal/testutils/testutils.go`** — test helpers that take or build `Logger`s.

### D. External callers (the real audience)

Any application that imports `github.com/sirupsen/logrus` calls into `logger.go` in one of two ways:

1. **Via the package-level helpers in `exported.go`** — `logrus.Info(...)`, `logrus.WithField(...)`, `logrus.SetLevel(...)`. Every one of those calls is a thin wrapper that ultimately invokes a method defined in `logger.go` on the shared `std` instance.
2. **Via `logrus.New()` (or a struct literal) plus methods on the returned `*Logger`** — for users who want isolated loggers, custom outputs, or per-component configuration. Hook authors and library integrators typically take `*logrus.Logger` as a parameter.

---

## Mental model in one paragraph

`logger.go` is the "front door" of the package: it defines the configurable `Logger` value, owns the entry pool and the mutex around `Out`/`Hooks`/`Formatter`, and exposes the full level-by-level logging API. It delegates the actual formatting, hook firing, and writing to `Entry` (in `entry.go`), which holds a back-pointer to its `Logger` to read `Formatter`, `Out`, `BufferPool`, level, and hooks. `exported.go` mirrors the same API on a hidden `std` `*Logger` for the common "just use the package" pattern, and `writer.go` bolts an `io.Writer` adapter onto the same type. Outside the package, applications and hook libraries are the real callers, either through the `std` shortcut or through their own `*Logger` instances.
